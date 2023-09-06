---
title: mysql-storage-engine
date: 2023-08-29 09:37:33
categories:
- 数据库
- mysql
tags:

---

# mysql-存储引擎

## mysql体系结构

![image-20230829093941681](..\images\image-mysql体系结构.png)

连接层，服务层，引擎层，存储层

## 存储引擎

存储引擎是存储数据，建立索引，更新/查询数据等技术的实现方式。存储引擎是基于表的，而不是基于库的，所以存储引擎可以被称为表类型。

### InnoDB

兼顾高可靠性和高性能的通用存储引擎，在mysql5.5之后，InnoDB是默认的存储引擎。

特点：

DML操作遵循ACID模型，支持**事务**；

**行级锁**，提高并发访问性能；

支持外键foreign key约束，保证数据的完整性和正确性。

文件：

xxx.ibd：xxx代表的是表名，innoDB引擎的每张表都会对应这样一个表空间文件，存储该表的表结构（frm，sdi），索引和数据。

参数：innodb_file_per_table

### MyISM

mysql早期的默认存储引擎。

特点：

不支持事务，不支持外键

支持表锁，不支持行锁

访问速度快

文件：

xxx.sdi：存储表结构信息

xxx.MYD：存储数据
xxx.MYI：存储索引

### Memory

表数据存储在内存中，由于受到硬件问题，或断电问题的影响，只能将这些表作为临时表或缓存使用

特点：

内存存放

hash索引（默认）

文件：

xxx.sdi：存储表结构信息

![image-20230829100341545](..\images\image-引擎特点.png)

## InnoDB

![image-20230906102612387](..\images\innodb.png)

表空间

ibd文件，一个mysql实例可以对应多个表空间，用于存储记录，索引等数据

段

分为数据段，索引段，回滚段，innodb是索引组织表，数据段就是b+树的叶子节点，索引段为b+树的非叶子节点。

区

表空间的单元结构，每个区的大小为1M。默认情况下，innodb存储引擎页大小为16k，即一个区中一共有64个连续的页。

页

innodb存储引擎磁盘管理的最小单元，每个页的大小默认为16kb，为了保证页的连续性，innodb存储引擎每次从磁盘申请4-5个区。

行

innodb存储引擎数据是按行进行存放的。

### 架构

![image-20230906103426248](..\images\innodb架构.png)

### 后台线程

![image-20230906133932389](F:\swag-lay.github.io\source\images\innodb后台线程.png)

master thread

核心后台线程，负责调度其他线程，还负责将缓冲池中的数据异步刷新到磁盘中，保持数据的一致性，还包括脏页的刷新，合并插入缓存，undo页的回收

io thread

大量使用了aio来处理io请求，这样可以极大地提高数据库的性能，而io thread负责这些io的回调

| 线程类型             | 默认个数 | 职责                         |
| -------------------- | -------- | ---------------------------- |
| read thread          | 4        | 负责读操作                   |
| write thread         | 4        | 负责写操作                   |
| log thread           | 1        | 负责将日志缓冲区刷新到磁盘   |
| insert buffer thread | 1        | 负责将写缓冲区内容刷新到磁盘 |

purge thread

主要用于回收事务已经提交了的undo log，在事务提交之后，undo log可能不用了，就用它来回收。

page cleaner thread

协助master thread刷新脏页到磁盘的线程，他可以减轻master thread的工作压力，减少阻塞

### 事务原理

不可分割的工作单位，把所有的操作作为一个整体一起向系统提交或撤销操作请求，即这些操作要么同时成功，要么同时失败。

acid特性

原子性，一致性，持久性是由redo log，undo log决定的。

隔离性是由锁，mvcc决定的。

redo log：重做日志，记录的是事务提交时数据页的物理修改，是用来实现事务的持久性。改日志文件由两部分组成：重做日志缓冲（redo log buffer）以及重做日志文件（redo log file），前者是在内存中，后者在磁盘中。当事务提交之后会把所有修改信息都存到改日志文件中，用于在刷新脏页到磁盘，发生错误时，进行数据恢复使用。



undo log：回滚日志，用于记录数据被修改前的信息，作用包含两个：提供回滚和mvcc（多版本并发控制）

undo log和redo log记录物理日志不一样，它是逻辑日志。

undo log销毁：undo log在事务执行时产生，事务提交时，并不会立即删除undo log，因为这些日志可能还用于mvcc

undo log存储：undo log采用段的方式进行管理和记录，存放在前面介绍rollback segment回滚段中，内部包含1024个undo log segment

当insert的时候,产生的undo log日志只在回滚时需要,在事务提交后,可被立即删除.

而update,delete的时候,产生的undo log日志不仅在回滚时需要,在快照时也需要,不会立即被删除

undo log版本链:不同事务或相同事务对同一条记录进行修改,会导致该记录的undolog生成一条记录版本链表,链表的头部是最新的旧记录,链表尾部是最早的旧记录.

### MVCC

####  当前读

读取的是记录的最新版本，读取时还要保证其他并发事务不能修改当前记录，会对读取的记录进行加锁。对于我们日常的操作，如：select...lock in share mode(共享锁)，select...for update,update,insert,delete(排他锁)都是一种当前读

####  快照读

简单的select（不加锁）就是快照读，快照读，读取的是记录数据的可见版本，有可能是历史数据，不加锁，是非阻塞读

read committed：每次select，都生成一个快照读

repeatable read：开启事务后第一个select语句才是快照读的地方

serializable：快照读会退化为当前读

#### mvcc

多版本并发控制.指维护一个数据的多个版本,使得读写操作没有冲突,快照读为mysql实现mvcc提供了一个非阻塞读功能.mvcc的具体实现,需要依赖于数据库记录中的三个隐式字段,undo log日志,readview.

三个隐式字段

| 隐藏字段    | 含义                                                         |
| ----------- | ------------------------------------------------------------ |
| db_trx_id   | 最近修改事务id,记录插入这条记录或最后一次修改该记录的事务ID  |
| db_roll_ptr | 回滚指针,指向这条记录的上一个版本,用于配合undo log,指向上一个版本 |
| db_row_id   | 隐藏主键,如果表结构没有指定主键,将会生成该隐藏字段           |

readview

readview是快照读sql执行时mvcc提供数据的依据,记录并维护系统当前活跃的事务(未提交的)id.

readview中包含四个核心字段

| 字段           | 含义                                              |
| -------------- | ------------------------------------------------- |
| m_ids          | 当前活跃的事务id集合                              |
| min_trx_id     | 最小活跃事务id                                    |
| max_trx_id     | 预分配事务id,当前最大事务id+1(因为事务id是自增的) |
| creator_trx_id | readview创建者的事务id                            |

不同的隔离级别,生成readview的时机不同

read committed:在事务中每一次执行快照时生成readview

repeatable read:仅在事务中第一次执行快照读时生成readview,后续复用readview