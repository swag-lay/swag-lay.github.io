---
title: transaction_sql
date: 2023-08-28 17:00:31
categories:
- 数据库
- mysql
tags:

---

# 事务

事务是一组操作的集合，它是一个不可分割的工作单位，事务会把所有的操作作为一个整体向系统提交或撤销操作请求，即这些操作要么同时成功，要么同时失败。

## 事务操作

查看/设置事务提交方式

```mysql
select @@autocommit;
set @@autocommit=0;#设置为手动改提交
```

提交事务

```mysql
commit;
```

回滚事务

```mysql
rollback；
```

开启事务

```mysql
start transaction 或 begin;
```

## 事务四大特性

原子性（Atomicity）：事务是不可分割的最小操作单元，要么全部成功，要么全部失败。

一致性（Consistency）：事务完成时，必须使所有的数据都保存一致状态。

隔离性（Isolation）：数据库系统提供的隔离机制，保证事务在不受外部并发操作影响的独立环境下允许。

持久性（Durability）：事务一旦提交或回滚，它对数据库中的数据的改变就是永久的。

## 并发事务问题

脏读：一个事务读到另一个事务还没有提交的数据

不可重复读：一个事务先后读取同一条记录，但两次读取的数据不同，称为不可重复读

幻读：一个事务按照条件查询数据时，没有对应的数据行，但是在插入数据时，又发现这行数据已经存在，好像出现了幻读

## 事务隔离级别

| 隔离级别                | 脏读 | 不可重复读 | 幻读 |
| ----------------------- | ---- | ---------- | ---- |
| read uncommitted        | √    | √          | √    |
| read committed          | ×    | √          | √    |
| repeatable read（默认） | ×    | ×          | √    |
| serializable            | ×    | ×          | ×    |

```mysql
-- 查看事务隔离级别
select @@transaction_isolation;

-- 设置事务隔离级别
set [session|global] transaction isolation level {read uncommitted | read committed | repeatabel read | serializable}
```

事务隔离级别越高，数据越安全，但安全越低。