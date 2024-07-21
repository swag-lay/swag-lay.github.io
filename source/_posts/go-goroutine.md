---
title: go_goroutine
date: 2023-12-19 15:52:12
categories:
- golang
- 协程
tags:
---

# go的协程
## goroutine
goroutine是与其他函数或方法同时进行的函数和方法。可以被认为是轻量级的线程，创建goroutine的成本很小，他就是一段代码，一个函数入口。以及在堆上为其分配的一个堆栈（初始大小为4k，会随着程序的执行自动增长删除）。
### 优势
1. 与线程相比所占空间小成本低，可以根据应用程序的需要增长和收缩。线程中大小必须指定且固定
2. goroutine被多路复用到较少的os线程上。一个程序中可能只有一个线程和数千个goroutines
3. 当使用goroutines访问共享内存时，通过设计的通道可以防止竞态条件发生。通道可以被认为是goroutines通信的管道

## goroutine并发控制和通信
- 全局共享变量
所有子goroutine共享这个变量，并不断轮询这个变量的变化，主进程更新该全局变量
- channel通信（csp模型）
- context包
