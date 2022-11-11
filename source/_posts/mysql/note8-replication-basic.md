---
title: MySQL 复制概述
date: 2020-03-21 17:12:19
tags: [MySQL]
categories: [数据库]
---


复制是构建大规模、高性能应用的基础。

<!--more-->

复制可以让一台 MySQL 服务器与其他服务器保持同步， 提升集群的可用性， 避免因为一台MySQL 服务器挂掉导致整个系统不可用。

# 复制概述

## 复制的类型

MySQL 有两种复制方式： **基于行的复制和基于语句的复制**。二者都是通过记录主库的二进制日志来进行复制。

* 基于语句的复制：在主服务器执行的 SQL 语句，在从服务器执行相同的语句， 基于语句的复制是mysql 默认的复制方式， 效率高。

* 基于行的复制：从 5.1 版本开始支持， 把改变的内容复制过去。

* 混合复制： 当发现基于语句的复制不能执行下去时， 转换为基于行的复制。

MySQL 的复制是向后兼容的， 新版本的服务器可以做为旧版本的服务器的备库。

复制并不会增加主库的开销， 主要是启动二进制日志带来的开销。

##  复制解决了什么问题

MySQL 复制具有以下一些用途：

* 数据分布
* 负载均衡
* 备份
* 高可用性和故障切换
* MySQL 升级测试

## 复制是如何工作的

复制分三步：

1. 主库 (master) 将数据的更改记录到二进制日志 (binary log）中;
2. 备库 (slave) 从主库的日志复制到自身的中继日志 (relay log);
3. 备库读取中继日志中的事件， 修改和主库相对应的数据。

![复制原理](/replication-works.jpg)

# 复制配置
有两台 MySQL 服务器 master 和 slave, master 为主服务器， slave 为从服务器， 初始状态时两台服务器数据相同， 当 master 服务器发生变化时， slave 也跟着发生相应的变化， 使得 master 与 slave 的数据信息同步达到备份的目的。

## 创建账号
在 master 数据库创建一个备份账户，进行复制操作的用户被赋予 REPLICATION SLAVE 权限。

```java
mysql> GRANT REPLICATION SLAVE,RELOAD,SUPER ON *.*  TO backup@'10.26.15.167' IDENTIFIED BY '1234';
Query OK, 0 rows affected (0.00 sec)
```

建立一个帐户backup，并且只能允许从 10.26.15.167 这个地址上来登陆，密码是1234。

## 拷贝数据
将主库的数据拷贝到从库中去， 拷贝的过程中， 禁止在主库和从库进行写操作， 这样做的目的是为了保持两数据库中的数据一致。

## 设置主库、备库
假设主库是 server1， 需要在主库的 my.cnf 下增加或修改相应的内容：

```java
log_bin = mysql-bin 
server_id = 10
```
备库的配置也需要在 my.cnf 中增加类似的配置， 配置完成后重启服务器。

```java
log_bin     = mysql-bin 
server_id   = 2 
relay_log   = /var/lib/mysql/mysql-relay-bin   # 中继日志
log_slave_updates = 1
read_only  = 1
```
实际情况中， 以上的这些参数只有 server_id 是必需的。

## 启动slave

接下来就是让 slave 连接 master，并开始重做 master 二进制日志中的事件。具体操作通过 CHANGE MASTER TO 来执行， 替代了 my.cnf 中相应的配置。

```java
mysql> CHANGE MASTER TO MASTER_HOST='server1',
-> MASTER_USER='repl',
-> MASTER_PASSWORD='p4ssword',
-> MASTER_LOG_FILE='mysql-bin.000001',
-> MASTER_LOG_POS=0;
```
MASTER_LOG_POS 被置为 0 ，因为它是日志开始的地方。设置完毕后通过 SHOW SLAVE STATUS 来查看备库是否成功启动。

```java
mysql> SHOW SLAVE STATUS\G
*************************** 1. row ***************************
             Slave_IO_State:
                Master_Host: server1
                Master_User: repl
                Master_Port: 3306
              Connect_Retry: 60
            Master_Log_File: mysql-bin.000001
        Read_Master_Log_Pos: 4
             Relay_Log_File: mysql-relay-bin.000001
              Relay_Log_Pos: 4
      Relay_Master_Log_File: mysql-bin.000001
           Slave_IO_Running: No
          Slave_SQL_Running: No
                             ...omitted...
      Seconds_Behind_Master: NULL
```

其中 Slave_IO_State，Slave_IO_Running， Slave_SQL_Running 置为 No， 表示还没有开始复制。Relay_Log_Pos 为 4 意味着日志的起始位置是 4 而不是 0， 因为 0 只是日志文件开始的位置，并不是日志位置，MySQL 知道第一个事件起始于 4。

开启复制：

```java
mysql> START SLAVE;
```

现在再用 SHOW SLAVE STATUS 命令检查：

```java
mysql> SHOW SLAVE STATUS\G

*************************** 1. row ***************************
             Slave_IO_State: Waiting for master to send event
                Master_Host: server1
                Master_User: repl
                Master_Port: 3306
              Connect_Retry: 60
            Master_Log_File: mysql-bin.000001
        Read_Master_Log_Pos: 164
             Relay_Log_File: mysql-relay-bin.000001
              Relay_Log_Pos: 164
      Relay_Master_Log_File: mysql-bin.000001
           Slave_IO_Running: Yes
          Slave_SQL_Running: Yes
                             ...omitted...
      Seconds_Behind_Master: 0
```

可以看到 I/O 线程和 SQL 线程已经开始运行。Seconds_Behind_Master 也不再为 NULL, 日志的位置也发生了变化。 查看下线程列表中的复制线程， 在主库可以看到由备库 I/O 线程向主库发起的连接。

```java
mysql> show processlist \G

*************************** 1. row ***************************
     Id: 1
   User: root
   Host: localhost:2096
     db: test
Command: Query
   Time: 0
 State: NULL
   Info: show processlist
*************************** 2. row ***************************
     Id: 2
   User: repl
   Host: localhost:2144
     db: NULL
Command: Binlog Dump
   Time: 1838
 State: Has sent all binlog to slave; waiting for binlog to be updated
   Info: NULL
2 rows in set (0.00 sec)
```

在slave服务器上运行该语句， 其中行 1是 I/O 线程， 行 2 是 SQL 线程

```java
mysql> show processlist \G

*************************** 1. row ***************************
     Id: 1
   User: system user
   Host:
     db: NULL
Command: Connect
   Time: 2291
 State: Waiting for master to send event
   Info: NULL
*************************** 2. row ***************************
     Id: 2
   User: system user
   Host:
     db: NULL
Command: Connect
   Time: 1852
 State: Has read all relay log; waiting for the slave I/O thread to update it
   Info: NULL

*************************** 3. row ***************************
     Id: 5
   User: root
   Host: localhost:2152
     db: test
Command: Query
   Time: 0
 State: NULL
   Info: show processlist
3 rows in set (0.00 sec)
```

## 增加新的 SLAVE 服务器

前面讲到的例子是主备两台服务器的数据相同， 并且知道当前主库的二进制日志。 但是大多数情况下是有一个已经运行了一段时间的主库， 然后用新安装的备库与之同步， 此时备库还没有数据。

有几种方法来初始化备库或者从其他服务器克隆数据到备库。 包括从主库复制数据、从另一台备库克隆数据， 以及使用最近的一次备份启动备库。 需要三个条件让主库、备库保持一致：

* master 某个时间点的数据快照。
* master 当前的二进制日志文件， 和获得数据快照时在该二进制日志文件中的偏移量， 通过这两个值可以确定二进制日志的位置。
* master从快照时间到现在的二进制日志。

可以通过以下几种方法从 master 克隆到 slave：

1. 冷拷贝(cold copy)
关闭master， 将数据复制到 slave， 复制完毕后重启 master。 缺点： 复制时需要关闭 master。

2. 热拷贝(warm copy)
仅限于 MyISAM表， 可以在 master 运行时使用 mysqlhotcopy 或 rsync 来复制数据。

3. 使用mysqldump

* 锁表， 避免其他连接修改数据库。 否则会造成数据不一致的问题
    mysql> FLUSH TABLES WITH READ LOCK;
* 在另一个连接使用 mysqldum 创建一个你想进行复制的数据库的存储：
shell> mysqldump --all-databases --lock-all-tables >dbdump.db
* 对表释放锁


# 复制的原理

之前介绍的是一些复制的基本概念， 接下来深入地了解复制， 看看复制是如何工作的， 有哪些优缺点。

## 基于语句的复制

MySQL5.0 及之前更老的版本是只支持基于语句的复制（逻辑复制）。 在基于语句的复制模式下， 主库会记录那些造成数据更改的查询， 当备库读取并重放这些事件时， 实际上是把主库执行过的语句再执行一遍。

基于语句的复制十分简单，理论上将简单地记录和执行这些语句即可让主备保持同步， 另一个好处是二进制日志更加紧凑， 所以基于语句的复制不会占用太多的带宽。

但是在基于语句复制的过程中还依赖于其他的元素，比如当前的时间戳，除此之外还存在一些无法被正确复制的SQL， 例如使用 CURRENT_USER()函数的语句；

另一个问题是更新必须是串行的，需要更多的锁。

## 基于行的复制

MySQL5.1 开始支持基于行的复制， 这种方式会将实际数据记录在二进制日志中。 优点是任何语句都可以正确的工作， 一些语句的效率更高。 缺点是二进制文件可能很大而且不直观。

对于一些语句，基于行的复制能够更有效的工作，如：

```java
mysql> INSERT INTO summary_table(col1, col2, sum_col3)
    -> SELECT col1, col2, sum(col3)
    -> FROM enormous_table
    -> GROUP BY col1, col2;
```
假设，只有三种唯一的col1和col2的组合，但是，该查询会扫描原表的许多行，却仅返回三条记录。此时，基于行的复制效率更高。

另一方面，下面的语句，基于语句的复制更有效：

```java
mysql> UPDATE enormous_table SET col1 = 0;
```
此时使用基于行的复制代价会非常高。由于两种方式不能对所有情况都能很好的处理，所以，MySQL 5.1支持在基于语句的复制和基于记录的复制之前动态交换。你可以通过设置session变量binlog_format来进行控制。

## 基于语句复制与基于行复制的优缺点

|  | 优点 | 缺点 |
| --- | --- | --- |
|基于语句的复制模式  |  基于语句的复制更灵活， 哪怕是主备表不同单类型能够兼容或者是列不同等， 而且基于语句出现问题容易排查| 对触发器和存储过程不友好， 有很多 bug |
|基于行的复制模式  | SQL 构造器、触发器、存储过程都能正确执行，应用广阔。不需要太多的锁。占用更少的 CPU | 无法判断执行了哪些SQL， 出现问题难以排查 |

## 复制文件 

复制中除了二进制日志和中继日志文件呢， 还用到了以下的一些文件：

* mysql-bin.index：

    当在服务器开启二进制日志时， 同时会生成一个和二进制日志同名但以.index 为后缀的文件，它用来定位磁盘上的二进制日志文件。
    
* mysql-relay-bin-index：

    中继日志的索引文件， 和mysql-bin.index 的作用类似。
    
* master.info:

    保存 master 库的信息， 不能删除， 否则 slave 无法连接到 master。
    
* relay-log.info：

    包含当前备库复制的二进制日志和中继日志坐标， 同样不能删除， 否则无法获知从哪个位置开始复制。
    
## 发送复制事件到其他 slave

当开启 log_slave_updates 选项可以让 slave 变成其他服务器的master。 slave 把执行过的事件记录在自己的二进制日志中， 它的slave 就可以从日志中检索并执行事件

![slave 作为 master 发送复制事件](/replicaiton-to-slave.jpg)

## 复制过滤器

复制过滤器可以让你只复制一部分数据， 有两种复制过滤：master 上过滤记录到二进制日志文件的事件， slave 上过滤记录到中继日志的事件。

![复制过滤器](/replication-of-filter.jpg)


# 复制的常见拓扑结构

复制体系结构有以下一些基本原则：
* 一个 MySQL slave 实例只能有一个 master。
* 每个 slave 必须有一个唯一的 server id。
* 一个 master 可以有多个 slave
* 如果打开了 log_slave_updates 选项， 一个 slave 可以将主库的数据变化传播给其他的 slave。

## 一个 master 多个 slave

一主多备是最简单的拓扑结构， slave 之间没有交互。
当少量写大量读时这种结构非常有用， 将读操作分摊到多个 slave， 从而减小 master 的压力。但是当 slave 增加到一定程度时， slave 对 master 的负载以及网络带宽都会成为性能瓶颈。

![一主多从](/一主多从.jpg)

这种结构虽然简单， 但是很灵活， 足以满足大多数需求， 一些建议：
* 不同的 slave 起到不同的作用（使用不同的索引或者不同的存储引擎）。
* 用一个 slave 当做代用的 master， 只进行复制。
* 用一个远程 slave， 作为灾备

## 主动-主动模式下的 Master-Master 复制

Master-Master 复制的两台服务器，既是  master，又是另一台服务器的 slave。任何一方的变更都会同步给另一方。

![主动模式下的多主复制](/master-master-of-active.jpg)

master-master 模式带来的最大问题是如何解决冲突， 如果两台服务器同时修改一条记录。 在 MySQL5.0 之后通过 auto_increment_inrement 和auto_increment_offset， 可以让 MySQL 自动为 INSERT 语句选择不互相冲突的值， 但这仅仅解决了同时想一个包含 AUTO_INCREMENT 列的表插入数据造成的冲突。
在两台主库上同时写入数据还是很危险， 假设同时执行下面的两条语句：

* 第一台 master 上：
    msyql> UPDATE tbl SET col = col + 1;
    
* 第二台 master 上：
    msyql> UPDATE tbl SET col = col * 2;

结果可能是一台服务器上值为 4， 一台服务器上值为 3， 而且没有报任何错误。所以这种模式很少使用。

## 主动-被动模式下的 Master-Master 复制

这是master-master结构变化而来的，它避免了M-M的缺点，实际上，这是一种具有容错和高可用性的系统。它的不同点在于其中一个服务只能进行只读操作。如图：

![被动模式下的多主复制](/master-master-of-passive.jpg)


这种拓扑结构使得反复切换主动和被动服务器很方便， 可以应用在故障转移和故障恢复中。

## 用于备库的主-主结构(Master-Master with Slaves)

![多主多备](/master-master-with-slaves.jpg)

这种配置的有点事增加了冗余， 对于不同地理位置的复制拓扑， 能够献出单点失效的问题， 同样的， 查询还是分配在 slave 上。