---
title: go_gmp
date: 2024-01-29 15:16:43
categories:
- golang
- 并发编程
tags:
---

# GMP模型
![image-20240522165618052](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240522165618052.png)

goroutine的并发编程模型基于gmp模型
G:表示goroutine，每个goroutine都有自己的栈空间，定时器，初始化的栈空间在2k左右，空间会随着需求增长
M:抽象化代表内核线程，记录内核线程栈信息，当goroutine调度到线程时，使用该goroutine自己的栈信息
P:代表调度器，负责调度goroutine，维护一个本地goroutine队列，M从P上获得goroutine并执行，同时还负责部分内存的管理

# 互斥锁Mutex
互斥锁用于提供一种锁定机制，以确保在任何时刻只有一个goroutine在运行代码的关键部分，从而防止竞争条件的发送。

## mutex channels
我们使用互斥锁和channel解决竞争问题。
一般来说当goroutine需要互相通信时使用channel，当只有一个goroutine应该访问代码的关键部分时使用互斥锁