---
title: redis分布式锁
date: 2024-03-29 15:00:16
categories:
- 分布式微服务
- redis分布式锁
tags:
---

# redis实现分布式锁

## redis分布式锁的意义
为了解决多机资源竞争问题，保证在同一时刻只能有一个机器或者线程使用某一资源。常用于定时任务，超卖等问题。利用了redis的原子性操作，结合分布式，实现对锁的获取，释放和操作。
通常包括原子性，超时处理，可重入，竞态问题
- 原子性利用setnx，setex，保证多个机器获取锁只有单个机器能够获取成功 ，保证资源的互斥。
- 超时通过设置过期时间，避免长时间占用，同时我们往往利用看门狗机制设置间隔对锁的过期时间进行延长，保证业务完成，防止出现锁过期而业务没有完成的现象。
- 可重入是保证同一机器多次获取锁，此处是为了防止死锁的发生，不会让自己阻塞自己。我们往往通过设置value+1代表获取锁的次数，锁尝试释放value-1，直到value=0，才能释放锁。这里设置value=0才释放是为了保障释放锁的完整性，如果value每次-1就直接释放，当每次value-1的时候该线程其他代码可能仍然需要这个锁，如果其他线程直接获取该锁，导致数据出现竞态问题。
- 竞态问题往往通过重试机制，或者公平锁的实现，保障客户端都有机会获取锁。

## redis分布式锁的问题
当在哨兵模式下，当一个线程已经对master节点获取锁了，master在对从节点进行异步复制的过程中，发生了宕机，从节点切换为master，此时另一个线程又对master获取锁，导致出现多个线程对同一个分布式锁完成了加锁，出现脏数据的产生。
RedLock可以解决，设置需要在多数机器上同时加锁，但一般不用，成本太高。
常用zookeeper实现的分布式锁解决这个问题。

## 方案一：setnx+expire
setnx key value，如果key不存在，则setnx成功返回1，如果这个key以及存在，则返回0
比如电商的秒杀活动，key可以设置为key_resource_id，value设置为任意值。
```c#
if（jedis.setnx(key_resource_id,lock_value) == 1）{ //加锁
    expire（key_resource_id，100）; //设置过期时间
    try {
        do something  //业务请求
    }catch(){
  }
  finally {
       jedis.del(key_resource_id); //释放锁
    }
}
```
此时并没有保证原子性，当设置过期时间时，进程crash或者重启了，那么这个锁就永远存在，**导致别的锁永远获取不到锁**。

## 方案二：setnx+value值是（系统时间+过期时间）
直接将过期时间值放入setnx中的value中，当加锁失败，再拿出value值校验一下即可
```c#
long expires = System.currentTimeMillis() + expireTime; //系统时间+设置的过期时间
String expiresStr = String.valueOf(expires);

// 如果当前锁不存在，返回加锁成功
if (jedis.setnx(key_resource_id, expiresStr) == 1) {
        return true;
} 
// 如果锁已经存在，获取锁的过期时间
String currentValueStr = jedis.get(key_resource_id);

// 如果获取到的过期时间，小于系统当前时间，表示已经过期
if (currentValueStr != null && Long.parseLong(currentValueStr) < System.currentTimeMillis()) {

     // 锁已过期，获取上一个锁的过期时间，并设置现在锁的过期时间（不了解redis的getSet命令的小伙伴，可以去官网看下哈）
    String oldValueStr = jedis.getSet(key_resource_id, expiresStr);
    
    if (oldValueStr != null && oldValueStr.equals(currentValueStr)) {
         // 考虑多线程并发的情况，只有一个线程的设置值和当前值相同，它才可以加锁
         return true;
    }
}
        
//其他情况，均返回加锁失败
return false;
}
```
问题：
- 分布式环境下客户端时间必须同步
- 当锁过期的时候，并发多个客户端同时请求过来，都执行getSet()，最终只能有一个客户端加锁成功，但是该客户端的过期时间，可能被别的客户端覆盖。
- 锁没有保存持有者的唯一标识，可能被别的客户端释放/解锁。

## 使用lua脚本（setnx+expire）
lua脚本保证原子性
```vb
if redis.call('setnx',KEYS[1],ARGV[1]) == 1 then
   redis.call('expire',KEYS[1],ARGV[2])
else
   return 0
end;
```
```javascript
 String lua_scripts = "if redis.call('setnx',KEYS[1],ARGV[1]) == 1 then" +
            " redis.call('expire',KEYS[1],ARGV[2]) return 1 else return 0 end";   
Object result = jedis.eval(lua_scripts, Collections.singletonList(key_resource_id), Collections.singletonList(values));
//判断是否成功
return result.equals(1L);

```

## set的扩展命令（set ex px nx）
set key value [EX seconds]  [PX milliseconds] [nx|xx]
```c# 
if（jedis.set(key_resource_id, lock_value, "NX", "EX", 100s) == 1）{ //加锁
    try {
        do something  //业务处理
    }catch(){
  }
  finally {
       jedis.del(key_resource_id); //释放锁
    }
}
```
问题
- 锁过期释放，业务没有执行完成。
- 锁被其他的线程误删

## set ex px nx + 校验唯一随机值，再删除
为解决误删问题，给value值设置一个标记当前线程唯一的随机数，在删除的时候进行校验
```c#
if（jedis.set(key_resource_id, uni_request_id, "NX", "EX", 100s) == 1）{ //加锁
    try {
        do something  //业务处理
    }catch(){
  }
  finally {
       //判断是不是当前线程加的锁,是才释放
       if (uni_request_id.equals(jedis.get(key_resource_id))) {
        jedis.del(lockKey); //释放锁
        }
    }
}
```
同样在判断唯一和释放的时候，并不是原子性
一般也采用lua脚本执行
```vb
if redis.call('get',KEYS[1]) == ARGV[1] then 
   return redis.call('del',KEYS[1]) 
else
   return 0
end;
```

## redisson框架

![6401](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/640(1).png)

只要线程一加锁成功，就会启动一个`watch dog`看门狗，它是一个后台线程，会每隔10秒检查一下，如果线程1还持有锁，那么就会不断的延长锁key的生存时间。因此，Redisson就是使用Redisson解决了**锁过期释放，业务没执行完**问题。

## 多机实现分布式锁Redlock+Redisson

![640 (1)](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/640%20(1).png)

如果线程一在Redis的master节点上拿到了锁，但是加锁的key还没同步到slave节点。恰好这时，master节点发生故障，一个slave节点就会升级为master节点。线程二就可以获取同个key的锁啦，但线程一也已经拿到锁了，锁的安全性就没了。

为了解决这个问题，Redis作者 antirez提出一种高级的分布式锁算法：Redlock。Redlock核心思想是这样的：

搞多个Redis master部署，以保证它们不会同时宕掉。并且这些master节点是完全相互独立的，相互之间不存在数据同步。同时，需要确保在这多个master实例上，是与在Redis单实例，使用相同方法来获取和释放锁。



我们假设当前有5个Redis master节点，在5台服务器上面运行这些Redis实例。

![640 (2)](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/640%20(2).png)

RedLock的实现步骤:如下

> - 1.获取当前时间，以毫秒为单位。
> - 2.按顺序向5个master节点请求加锁。客户端设置网络连接和响应超时时间，并且超时时间要小于锁的失效时间。（假设锁自动失效时间为10秒，则超时时间一般在5-50毫秒之间,我们就假设超时时间是50ms吧）。如果超时，跳过该master节点，尽快去尝试下一个master节点。
> - 3.客户端使用当前时间减去开始获取锁时间（即步骤1记录的时间），得到获取锁使用的时间。当且仅当超过一半（N/2+1，这里是5/2+1=3个节点）的Redis master节点都获得锁，并且使用的时间小于锁失效时间时，锁才算获取成功。（如上图，10s> 30ms+40ms+50ms+4m0s+50ms）
> - 如果取到了锁，key的真正有效时间就变啦，需要减去获取锁所使用的时间。
> - 如果获取锁失败（没有在至少N/2+1个master实例取到锁，有或者获取锁时间已经超过了有效时间），客户端要在所有的master节点上解锁（即便有些master节点根本就没有加锁成功，也需要解锁，以防止有些漏网之鱼）

简化下步骤就是：

- 按顺序向5个master节点请求加锁
- 根据设置的超时时间来判断，是不是要跳过该master节点。
- 如果大于等于3个节点加锁成功，并且使用的时间小于锁的有效期，即可认定加锁成功啦。
- 如果获取锁失败，解锁！s