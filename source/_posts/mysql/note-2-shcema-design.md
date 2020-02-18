---
title: 《高性能MySQL》-Schema设计与数据类型优化
date: 2020-02-16 16:30:50
tags: [MySQL]
categories: [数据库]
---

MySQL常见的数据类型有哪些？ 数据库在设计的过程中应该注意哪些地方？

<!--more-->

# 如何选择数据类型

* 越小越好

    越小的数据类型占用更少的磁盘、内存、CPU缓存。 但需要注意当前类型存储值的范围。
   
* 简单就好

    简单的数据类型需要更少的CPU周期， 整形比字符串的代价要低。 两个例子， 使用MySQL内建的datetime或者timestamp存储日期和时间而不是用字符串存储。还有就是应该使用整形存储ip， 而不是字符串存储。
    
* 尽量避免null值

    通常情况下最好指定列为not null， 查询包含null的列成本更高， 而且会使用更多的存储空间， 当NULL列被索引时， 需要额外的一个字节去存储。
    
 
# 常见的数据类型

## 整数类型

整数类型分别有以下： TINYINT, SMALLINT, MEDIUMINT, INT, BIGINT。分别占用8， 16， 24， 32， 64位存储空间，以TINYINT为例， 存储范围为-2^7 ~ 2^7-1, 即-128~127。

整数类型可选UNSIGNED属性， 表示不允许为负值， 例如 TINYINT UNSIGNED可存储的范围0~255。

限定整数类型的范围是没有意义的， INY(2)与INT(20)是相同的， 只是展示字符的个数会受到限制。

## 实数类型

实数指的是带有小数部分的数字，但不一定只用实数存储带小数的值， 还可以用DECIMAL存储比BIGINT还大的整数。 因为CPU是不支持DECIMAL的直接运算的，MySQL内部支持高精度运算。因为CPU支持浮点数的计算， 所以浮点预算的速度更快， MySQL内部是以DOUBLE作为内部浮点计算的类型。

因为需要额外的空间和计算开销， 所以只在精确计算时才使用DECIMAL， 但是当数据量比较大时， 可以考虑使用BIGINT代替DECIMAL， 将需要存储的小数乘以相应的倍数， 避免浮点数存储计算不精确和DECIMAL计算代价过高的问题。

## 字符串类型

MySQL支持多种字符串类型， 每种类型还有很多变种。 其中VARCHAR和CHAR是最主要的两种字符串类型。

* VARCHAR

    VARCHAR类型用于存储可变长字符串， 是最常见的数据类型， 比定长类型更节省空间。越短的字符串使用越少的空间， 但在ROW_FORMAT=FIXED时是例外情况，每一行都会使用定长存储。
    
    VARCHAR需要使用1到2个额外的字节记录字符串的长度。 如果列的最大长度=255字节， 只需要1个字节表示，否则需要2个。 2^8 = 256， VARCHAR(10)需要11个字节的存储空间， VARCHAR(1000)需要1002个字节， 因为需要2个字节存储长度信息。
    
    VARCHAR节省了空间， 但是当执行UPDATE操作时， 行可能变的更长， 当页内没有多余的空间可以存储时，InnoDB需要分裂页来处理，MyISAM需要将行拆成不同的片段存储， 但InnoDB会将过长VARCHAR存储为BLOB
    
* CHAR 

    CHAR是定长的， MySQL会根据定义的字符串长度分配足够的空间。 CHAR适合存储很短的字符串， 或者所有值都接近同一长度， 例如密码的MD5值。 **CHAR有一个特点， 就是会删除所有末尾的空格**， 存储数据时需要注意。
    
例子：

```java
CREATE TABLE char_test(
    char_col CHAR(10),
    varchar_col VARCHAR(10)
);

INSERT INTO char_test(char_col) VALUES ('string1'), ('  string2'), ('string3  ');
INSERT INTO char_test(varchar_col) VALUES ('string1'), ('  string2'), ('string3  ');

select CONCAT("'",char_col, "'")  from char_test;
+---------------------------+
| CONCAT("'",char_col, "'") |
+---------------------------+
| 'string1'               |
| '  string2'             |
| 'string3'               |
+---------------------------+

select CONCAT("'",varchar_col, "'")  from char_test;
+------------------------------+
| CONCAT("'",varchar_col, "'") |
+------------------------------+
| 'string1'                  |
| '  string2'                |
| 'string3  '                |
+------------------------------+
```

可以看到char存储字符串数，string3的末尾的空格被截断了， 而varchar存储数据时不会存在这种情况。


* BLOB与TEXT类型

	BLOB与TEXT都是为存储很大的数据而设计的字符串数据类型， 分别采用二进制和字符方式存储， MySQL将BLOB与TEXT值当一个独立的对象处理，当BLOB和TEXT的值太大时，MySQL会专门建立一个存储区域来进行存储， 在行内用1~4个字节存储一个指针， 指向外部存储实际值的区域。
    
	BLOG与TEXT的区别在于BLOB存储的是二进制数据， 没有排序规则或者字符集，TEXT类型有字符集和排序规则。

	此外MySQL对BLOB与TEXT列进行排序与其他类型是不同的： 只对每列最前max_sort_length字节而不是整个字符串进行排序，可以动态调整max_sort_length或者ORDER BY SUNSTRING(column, length)来进行排序。
  
	MySQL不能将BOLB与TEXT列的全部长度的字符串进行索引， Memory引擎也不支持BLOB和TEXT类型， 如果查询BLOB或者TEXT列将生成隐式临时表， 即使临时表存储在内存上系统间的调用开销也很大， 所以尽量不要使用BLOB或者TEXT类型。 万不得已使用了，在用到BLOB字段的地方使用SUBSTRING(column, length）将列值转换为字符串。


* 枚举

	枚举代替常见的字符串

```java
CREATE TABLE enum_test(
	e ENUM('fish', 'apple', 'dog') NOT NULL);
    
--插入不属于枚举的值时--

INSERT INTO enum_test(e) VALUES ('A'), ('B'), ('C');
ERROR 1265 (01000): Data truncated for column 'e' at row 1

select * from enum_test;
+-------+
| e     |
+-------+
| dog   |
| apple |
| fish  |
+-------+
```

看一下实际枚举对应的整数是：

```java
select e + 0 from enum_test;
+-------+
| e + 0 |
+-------+
|     3 |
|     2 |
|     1 |
+-------+
```

**可以看到枚举字段在内部存储是按照整数而不是定义的字符串进行排序的**

插入的是fish(3), apple(2), dog(1), 查询的是dog(1), apple(2), fish(3)
如果想要按照自定义的顺序展示， 可以使用FIELD()函数

```java
select e from enum_test order by FIELD(e, 'dog', 'apple', 'fish');
+-------+
| e     |
+-------+
| dog   |
| apple |
| fish  |
+-------+
```

使用ENUM的建议： 不会做修改的列， 例如性别。
ENUM值千万不要使用数值型，会带来混淆。


## 日期与时间类型

MySQL提供两种相似的时间类型： DATETIME和TIMESTAMP

* DATETIME

    范围比TIMESTAMP大， 从1001年到9999年， 精度为秒， 把日期封装成YYYYMMDDHHMMSS的整数中， 与时区无关， 使用8个字节存储。
    
* TIMESTAMP

    保存了1970-01-01 00:00:00 以来的秒数， 和UNIX时间戳相同。 TIMESTAMP只使用四个字节的存储空间， 范围比DATETIME小得多， 只能表示从1970年只2038年的数据， 且TIMESTAMP显示的值是区分时区的。
    
    TIMESTAMP也有DATETIME没有的属性， 如果数据在插入时没有指定第一个TIMESTAMP列的值， MySQL会赋予这个列为当前时间。

通常使用TIMESTAMP保存时间。 MySQL现在没有提供合适的数据类型去处理比秒更小粒度的时间， 可以使用BIGINT类型存储微妙级别的时间戳

## 位数据类型

* BIT(**不推荐使用**)

    MySQL将BIT当做字符串类型而不是整数类型， 检索BIT(1)时， 结果是一个包含二进制0或1的字符串， 而不是ASCII码的"0"或"1"， 很容易令人迷惑， 举个例子：
    
```java
CREATE TABLE bit_test(a bit(8));
INSERT INTO bit_test VALUES(B'01000001');
SELECT a, a+0 FROM BIT_TEST;

+------+------+
| a    | a+0  |
+------+------+
| A    |   65 |
+------+------+
```

当存储'01000001'字符串时， 检索出的结果为ASCII码为65的字符"A"， 在数字上下文的检索中， 得到的是数字65。

    
* SET
    唯一的缺点是修改表的成本比较高， 当保存很多的true/false时可以考虑使用SET， 这里看一下SET的用法：
    
```java

CREATE TABLE acl(perms SET('CAN_READ', 'CAN_WRITE', 'CAN_DELETE') NOT NULL);
INSERT INTO acl(perms) VALUES('CAN_READ,CAN_DELETE');
SELECT perms FROM acl WHERE FIND_IN_SET('CAN_READ', perms);

+---------------------+
| perms               |
+---------------------+
| CAN_READ,CAN_DELETE |
+---------------------+

SELECT perms FROM acl WHERE FIND_IN_SET('CAN_DELETE', perms);

+---------------------+
| perms               |
+---------------------+
| CAN_READ,CAN_DELETE |
+---------------------+
```

## 如何选择合适的类型作为标识列

* 整数类型

    标识列的不二选择， 通常作为主键， 因为可以使用AUTO_INCREMENT。

* ENUM与SET

    不要使用ENUM,SET作为标识列， ENUM与SET适合存储固定信息，例如有序的状态、人的性别。

* 字符串类型

    避免使用字符串作为标识列， 原因在于很消耗空间，并且比数字类型慢。

# 范式与反范式

三范式的定义分别是：

* 一范式

    指数据库表的每一列都是不可分割的基本数据项，同一列中不能有多个值

* 二范式

    表中的每个实例（每行）都可以被唯一区分，为了被区分，通常为表加上一列以存储每个实例的唯一标示（**主键**）

* 三范式
    
    数据库表不包含已在其他表的非主关键字信息，即包含已在其他数据库表中的主关键字信息（**外键**）
    
举例子来说， 下面是一张 雇员-部门-上级表：

| EMPLOYEE |DEPARTMENT  |HEAD  |
| --- | --- | --- |
| Jones |Accounting  |Jones  |
| Simth | Engineering |Simth  |
|Brown  | Accounting | Jones |
| Green | Engineering | Simth |

这种表满足第一范式， 每列数据都是单独的值， 但是不满足第二范式， 不能唯一的标识一列， 加入Brown接替Accounting部门， 将同时修改两行数据， 此外如果删除了Accounting部门的所有雇员， 就失去了关于这个部门的记录。为了满足第二范式， 我们将表拆分成 **雇员-部门**，**部门-上级** 这种关系。

雇员-部门

| EMPLOYEE_NAME |DEPARTMENT |
| --- | --- |
| Jones |Accounting  |
| Simth | Engineering |
|Brown  | Accounting | 
| Green | Engineering |

部门-上级

|DEPARTMENT  |HEAD  |
| --- | --- |
|Accounting  |Jones  |
| Engineering |Simth  |

这样就算修改任意部门的领导也主需要修改部门-上级即可， 对部门雇员的数据是没有影响的， 哪怕Accounting部门没有雇员， 对这个部门也是没有影响的。

## 范式的优缺点

优点：

* 范式的更新操作通常比反范式更快。
* 符合范式， 很少或者没有重复数据，因此修改数据也会少很多。
* 范式化的表通常更小。
* 很少有多余的数据意味着检索数据时更少执行DISTINCT和GROUP BY操作。比如DEPARTMENT是一张单独的表， 就不需要对雇员表进行GROUP BY department操作。
   
缺点：

* 查询数据时需要进行多张表的关联。

## 反范式的优缺点 

优点:

* 所有数据都在一张表， 可以很好的避免关联。

缺点：

* 所有数据都在一张表， 带来的问题就是字段过多， 修改成本高。


# 缓存表与汇总表-提升查询效率

* 缓存表
    存储从从其他查询速度较慢的表的数据
    
* 汇总表
    保存的是分组后的数据

其中有一个计数器的策略很有意思， 为了避免对某行数据并发写阻塞全局， 将计数放到多行进行， 最终统计多行的数据总和即可。

# 提升ALTER TABLE效率 

对大表来说， ALTER TABLE可能需要几小时甚至数天才能完成操作， 因为ALTER操作涉及到锁表和重建表。
常见的ALTER TABLE有两种操作：
1. 先在不提供服务的机器上执行ALTER TABLE操作，然后进行主库切换；
2.  创建一张与源表结构一致的表， 然后通过重命名与删表操作交换两张表。

其中

```java
ALTER TABLE table_name MODIFY COLUMN column_name ....;
```

执行过程是先读取表中所有数据， 再将列替换插入到新表， 所以**MODIFY COLUMN**操作是相当慢的

```java
ALTER TABLE table_name ALTER COLUMN column_name ....;
```

这条语句会直接修改.frm文件， 而.frm文件中存的是表的默认值， 所以这种操作是非常快的。一句话，修改表的默认值应该使用ALTER COLUMN 操作
为高效地载入数据到MyISAM， 一个技巧是先禁用索引、载入数据、重启索引， 唯一的问题是这个方法对唯一索引是无效的。
在InnoDB也有类似的操作， 先删除所有的非唯一索引， 增加新的列， 然后再创建新的索引。