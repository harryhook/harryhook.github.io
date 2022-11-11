---
title: MySQL 优化器与特定类型优化
date: 2020-03-12 22:46:32
tags: [MySQL]
categories: [数据库]
---

优化器存在哪些问题？ 日常的查询需要注意哪些点？
<!--more-->

# 查询优化器存在的问题

## 关联子查询

常见的一种查询， 希望找到 Sakila 数据库中， 演员 PG(actor_id = 1) 所参演的影片信息, 一般来讲我们都会以下面的方式完成子查询：

```java
mysql> SELECT * FROM sakila.film
-> WHERE film_id IN(
-> SELECT film_id FROM sakila.film_actor WHERE actor_id = 1);
```
大多数情况下我们都会以为上面的语句执行时会按照下面的顺序执行：
```java
-- SELECT GROUP_CONCAT(film_id) FROM sakila.film_actor WHERE actor_id = 1;
-- Result: 1,23,25,106,140,166,277,361,438,499,506,509,605,635,749,832,939,970,980 

SELECT * FROM sakila.film
WHERE film_id IN(1,23,25,106,140,166,277,361,438,499,506,509,605,635,749,832,939,970,980);
```
但是时间情况却是MySQL 会先全表扫描 film 表， 然后进行子查询，依次比对 film 表中的 film_id 与 file_actor 表中 actor_id = 1的 film_id， sql 在执行时将会改写成以下语句：
```java
SELECT * FROM sakila.film 
WHERE EXISTS (
    SELECT * FROM sakila.film_actor WHERE actor_id = 1
    AND film_actor.film_id = film.film_id);
```
如果外层的表 film 数据量并不是很大的时候， 对性能的影响不会引起注意， 如果外层的表很大， 这个查询的性能就会很糟糕， 常见的一个优化的方式为：

```java
mysql> SELECT film.* FROM sakila.film
-> INNER JOIN sakila.film_actor USING(film_id) 
-> WHERE actor_id = 1;
```
至于使用子查询还是内/外连接， 需要看具体的执行计划， 并不一定说内/外连接的性能一定比子查询好。

## UNION 限制

UNION 的限制条件只对已联合的数据生效， 如果想得到更好的性能则需要在联合前就对数据做出限制：

比如, actor 有 200 条数据， customer 有 599 条数据，按 last_name 合并后， 会生成一张含有 799 条数据的临时表， 然后再从临时表去除 20 条数据：
```java
(SELECT first_name, last_name FROM sakila.actor
ORDER BY last_name)
UNION ALL
(SELECT first_name, last_name
FROM sakila.customer
ORDER BY last_name) LIMIT 20;
```
数据量很大时， 临时表可能会很大， 可以在 UNION 的子查询中加上 LIMIT 来减少临时表的数据：
```java
(SELECT first_name, last_name FROM sakila.actor
ORDER BY last_name
LIMIT 20)
UNION ALL
(SELECT first_name, last_name
FROM sakila.customer ORDER BY last_name LIMIT 20)
LIMIT 20;
```

## 索引合并优化

当查询语句中有 OR 时, 会求并集。查询SELECT * FROM TB1 WHERE c1="xxx" OR c2=""xxx"时，如果c1和c2列上分别有索引，可以按照c1和c2条件进行查询，再将查询结果合并（union）操作。

当查询语句中有 OR 时, 会求交集。如查询SELECT * FROM TB1 WHERE c1="xxx" AND c2=""xxx"时，如果c1和c2列上分别有索引，可以按照c1和c2条件进行查询，再将查询结果取交集（intersect）操作。

索引索引合并的性能并不及复合索引。

## 等值传递

非常大的 IN()列表 MySQL 优化器会将这个列表的值与另一个表做关联，类似于子查询是对 IN 语句的改写。

## 哈希关联
MySQL 的查询都是嵌套循环查询， 不支持哈希关联。 

## 松散索引扫描
MySQL 的索引扫描需要指定起点和终点。假设有索引 key(a,b), 如何查询的语句中只要字段 b， MySQL 是无法使用这个索引的， 只能通过全表扫描查找数据。
 而松散索引是先扫描 a 列对应的 b 列的范围，再跳到 a 列第二个不同值扫描对应 b 列的范围。更好的做法是为 b 增加一个索引，但是也会遇到第一列为范围查询， 第二列为等值查询的情况。
 
## 最大最小值优化

```java
SELECT MIN(actor_id) FROM sakila.actor WHERE first_name = 'PENELOPE';
```
first_name 没有索引时， 会进行全表扫描。 当进行主键扫描时（actor_id）为主键， 读到的第一行记录就是最小值。此时采用 LIMIT 将查询重写：
```java
mysql> SELECT actor_id FROM sakila.actor USE INDEX(PRIMARY)
    -> WHERE first_name = 'PENELOPE' LIMIT 1;
```
虽然进行了优化， 但是会让人产生迷惑，不知到这个 sql 语句想表达什么。

## 同一个表的查询与更新
MySQL 不允许对同一张表同时进行查询和更新。


# 优化特定的类型
## 优化 COUNT()
COUNT()函数有两个作用：
*  统计某个列值的数量。 统计列值时要求列值是非空的（NOT NULL）或者表达式的值补不为NULL。
*  统计结果集的行数。COUNT(*）时， * 并不会扩展成所有列，而是直接统计行数。

MyISAM count()的速度很快是因为前提条件是没有任何WERE条件的COUNT(*)。
技巧1： 查询一个很多行时， 反向查询
SELECT (SELECT COUNT(*) FROM CITY ) - COUNT(*) FROM CITY WHERE id <= 5; 可以迅速查的id大于5的城市
技巧2：COUNT 替代SUM
SELECT COUNT(color = 'blue' OR NULL) as blue, COUNT(color = 'red' OR NULL) as red FR0M ITEMS;


## 优化关联查询
* 当表 A 与表 B 关联是， 只需要在关联顺序的第二张表相应列创建索引即可
* GROUP BY 和 ORDER BY 只涉及到一个表的列才能使 MySQL 索引生效

## 优化子查询
子查询尽量使用关联来代替， “尽可能地使用关联”，但不一定正确。

## 优化 GROUP BY 
无法使用索引时， GROUP BY 使用临时表或者文件排序来实现分组

## 优化 LIMIT分页
LIMIT 分页时需要留意偏移量， 特别是偏移量非常大的时候， 例如 LIMIT 1000, 20, 只返回了最后20条数据， 前面的 1000 条被丢弃。
采用延迟关联, 获取需要访问的数据后再根据关联列会原表查询所需要的列
```java
SELECT film_id, description FROM film ORDER BY title LIMIT 50, 5;
改写成：
SELECT film_id, description 
FROM film
    INNER JOIN(
        SELECT film_id FROM film ORDER BY title LIMIT 50, 5
    ) AS lim UNSING(film_id);
```

或者将 LIMIT 查询转换成已知位置查询
SELECT film_id, description FROM film WHERE position； BETWEEN 50 AND 54  ORDER BY position；

OFFSET 在使用时可以记录上一次查询的位置配合 LIMIT 使用
```java
SELECT * FROM rental WHERE  ORDER BY rental_id DESC LIMIT 16049, 20;

SELECT * FROM rental WHERE rental_id < 16030 ORDER BY rental_id DESC LIMIT 20;
```
## 优化 UNION
UNION在使用时要注意是否是UNION ALL， 否则在查询时还需要对临时表去重

## 使用自定义变量
使用自定义变量需要注意：
* 使用自定义变量时， 无法查询缓存；
* 不能在使用常量、标识符的地方使用自定义变量，例如表名、列名、LIMIT 子句中；
* 用户的自定义变量只在一个session中有效， 不能做为连接间的通信；
* 不能显示声明自定义变量的类型；赋值为0默认为整形， 0.0为浮点型， ‘’为字符串；
* 赋值符号  := 的优先级非常低；
* 自定义变量不会产生语法错误。