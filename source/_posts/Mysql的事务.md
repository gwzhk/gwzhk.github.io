---
title: Mysql的事务
date: 2018-09-21 15:33:01
categories:
- Mysql
tags:
- Mysql
- 事务
- 隔离级别
---
> 数据库具有ACID属性，即原子性、一致性、隔离性和持久性。Mysql的事务隔离级别一共有四种，分别是读未提交、读已提交、可重复读和可序列化。下面会具体演示脏读、不可重复读和幻读，要解决以上的问题需要设定相应的事务隔离级别。

<!-- more -->

## Mysql事务
### 事务的简介
#### 为什么需要事务
现在的很多软件都是多用户，多程序，多线程的，对同一个表可能同时有很多人在用，为保持数据的一致性，所以提出了事务的概念。
> A 给B 要划钱，A 的账户-1000元， B 的账户就要+1000元，这两个update 语句必须作为一个整体来执行，不然A 扣钱了，B 没有加钱这种情况很难处理。

#### 什么样的存储引擎支持事务
查看数据库下面的存储引擎是否支持事务?

```
//只有InnoDB支持事务
SHOW ENGINES;
```
![mysql存储引擎](https://img-blog.csdn.net/20180918112128312?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2d3el9oaw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

#### 查看mysql当前的存储引擎

```
SHOW VARIABLES LIKE '%storage_engine%';
```
![默认的存储引擎](https://img-blog.csdn.net/20180918112221199?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2d3el9oaw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

####查看某张表的存储引擎

```
SHOW CREATE TABLE 表名;
```

```
//最后ENGINE=CVS，该表的存储引擎为CVS
CREATE TABLE `slow_log` (
  `start_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  `user_host` mediumtext NOT NULL,
  `query_time` time NOT NULL,
  `lock_time` time NOT NULL,
  `rows_sent` int(11) NOT NULL,
  `rows_examined` int(11) NOT NULL,
  `db` varchar(512) NOT NULL,
  `last_insert_id` int(11) NOT NULL,
  `insert_id` int(11) NOT NULL,
  `server_id` int(10) unsigned NOT NULL,
  `sql_text` mediumtext NOT NULL,
  `thread_id` bigint(21) unsigned NOT NULL
) ENGINE=CSV DEFAULT CHARSET=utf8 COMMENT='Slow log'
```
#### 修改表的存储引擎

```
#创建表的时候指定存储引擎
CREATE TABLE '表名'()......ENGINE=INNODB;
#修改表的存储引擎
ALTER TABLE 表名 ENGINE=INNODB;
```

#### 事务的特性
事务应该具有4个属性：原子性、一致性、隔离性、持久性。这四个属性通常称为ACID特性。
##### 原子性（atomicity）：
> 一个事务是一个不可分割的工作单位，事务中包括的所有操作要么都做，要么都不做。



例子
一个事务必须被视为一个不可分割的最小单元，整个事务中的所有操作要么全部提交成功，要么全部失败，对于一个事务来说，不可能只执行其中的一部分操作。

> 公司给员工发工资
1、公司账户扣除5000元；
2、员工账户增加5000元。
不能出现工资账户扣除了5000元，但是员工账户没有增加5000元的情况。
要么全部成功，要么全部失败。

##### 一致性（consistency）：
> 事务必须是使数据库从一个一致性状态变到另一个一致性状态，一致性与原子性是密切相关的。


**例子**

一致性是指事务将数据库从一种一致性转换到另外一种一致性状态，在事务开始之前和事务结束之后数据库中数据的完整性没有被破坏。
> 公司给员工发工资
1、公司账户扣除5000元；
2、员工账户新增5000元；
2、员工账户新增1000元。
不能因为任何原因，员工收到两次钱。

##### 隔离性（isolation）：
隔离性要求一个事务对数据库中数据的修改，在未提交完成前对于其他事务是不可见的。
> 一个事务的执行不能被其他事务干扰，即一个事务内部的操作及使用的数据对并发的其他事务是隔离的，并发执行的各个事务之间不能互相干扰。


##### 持久性（durability）：
一旦事务提交，则其所做的修改就会永久保存到数据库中。此时即使系统崩溃，已经提交的修改数据也不会丢失。
> 持久性也称永久性（permanence），指一个事务一旦提交，它对数据库中数据的改变就应该是永久性的。接下来的其他操作或故障不应该对其有任何影响。

### 事务的隔离级别
mysql默认的事务隔离级别是repeatable-read（可重复读）。<br>
 查看事务的隔离级别

```
SHOW VARIABLES LIKE '%tx_isolation%';
```
 事务有4种隔离级别
> READ UNCOMMITED（读未提交，脏读）
> READ COMMITTED（读已提交，不可重复读）
> REPEATABLE READ（可重复读）
> SERIALIZABLE（可串行化）

#### 事务并发的问题
- 脏读：事务A读取了事务B更新的数据，然后事务B回滚操作，那么A读取到的数据就是脏数据。
- 不可重复读：事务A多次读取同一数据，事务B在事务A多次读取的过程中，对数据做了更新并提交，导致事务A多次读取的同一数据时，结果不一致。
- 幻读：系统管理员A将数据库中所有学生的成绩从具体分数改为ABCDE等级，但是系统管理员B就在这个时候插入了一条具体分数的记录，当系统管理员A修改结束后发现还有一条记录没有改过来，就好像发生了幻觉一样，这就是幻读。

> 不可重复读和幻读很容易混淆，不可重复读侧重于修改，幻读侧重于新增或删除。解决不可重复读的问题只需锁住满足条件的行，解决幻读需要锁表。

#### 读未提交（READ UNCOMMITED，脏读）

```
#将SESSION事务隔离级别改为READ UNCOMMITTED
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
#当前session开启一个事务
START TRANSACTION;
#开启一个新的session、设置事务隔离级别为READ UNCOMMITTED，并开启一个事务
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
START TRANSACTION;
#在第一个session中将id为1的账户余额减掉50
UPDATE account SET balance = balance - 50 WHERE id = 1;
#在第二个session中查询
SELECT * FROM account WHERE id = 1;//发现此时查询账户中已经少了50
#在第一个session中rollback
ROLLBACK;
#在第二个session中再次查询
SELECT * FROM account WHERE id = 1;//发现此时查询账户的钱又变回去了
```

时刻 | session1 | session2
---|---|---
1 | SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED; | SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
2 | START TRANSACTION; | START TRANSACTION;
3 | UPDATE account SET balance = balance - 50 WHERE id = 1; | 
4 |  | SELECT * FROM account WHERE id = 1;
5 | ROLLBACK; | 
6 |  | SELECT * FROM account WHERE id = 1;
7 |  | COMMIT;

####  读已提交（READ COMMITTED，不可重复读）

```
#开启两个session并把每个session的事务隔离级别设置为READ COMMITTED
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
#在两个session中分别开启事务
START TRANSACTION;
#在一个session中执行update
UPDATE account SET balance = balance - 50 WHERE id = 1;
#在第二个session中查询
SELECT * FROM account WHERE id = 1;//此时查询数据未发生变化
#在第一个session中执行commit
COMMIT;
#在第二个session中再次查询
SELECT * FROM account WHERE id = 1;//此时查询的数据已经发生变化
```
时刻 | session1 | session2
---|---|---
1 | SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED; | SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
2 | START TRANSACTION; | START TRANSACTION;
3 | UPDATE account SET balance = balance - 50 WHERE id = 1; | 
4 |  | SELECT * FROM account WHERE id = 1;
5 | COMMIT; | 
6 |  | SELECT * FROM account WHERE id = 1;
7 |  | COMMIT;

####  可重复读（REPEATABLE READ,会出现幻读）
正常情况

```
#开启两个session并把每个session的事务隔离级别设置为REPEATABLE READ
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
#在两个session中分别开启事务
START TRANSACTION;
#在第二个session中执行查询
SELECT * FROM account WHERE id = 1;
#在一个session中执行update
UPDATE account SET balance = balance - 50 WHERE id = 1;
#在第二个session中查询
SELECT * FROM account WHERE id = 1;//此时查询的数据与第一次查询的一致
#在第一个session中执行commit
COMMIT;
#在第二个session中再次查询
SELECT * FROM account WHERE id = 1;//此时查询的数据还是与第一次查询的一致
```
时刻 | session1 | session2
---|---|---
1 | SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ; | SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
2 | START TRANSACTION; | START TRANSACTION;
3 | | SELECT * FROM account WHERE id = 1;
4 | UPDATE account SET balance = balance - 50 WHERE id = 1; | 
5 |  | SELECT * FROM account WHERE id = 1;
6 | COMMIT; | 
7 |  | SELECT * FROM account WHERE id = 1;
8 |  | COMMIT;
出现幻读的情况
```
#开启两个session并把每个session的事务隔离级别设置为REPEATABLE READ
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
#在两个session中分别开启事务
START TRANSACTION;
#分别在两个session中查询记录数
SELECT * FROM account;//发现都是3条记录
#在第一个session中执行insert
INSERT INTO account VALUE(4,'gwz',520);
#在第一个session中再次查询数量
SELECT * FROM account;//发现变为4条记录
#在第二个session中再次查询数量
SELECT * FROM account;//发现还是3条记录
#在第二个session中执行insert
INSERT INTO account VALUE(4,'hk',1314);
#在第二个session中再次查询数量
SELECT * FROM account;//发现变为4条记录
#两个session中的事务都提交后，再次查询
SELECT * FROM account;//发现有5条记录
```
时刻 | session1 | session2
---|---|---
1 | SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ; | SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
2 | START TRANSACTION; | START TRANSACTION;
3 | SELECT * FROM account; | SELECT * FROM account;
4 | INSERT INTO account VALUE(4,'gwz',520); | 
5 | SELECT * FROM account; | SELECT * FROM account;
6 |  | INSERT INTO account VALUE(4,'hk',1314);
7 | SELECT * FROM account; | SELECT * FROM account;
8 | COMMIT; | COMMIT;
9 | SELECT * FROM account; | 
> 将事务隔离级别改为SERIALIZABLE后，第一个session可以正常插入，第二个session插入会报错，可以避免幻读，因为可串行化会锁表。


####  事务隔离级别总结
![事务隔离级别](https://img-blog.csdn.net/20180918112513130?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2d3el9oaw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

> 事务隔离级别为可重复读时，如果有索引（包括主键索引）的时候，以索引列为条件更新数据，会存在[间隙锁](https://blog.csdn.net/lz710117239/article/details/78776446)、行锁、页锁的问题，从而锁住一些行；如果没有索引的时候，更新数据时会锁住整张表。<br>
事务隔离级别为串行化时，读写数据都会锁住整张表。<br>
隔离级别越高，越能保证数据的完整性和一致性，但是对并发性能的影响也越大，对于多数应用程序，可以<font color=red>优先考虑把数据库系统的隔离级别设为Read Committed</font>，它能够避免脏读取，而且具有较好的并发性能。

###  事务的语法
#### 开启事务

```
BEGIN;
START TRANSACTION;#推荐
BEGIN WORK;
```
#### 事务的回滚

```
ROLLBACK;
```
####  事务的提交

```
COMMIT;
```
####  还原点

```
#查询自动提交事务是否开启
SHOW VARIABLES LIKE '%autocommit%';
#关闭事务自动提交
SET autocommit = 0;
#开启事务
BEGIN;
SELECT * FROM account;
INSERT INTO account VALUE(7,'AAA',111);
#设置还原点s1
SAVEPOINT s1;
INSERT INTO account VALUE(8,'BBB',222);
#设置还原点s2
SAVEPOINT s2;
INSERT INTO account VALUE(9,'CCC',333);
#设置还原点s3
SAVEPOINT s3;
#回退到还原点s1
ROLLBACK TO SAVEPOINT s1;
#全部回退
ROLLBACK;
```



