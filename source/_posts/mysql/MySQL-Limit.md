---
title: MySQL Limit 用法
date: 2022-03-13 23:03:43
tags: [MySQL]
categories: [数据库]
---

常见的mysql limit有以下3种
<!--more-->

1. limit 3
    查询满足条件的前三条数据
2. limit 3, 1
    满足条件的三条数据， 从1开始， 查出2，3，4
3. limit 3 offset 1， 取出三条数据，跳过1，查出2，3，4

针对limit offset 的优化，又名书签优化法

优化前：

```sql
select * from table  limit 10 offset 80001;
```

优化后：

```sql
select * from table id >80000 limit 10;
```