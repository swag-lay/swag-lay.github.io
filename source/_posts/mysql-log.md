---
title: mysql-log
date: 2023-07-22 16:14:25
categories:
tags:
---

# undo log
回滚日志 保证原子性
在每次执行事物的时候记录undo log，当事务崩溃或者回滚的时候执行回滚事务

每一次参数的undo log格式都有一个roll_pointer和一个trx_id事务id
通过trx_id直到该记录被哪个事务修改
通过roll_pointer将undo log连成链表

undo log+read view可以实现mvcc

刷盘：
buffer pool中由undo页，对undo页的修改也都会记录到redo log中，redo log每秒会刷盘，提交事务也会刷盘，数据页和undo页都是靠这个机制保证持久化的。

# redo log
重做日志 保证持久性
防止机器断电导致数据丢失，当一条记录需要更新时，innodb就会先更新内存（同时标记为脏页），（为了减少磁盘io，不会立即将脏页写入磁盘，后续由后台线程选择合适的时机将脏页写入磁盘），然后对这个页的修改会以redo log形式记录下来，**这个时候更新完成**
前面提到的由后台线程写入磁盘就是WAL技术

当事务提交之后，只要直接将redo log持久化到磁盘，就不用等脏页持久化了。
当修改undo页面的时候，也需要记录redo log，例如在开启事务后，innodb更新记录前，首先要记录相应的undo log，如果是更新操作，需要把被更新的列的旧值记录下来，生成undo log，undo log 写入undo页面，在内存修改undo 页面后，记录redo log

redo log也有redo log buffer（默认16mb），
刷盘时机：
- mysql正常关闭
- 当redo log buffer中记录的写入量大于一半时，发生落盘
- 后台线程每隔1s，持久化一次
- 事务提交时

redo log采用循环写的方式，write pos要写的位置，check point要擦除的位置。
当write pos追上check point意味着redo log满了，这个时候mysql会被阻塞，此时会停下来将buffer pool中的脏页刷新到磁盘，然后标记哪些redo log记录会被擦除，当擦除完，checkpoint往后移动 mysql恢复正常

![image-20240722164924965](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240722164924965.png)

# binlog
二进制日志，server层生成
当完成一条更新操作后，server层生成一条binlog，当事务提交，统一将binlog写入binlog文件
记录了所有数据库表结构变更和表结构修改的日志，不会记录查询类的操作，比如select和show操作
binlog是追加写，写完一个文件，创建一个新文件继续写，不会覆盖之前的日志。
主要用于备份，主从复制