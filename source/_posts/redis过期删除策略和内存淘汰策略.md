---
title: redis过期删除策略和内存淘汰策略
date: 2024-04-18 17:52:18
categories:
- redis
tags:
---

# redis过期删除策略
当我们对key设置过期时间之后，我们需要对已过期的键值对进行删除，而做这个工作的就是过期键值删除策略

而redis如何判断key已经过期是通过将key带上过期时间存储到一个过期字典中
```c++
typedef struct redisDb{
	dict *dict;//存key value
	dict *expire;
}redisDb;
```
当我们查询某个key-value的时候，如果不在过期字典也就是没设置过期时间那肯定是正常的，当在过期字典中，我们看当前系统时间是否小于过期时间，小于键值对没有过期，反之则过期。

redis采用**惰性删除+定期删除**两种方式配合。

![img](..\images\过期删除策略.jpg)

# redis内存淘汰策略
当redis内存消耗过高，需要使用内存淘汰策略删除符合条件的key，以此保障redis高效运行。

进行内存淘汰主要分为两种
- 不进行数据淘汰
- 进行数据淘汰

1. 在设置了过期时间的数据中进行淘汰
- 随机淘汰
- ttl淘汰更早过期的
- redis3.0前才用lru
- redis4.0后新增lfu
2. 在所有数据范围内进行淘汰
同上

![img](..\images\内存淘汰策略.jpg)