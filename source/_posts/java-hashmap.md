---
title: java-hashmap
date: 2023-06-26 11:05:11
categories:
- java
- HashMap
tags:
---

# HashMap基本概念
1. 非线程安全
2. 支持存储null的key和value，为null的key只有一个，为null的value可以多个
3. 默认容量为16，每次扩容为之前的两倍、
4. java1.8之前都是数组+链表，在java1.8之后，当解决哈希冲突的时候，当链表长度大于阈值（默认8），链表变为红黑树（需要保证数组长度大于64，否则先扩容）

## HashMap的线程问题
首先我们需要知道HashMap放置value的逻辑，通过key的hashcode经过扰动函数处理的到hash值，然后通过（n-1）&hash判断当前元素存放的位置（n为数组的长度），如果当前位置存在元素，判断加入元素的hash值和该元素以及key是否相同，如果相同直接覆盖，不同通过拉链法解决冲突。
```java
//java 1.8
    static final int hash(Object key) {
      int h;
      // key.hashCode()：返回散列值也就是hashcode
      // ^：按位异或
      // >>>:无符号右移，忽略符号位，空位都以0补齐
      return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
  }
```

在java1.7之前，多线程下，当一个节点有多个元素需要进行扩容，多个线程对链表进行操作，因为采用的头插法，所以可能导致链表中的节点指向错误，比如链表变为环状，此时查询操作就会陷入死循环。
而java1.8采用尾插法解决环状问题，当时多线程下还是可能存在数据覆盖的问题，因为多线程下往往使用ConcurrentHashMap