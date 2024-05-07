---
title: redis_mysql数据一致性
date: 2024-05-07 10:25:18
categories:
- redis
- 数据一致性保证
tags:
---

# 保证redis和mysql的数据一致性

存在问题的方案：
1. 先写mysql再写redis
问题：当高并发场景下，多个mysql执行，某条mysql执行完写入redis迟缓导致数据更新不一致
2. 先写redis再写mysql
依然存在写入出现延迟或者迟缓现象导致数据不一致
3. 先删除redis再写mysql
当写入mysql之前出现读取操作 导致redis中依然存入之前的信息，数据没有即使更新到mysql导致数据不一致

以上问题都可以归结于写读操作的并发，无法保证顺序

## 可行性方案
1. 删除redis，写入mysql，删除redis（缓存双删）
2. 可以采用消息队列的异步或者串行实现最后一次缓存删除，不要无脑sleep线程，同时删除缓存失败可以增加重试机制
3. 写mysql，再删redis（对于不是强一致性的业务可以，如果是秒杀，库存啥的不可以；比如写入mysql之前有查询操作，并且重写redis发生在删除redis之后导致实际没有删除redis）
4. 写mysql，通过binlog异步更新redis
监听binlog，将binlog相关消息推送给redis，通过顺序消费队列和重试机制进行更新；但依然存在问题，如果中途有请求查询数据，缓存没有数据不会出错，但缓存有数据会直接读取导致数据不一致。
