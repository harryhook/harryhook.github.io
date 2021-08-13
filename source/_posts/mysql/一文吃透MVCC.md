---
title: 一文吃透 MVCC
date: 2021-08-08 23:03:43
tags: [MySQL]
categories: [数据库]
---

MVCC 到底是个什么东西？为什么 RC 就不可重复读了！
<!--more-->

# 一文吃透 MVCC

# 案例

在了解 MVCC 机制前，先看以下的案例

数据库表 test 有以下数据

```java
mysql> select * from users;
+----+--------+-----+
| id | name   | age |
+----+--------+-----+
|  1 | 小兰   | 18  |
|  7 | 小明   | 19  |
|  8 | 小东   | 20  |
| 12 | 张三   | 21  |
+----+--------+-----+
```

```sql

---事务 1
begin;
update users set age = '20' where id = 1;
update users set age = '21' where id = 1;
commit;
```

```sql
---事务 2
begin;
select * from test;
```

在 RR 级别下同时开启事务 1 与事务 2， 先执行事务 1 的 update 的语句后， 事务 2 查询会得到

```java
+----+--------+
| id | name   |
+----+--------+
|  5 | 小兰   |
|  7 | 小明   |
|  8 | 小东   |
| 12 | 张三   |
+----+--------+
```

这是因为 RR 级别下不可重复读的特性使得事务在查询时始终都得到相同的结果， 再往深研究其实是 MVCC 机制发挥作用。同样的语句在 RC 级别下，事务 1 执行更新语句commit后，事务 2 的查询结果是：

```java
---事务 1 更新前
mysql> select * from users;
+----+--------+-----+
| id | name   | age |
+----+--------+-----+
|  1 | 小兰   | 18  |
|  7 | 小明   | 19  |
|  8 | 小东   | 20  |
| 12 | 张三   | 21  |
+----+--------+-----+
---事务 1 更新后
mysql> select * from users;
+----+--------+-----+
| id | name   | age |
+----+--------+-----+
|  1 | 小兰   | 21  |
|  7 | 小明   | 19  |
|  8 | 小东   | 20  |
| 12 | 张三   | 21  |
+----+--------+-----+
```

究其原因在与 RR 与 RC 级别下 MVCC的不同实现策略导致查询的结果不同

# MVCC

**多版本并发控制**技术的英文全称是 **Multiversion Concurrency Control**，简称 **MVCC**。

**MVVC** 是通过保存数据在某个时间点的快照来实现并发控制的。

## MVCC解决了哪些问题

提升了查询性能，降低了死锁概率， 只在写数据时加锁

解决了读写并发问题， 读写互不阻塞

解决一致性读问题

## MVCC 是如何工作的

### 事务版本号

每开启一个事务， 可以获得一个事务版本号，事务 ID 递增，通过事务 ID 可以知晓事务的执行时间顺序

### 行隐藏列

- DB_ROW_ID:6-byte，隐藏的行 ID，用来生成默认聚簇索引。如果我们创建数据表的时候没有指定聚簇索引，这时 InnoDB 就会用这个隐藏 ID 来创建聚集索引。采用聚簇索引的方式可以提升数据的查找效率。
- DB_ROLL_PTR:7-byte，回滚指针，也就是指向这个记录的 Undo Log 信息。
- DB_TRX_ID:6-byte，操作这个数据的事务 ID，也就是最后一个对该数据进行插入或更新的事务 ID。

![undolog](/undolog.png)

### undo log

InnoDB 将行记录快照保存在了 Undo Log 里，我们可以在回滚段中找到它们，如下图所示：

由版本链组成的视图


![MVCC](/mvcc.svg)

### READ VIEW

read view 是在 SQL 语句执行之前创建的，在 read view 中会保存：

- `low_limit_id` - 创建 `read view` 时 **尚未提交** 的事务中的 **最大** 的事务 ID
- `up_limit_id` - 创建 `read view` 时 **尚未提交** 的事务中的 **最小** 的事务 ID
- `trx_ids` - 创建 `read view` 时 **尚未提交** 的事务列表
- 

通过 `read view` 可以将所有事务分为三组：

- `trx_id < up_limit_id` - 创建 `read view` 时已经提交了的事务
- `up_limit_id <= trx_id <= low_limit_id` - 创建 `read view` 时正常执行的事务
- `trx_id > low_limit_id` - 创建 `read view` 时还未创建的事务

此时，我们可以根据 `read view` 来判断行记录的可见性：

1. 当记录的 `DB_TRX_ID` 小于 `read vew` 的 `up_limit_id` 时说明该记录在创建 `read view` 之前就已经提交，记录可见
2. 如果记录的 `DB_TRX_ID` 和事务创建者的 `TRX_ID` 一样时，记录可见
3. 当记录的 `DB_TRX_ID` 大于 `read vew` 的 `up_limit_id` 时，说明该记录在创建 `read view` 之后进行的新建事务修改提交的，记录不可见
4. 如果记录对应的 `DB_TRX_ID` 在 `read view` 的 `trx_ids` 里面，那么该记录也是不可见的, trx_ids的这些事务都未提交。

### RR & RC 创建视图时机

对于 RR 级别来说，就只能在事务开始之前创建 read view，在创建事务后提交的数据对于当前事务来说都是不可见的。

在 RC 隔离级别下，每个 SELECT 语句开始时，都会重新将当前系统中的所有的活跃事务拷贝到一个列表生成 ReadView。***二者的区别就在于生成 ReadView 的时间点不同，一个是事务之后第一个 SELECT 语句开始、一个是事务中每条 SELECT 语句开始***。

# 总结

`RC`、`RR` 两种隔离级别的事务在执行普通的读操作时，通过访问版本链的方法，使得事务间的读写操作得以并发执行，从而提升系统性能。`RC`、`RR` 这两个隔离级别的一个很大不同就是生成 `ReadView` 的时间点不同，`RC` 在每一次 `SELECT` 语句前都会生成一个 `ReadView`，事务期间会更新，因此在其他事务提交前后所得到的 `m_ids` 列表可能发生变化，使得先前不可见的版本后续又突然可见了。而 `RR` 只在事务的第一个 `SELECT` 语句时生成一个 `ReadView`，事务操作期间不更新。