---
title: mysql-storage-engine
date: 2023-08-29 09:37:33
categories:
- 数据库
tags:
- mysql
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

### MyISAM

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