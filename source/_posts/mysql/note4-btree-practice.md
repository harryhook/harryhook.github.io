---
title: B-Tree 索引实践
date: 2020-02-29 18:20:42
tags: [MySQL]
categories: [数据库]
---

看下 B-Tree 索引在实际的数据查询中起到了什么作用。
<!--more-->
# 准备测试数据

## 新建表

```java
create table tb_user(
  id int auto_increment primary key,
  name varchar(10),
  birth date
);
```

## 利用存储过程造数据

存储过程可以理解为 MySQL 的一个函数， 调用存储过程就可以执行函数中相应的操作；

在创建存储过程之前需要对 "," 特殊处理下， 因为MySQL 将 "," 作为一条语句的终结符号，为了是创建的存储过程中的 "," 生效， 需要提前使用 delimiter 将终结符 "," 替换成其他符号。

比如使用 **delimiter $$**， 就将 "," 替换成了 $$。

创建生成随机字符串与随机数生成的方法：

```java
delimiter $$
# 生成随机字符串
create function rand_str(n int) returns varchar(10)
begin
  declare CHARS char(52) default 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
  declare result varchar(255) default '';
  declare i int default 1;
  while i < n do
    set result = concat(result, substr(CHARS, floor(1 + RAND()*52), 1));
    set i = i + 1;
  end while;
  return result;
end
$$

# 生成随机数(i <= R < j)
create function rand_num(i int, j int) returns int
begin
  return floor(i + rand() * (j - i));
end
$$

delimiter ;
```

创建测试数据生成的存储过程：

```java

delimiter $$
create procedure insert_tb_user(c int)
begin
  declare start_date int default TO_DAYS(STR_TO_DATE('1970-01-1','%Y-%m-%e'));
  declare end_date int default TO_DAYS(CURDATE());
  declare i int default 0;
  set autocommit = 0;
  repeat
    set i = i + 1;
    insert into tb_user(name, birth) values(rand_str(10), FROM_DAYS(rand_num(start_date, end_date)));
  until i = c end repeat;
  set autocommit = 1;
end
$$
delimiter ;
```

调用存储过程生成一百万条随机数据：

```java
call insert_tb_user(1000000);
```

可能需要几分钟的时间，我的电脑上生成一百万条数据花费 286 秒。

# 使用主键查询

为了能更细致地看到每条语句的耗时， 我们需要把 profiles 打开

```java
mysql> set profiling=1;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> select * from tb_user where id = 10000;
+-------+-----------+------------+
| id    | name      | birth    |
+-------+-----------+------------+
| 10000 | jSMfqpAKV | 1994-01-25 |
+-------+-----------+------------+
1 row in set (0.00 sec)

mysql> show profiles;
+----------+------------+-------------------------------------------+
| Query_ID | Duration   | Query                                     |
+----------+------------+-------------------------------------------+
|        1 | 0.0.00027100 | select * from tb_user where id = 10000  |
+----------+------------+-------------------------------------------+
1 row in set, 1 warning (0.00 sec)
```

可以看到使用主键查询时耗时 **0.271毫秒**

看下当前 tb_user 的索引情况

```java
mysql> show index from tb_user\G;
*************************** 1. row ***************************
        Table: tb_user
   Non_unique: 0
     Key_name: PRIMARY
 Seq_in_index: 1
  Column_name: id # 索引列为 id 列
    Collation: A
  Cardinality: 978848
     Sub_part: NULL
       Packed: NULL
         Null:
   Index_type: BTREE # 主键索引也是 B-Tree 索引
      Comment:
Index_comment:
1 row in set (0.00 sec)

mysql> explain select * from tb_user where id = 10000\G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb_user
   partitions: NULL
         type: const
possible_keys: PRIMARY 
          key: PRIMARY   # 用到了主键索引
      key_len: 4
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
```


# 不使用索引查询

接下来我们根据 **name** 去查询数据， name 所在的字段现在还没有索引

```java
mysql> select * from tb_user where name = 'jSMfqpAKV';
+--------+-----------+------------+
| id     | name      | birth      |
+--------+-----------+------------+
|  10000 | jSMfqpAKV | 1994-01-25 |
+--------+-----------+------------+
2 rows in set (0.20 sec)
```

在 osx 上 mysql 并不区分大小写， 索引查询到两条数据， 接下来我们看下具体的耗时

```java
+----------+------------+------------------------------------------------+
| Query_ID | Duration   | Query                                          |
+----------+------------+------------------------------------------------+
|        2 | 0.19593300 | select * from tb_user where name = 'jSMfqpAKV' |
+----------+------------+------------------------------------------------+
1 rows in set, 1 warning (0.00 sec)
```
相比于使用 id 的主键查询， name 查询的速度慢了近千倍

```java
mysql> show profile for query 2;
+----------------------+----------+
| Status               | Duration |
+----------------------+----------+
| starting             | 0.000050 |
| checking permissions | 0.000006 |
| Opening tables       | 0.000012 |
| init                 | 0.000023 |
| System lock          | 0.000007 |
| optimizing           | 0.000008 |
| statistics           | 0.000013 |
| preparing            | 0.000009 |
| executing            | 0.000002 |
| Sending data         | 0.195761 |   # 速度变慢的主要原因
| end                  | 0.000008 |
| query end            | 0.000004 |
| closing tables       | 0.000006 |
| freeing items        | 0.000016 |
| cleaning up          | 0.000008 |
+----------------------+----------+
```

可以看到Sending data 是查询耗时的主要原因， 这个 Sending data 到底是什么东西？ 是传输数据吗？ 
[MySQL官方手册](https://dev.mysql.com/doc/refman/8.0/en/general-thread-states.html)的说法是 线程正在读取并处理select语句选择的行数据，然后将数据发送给客户端。因为这个状态期间的操作偏重执行大量的**磁盘访问(读取磁盘)**，它通常是整个查询生命周期中运行时间最长的状态。 所以说导致查询耗时的主要原因还是磁盘 IO。

# 使用索引查询

接下来我们对 name 字段加上索引

```java
mysql> create index idx_tb_user_name on tb_user(name);
Query OK, 0 rows affected (1.44 sec)

-- 目前表中的索引
mysql> show index from tb_user\G;
*************************** 1. row ***************************
        Table: tb_user
   Non_unique: 0
     Key_name: PRIMARY
 Seq_in_index: 1
  Column_name: id
    Collation: A
  Cardinality: 978848
     Sub_part: NULL
       Packed: NULL
         Null:
   Index_type: BTREE
      Comment:
Index_comment:
*************************** 2. row ***************************
        Table: tb_user
   Non_unique: 1
     Key_name: idx_tb_user_name
 Seq_in_index: 1
  Column_name: name
    Collation: A
  Cardinality: 988297
     Sub_part: NULL
       Packed: NULL
         Null: YES
   Index_type: BTREE  # 可以在创建索引时默认的类型也是 BTREE 索引
      Comment:
Index_comment:
2 rows in set (0.00 sec)
```

继续刚才的查询语句：

```java
mysql> select * from tb_user where name = 'jSMfqpAKV';

mysql> show profiles;
+----------+------------+------------------------------------------------+
| Query_ID | Duration   | Query                                          |
+----------+------------+------------------------------------------------+
|        1 | 0.00028700 | select * from tb_user where name = 'jSMfqpAKV' |
|        2 | 0.00027100 | select * from tb_user where id = 10000         |
+----------+------------+------------------------------------------------+
```

此时耗时只有 0.28 毫秒, 虽然和主键查询相差无几， 但仍然没有主键查询快， 这是因为用过 name 索引是二级索引， 在查询时二级索引不会保留具体的数据行，保留的是主键 id。 查询时会根据二级索引先查询到 id， 再根据主键 id 找到数据所在的行。

# 使用索引时的注意事项

## 禁止在查询语句中使用表达式

```java
mysql> explain select * from tb_user where id + 1 = 5\G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb_user
   partitions: NULL
         type: ALL  # 全部查询
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 998195
     filtered: 100.00
        Extra: Using where
1 row in set, 1 warning (0.00 sec)
```
可以看到使用表达式时索引是没有生效的。

## 避免将索引列作为参数

```java
--- 先对 birth 创建索引

mysql> create index idx_tb_user_birth on tb_user(birth);
mysql> select count(*) from tb_user where YEAR(current_date) - YEAR(birth) >= 18 \G;

+----------+------------+-----------------------------------------------------------------------------------+
| Query_ID | Duration   | Query                                                                             |
+----------+------------+-----------------------------------------------------------------------------------+
|       12 | 0.14596000 | select count(*) from tb_user where YEAR(current_date) - YEAR(birth) >= 18         |
+----------+------------+-----------------------------------------------------------------------------------+
```
可以看到索然 在 birth 字段上新增了索引， 但是查询耗时为 0.14 秒， 和之前没加索引查询的耗时是差不多的。


## 模糊查询时使用前缀索引

有时需要索引很长的字段时， 会让索引变的很大， 通常有两种做法， 一直是采用哈希索引， 另一种就是使用模糊查询。

```java
mysql>  select * from tb_user where name like  '%jSMfq%';
+--------+-----------+------------+
| id     | name      | birth      |
+--------+-----------+------------+
| 312246 | jsmfQLlVW | 2017-03-03 |
| 654276 | jsmfQMmbv | 2019-12-07 |
| 629126 | jSMfqmpLL | 1991-08-12 |
...
|  10000 | jSMfqpAKV | 1994-01-25 |
| 298551 | jsmfQPAkv | 1995-12-22 |
| 230069 | JSmFqPcQW | 1982-06-15 |
+--------+-----------+------------+
81 rows in set (**0.16** sec) --耗时 0.16 秒

mysql> explain select * from tb_user where name like  '%jSMfq%'\G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb_user
   partitions: NULL
         type: ALL # 全表扫描
possible_keys: NULL
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 998195
     filtered: 11.11
        Extra: Using where
1 row in set, 1 warning (0.00 sec)

mysql> explain select * from tb_user where name like  'jSMfq%'\G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb_user
   partitions: NULL
         type: range  # 范围查询
possible_keys: idx_tb_user_name
          key: idx_tb_user_name  
      key_len: 13
          ref: NULL
         rows: 11
     filtered: 100.00
        Extra: Using index condition # 使用索引进行条件查询
1 row in set, 1 warning (0.00 sec)

```
这是由于 BTREE 索引的数据是有序的使用 like '%xxx%'时是无法使用索引的。

##  不要对每一列都创建单独的索引

为每个列创建单独的索引大部分情况下并不能提升 MySQL 的查询性能，在 MySQL5.0 及更新的版本中，引入了索引合并的策略， 将每列查询的结果进行合并， 举个例子：

```java
mysql> explain  select * from tb_user where id = 10000 or name like 'jSMfqp%'\G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb_user
   partitions: NULL
         type: index_merge
possible_keys: PRIMARY,idx_tb_user_name
          key: idx_tb_user_name,PRIMARY
      key_len: 13,4
          ref: NULL
         rows: 4
     filtered: 100.00
        Extra: Using sort_union(idx_tb_user_name,PRIMARY); Using where   # MySQL 内部索引合并
1 row in set, 1 warning (0.00 sec)
```
通过 explain 可以看到， MySQL 在实际的查询过程内还进行类内部优化， 将 通过 id 查询的数据与 name 查询的数据做了合并操作，在实际的查询过程中建立一个相关列的多列索引效果会更好。

```java
mysql> explain select * from tb_user where name = 'jSMfqpAKV' and  birth ='1970-01-01'\G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb_user
   partitions: NULL
         type: index_merge
possible_keys: idx_tb_user_name,idx_tb_user_birth
          key: idx_tb_user_name,idx_tb_user_birth
      key_len: 13,4
          ref: NULL
         rows: 1
     filtered: 100.00
        Extra: Using intersect(idx_tb_user_name,idx_tb_user_birth); Using where; Using index   # 取了两部分数据的交集
1 row in set, 1 warning (0.00 sec)
```

此时 name 与 birth 都只有单独的索引， Extra中的信息表明使用了交集来进行数据合并。 我们为 name，birth 建立一个多列索引后再进行查询

```java

mysql> create index idx_name_birth on tb_user(name, birth);
Query OK, 0 rows affected (1.14 sec)
Records: 0  Duplicates: 0  Warnings: 0

mysql> explain select * from tb_user where name = 'jSMfqpAKV' and  birth ='1970-01-01'\G;
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: tb_user
   partitions: NULL
         type: ref
possible_keys: idx_tb_user_name,idx_tb_user_birth,idx_name_birth
          key: idx_name_birth
      key_len: 17
          ref: const,const
         rows: 1
     filtered: 100.00
        Extra: Using index  # 通过索引查询
1 row in set, 1 warning (0.00 sec)

```
可以看到在实际的查询中是走的索引进行查询， MySQL 无需再对查询的结果优化处理。


参考： [MySQL性能优化\[实践篇\]-使用B树索引](https://blog.hufeifei.cn/2018/04/24/DB/MySQL%E6%80%A7%E8%83%BD%E4%BC%98%E5%8C%96%5B%E5%AE%9E%E8%B7%B5%E7%AF%87%5D-%E4%BD%BF%E7%94%A8B%E6%A0%91%E7%B4%A2%E5%BC%95/)