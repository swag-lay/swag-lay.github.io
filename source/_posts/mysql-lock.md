---
title: mysql-lock
date: 2024-04-16 15:43:18
categories:
- 数据库
- mysql
tags:
---

# 锁的分类
- 全局锁：ftwrl
- 表级锁
1. 表锁
2. 元数据锁
3. 意向锁
4. auto-inc锁
- 行级锁
- record lock
- gap lock
- next-key lock

# mysql怎么加锁
## 行级锁的加入
innodb是默认支持行级锁的
普通的select不会对记录加锁（除非串行化隔离级别），因为它属于快照读，是通过mvcc实现
如果要在查询时对记录加行级锁，可以通过以下两种方式，这两种查询会加锁的语句称为锁定读
```mysql
// 对读取的记录加共享锁（s锁）
select ... lock in share mode;

// 对读取的记录加独占锁（x锁）
select ... for update;
```
update和delete也会加行级锁，且锁的类型都是独占锁（x锁）
s锁满足读读共享，读写互斥，x锁满足写写互斥，读写互斥

## mysql怎么加行级锁
**加锁的对象是索引，加锁的基本单位是next-key lock，next-key lock是前开后闭区间，而间隙锁是前开后开区间**
**当使用记录锁或者间隙锁能够避免幻读的场景下，next-key lock就会退化为记录锁或间隙锁**
用select * from performance_schema.data_locks\G;可以查看加锁的过程
如果LOCK_MODE为x，说明是next-key锁
如果LOCK_MODE为x,REC_NOT_GAP，说明是记录锁
如果LOCK_MODE为x,GAP，说明是间隙锁

![img](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/%E5%94%AF%E4%B8%80%E7%B4%A2%E5%BC%95%E5%8A%A0%E9%94%81%E6%B5%81%E7%A8%8B.jpeg)

![img](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/%E9%9D%9E%E5%94%AF%E4%B8%80%E7%B4%A2%E5%BC%95%E5%8A%A0%E9%94%81%E6%B5%81%E7%A8%8B.jpeg)
**当在update语句的where条件没有使用索引，就会全表扫描，于是就会给所有记录加上next-key锁，相当于把整张表锁住了**

那update语句的where带上索引就能避免全表记录加锁了吗？
并不是，**关键还得看这条语句在执行过程中，优化器最终选择的是索引扫描还是全表扫描，如果走了全表扫描，就会对全表的记录加锁了**

两个事务即使生成的间隙锁的范围是一样的，也不会发生冲突，因为间隙锁目的是为了防止其他事务插入数据，因此间隙锁与间隙锁之间是相互兼容的。
在执行插入语句时，如果插入的记录在其他事务持有间隙锁范围内，插入语句就会被阻塞，因为插入语句在碰到间隙锁时，会生成一个插入意向锁，然后插入意向锁和间隙锁之间是互斥的关系。
如果两个事务分别向对方持有的间隙锁范围内插入一条记录，而插入操作为了获取到插入意向锁，都在等待对方事务的间隙锁释放，于是就造成了循环等待，满足了死锁的四个条件：互斥、占有且等待、不可强占用、循环等待，因此发生了死锁。