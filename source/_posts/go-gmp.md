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

## 复用线程策略
- work stealing机制
当本线程无可用的G时，从其他线程绑定的P偷取G，而不是销毁线程

- hand off机制
当线程M0因为G0发生系统调用阻塞时，线程M0将绑定的P转移给空闲的线程（可以来自缓存池中也可以新建），进而该线程可以继续执行其他的G，这样就可以充分利用CPU。
当G0系统调用完成，线程被唤醒，如果M0此时能获取P，则继续执行G0
当不能获取P，则将G0放入全局队列中，等他其他线程调用，M0加入缓存池中休眠。

## work-stealing调度算法
当当前线程执行完P中的G，P会从全局队列中寻找G执行；如果全局队列为空，则线程选择从其他的P，从他的队列中拿一半的G到自己的队列中执行。

正常下Goroutine执行，以下情况会被阻塞进而执行其他的G
- 用户态阻塞/唤醒
- 系统调用阻塞：上面的hand off机制

# 互斥锁Mutex
互斥锁用于提供一种锁定机制，以确保在任何时刻只有一个goroutine在运行代码的关键部分，从而防止竞争条件的发送。

## mutex channels
我们使用互斥锁和channel解决竞争问题。
一般来说当goroutine需要互相通信时使用channel，当只有一个goroutine应该访问代码的关键部分时使用互斥锁