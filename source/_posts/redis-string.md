---
title: redis-string
date: 2024-07-17 16:49:03
categories:
- redis
tags:
---

# string的实现
redis的string是动态字符串，是可以修改的字符串，内部结构实现上类似于java的arraylist，采用预分配冗余空间的方式来减少内存的频繁分配。
- 一般字符串分配的空间capacity一般高于实际字符串长度len
- 字符串长度小于1M的时候，扩容都是加倍现有的空间
- 超过1M，扩容时一次只会多扩1M的空间
- 字符串最大长度为512M
- 字符串是由多个字节组成，每个字节由8个bit组成，也就是说一个字符串看成多个bit的组合


## string的使用场景
计数，缓存基础数据，限制请求次数，分布式下共享session，签到

分布式下共享session：
用redis将用户的session信息进行集中的管理，每个用户登录都从redis中集中获取
session除了redis存储外，
还可以采用session会话保持：用户每次请求都在同一台机器上，
session会话复制：每个应用服务器中的session信息复制同步到其他服务器节点中