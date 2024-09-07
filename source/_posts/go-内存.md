---
title: go-内存
date: 2023-09-05 11:31:29
categories:
- golang
tags:
---

# golang的内存分配

![image-20240905135030152](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240905135030152.png)

arena：存放最终的对象，每一块分为8k大小，每一块称为page。同时多个page组合起来叫mspan（golang中内存管理的基本单元）

bitmap：存放arena中对象的一些信息，包括对象的gc标志位，标识这个对象是否包含指针。这个主要是用于golang在进行垃圾回收的时候根据可达性分析来确定一个对象是否可以被回收，同时采用的是三色标记法进行标记对象。

spans：保存了mspan的指针。管理大房间

## 分配过程

![image-20240905135655604](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240905135655604.png)

1.大对象： >32KB 的对象，直接从heap上分配
2.小对象： 16B < obj <= 32KB 计算规格在mcache中找到合适大小的mspan进行分配（你有多大就住多大的房子竟可能的不要浪费房子的空间）
3.微小对象： <=16B 的对象使用mcache的tiny分配器分配；（如果将多个微小对象组合起来，用单块内存（object）存储，可有效减少内存浪费。）
秉持原则：给到最合适的大小，然后能凑一起的凑一起挤一挤