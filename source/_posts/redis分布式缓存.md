---
title: redis分布式缓存
date: 2024-04-08 11:35:16
categories:
- redis
- 分布式缓存
tags:
---

# 分布式缓存
## 高并发下的分布式缓存
通常都是b/s架构，中间挂一个服务器，后面挂一个数据库
当并发量加大，首先达到瓶颈的应该是数据库，这就会导致客户端的访问速度，访问效率降低
通常采用加缓冲层的方式，**我们把经常访问的数据称之为热点数据，存放在缓存中**

缓存通常包括：
- 本地缓存：Ehcache，Guava Cache，Spring Cache
- 分布式缓存：通常使用redis
- 多级缓存

## redis集群模式
当我们配置多台redis机器，这就涉及到了redis集群，也就是分布式缓存
### 主从备份集群
一台master，多台slave
当客户端去读数据，首先是找slave；当客户端去写数据，找的是master，写完之后master将数据同步给slave。
哨兵模式：哨兵是当主节点挂掉之后，需要监控将从节点切换为主节点，其他从节点跟随新的主节点。
主从备份集群解决不了海量的热点数据存储的问题
### 切片模式集群
将海量的热点数据进行一个切片，切成一块一块，每一块存放在一台redis上
cluster模式：分治，分片的，每一个节点存放的是一部分数据，单个节点挂掉会损失一部分数据，解决的是容量，压力问题。
通过采用一致性hash算法，环

## 缓存击穿，穿透，雪崩
### 缓存击穿
热点数据缓存过期

### 缓存穿透
数据即不在缓存也不再数据库

当大量的数据请求在缓存中都找不到时，都去数据库寻找，导致数据库垮掉
解决方法：在缓存和数据库中加一个过滤器
布隆过滤器：通过hash
### 缓存雪崩
大量缓存数据同时过期或redis故障
redis集群集体宕机，那所有缓存请求的数据都会到数据库，在这一刻数据库支持不住；但一般情况是**缓存里的数据都设置了同一有效时间，此时有效期一到redis会把它们删除，此时请求这些数据都会到数据库，数据库可能也会宕机，这种情况让redis里的数据陆续失效就可以了而不是同时失效。

## 常见的缓存更新策略有哪几种
### Cache Aside Pattern（旁路缓存模式）
使用比较多的一个缓存读写事件，比较适合读请求比较多的场景
Cache Aside Pattern中服务端需要同时维系数据库db和缓存cache，并且是以db的结果为准
读写策略：
写：
1. 先更新db
2. 直接删除cache

读：
1. 从cache中读取，读取到直接返回
2. cache读不到从db中读取数据
3. 将db读取的数据放到cache中

思考数据不一致问题
一般采用先更新数据库再更新缓存的方式，因为缓存的写入是快于数据库的。有时候会出现删除缓存失败的问题，这个时候我们一般采用
- 用重试机制，通过将删除操作放入消息队列，当失败的时候从队列中取操作，删除成功后移除消息队列中的操作
- 订阅mysql binlog 再操作缓存

### Read/Write Through Pattern（读写穿透）
读写穿透中服务端将cache视为主要数据存储，从中读取数据并将数据写入其中，cache服务负责将此数据读取和写入db，从而减轻了应用程序的职责
写：
1. 先查cache，cache中不存在，直接更新db
2. cache中存在，则先更新cache，然后cache服务自己更新db（同步更新cache和db）

读：
1. 从cache中读取数据，读取到则返回
2. 读取不到从db中加载，写入cache后返回响应

### Write Behind Pattern异步缓存写入
只更新缓存，不直接更新db，而是改为异步批量的方式来更新db

