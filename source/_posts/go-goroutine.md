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

在golang中m默认设置为10000，在初始化的时候在runtime.schedinit()中设置了最大数。
p也有限制，默认是cpu的核心数，在启动时由环境变量$GOMAXPROCS或者是由runtime.GOMAXPROCS()决定，

在任何情况下，go运行时并行执行（不是并发）的goroutinues数量是小于等于P的。
常规项目直接使用默认的核心数就好了，GOMAXPROCS 开太多的时候，针对计算密集型的处理性能提升反而没那么大，IO 密集(或者 syscall 较多)的 Go 程序，至少应该配置到CPU核心数目的5倍以上, 最大1024。

### 优势
1. 与线程相比所占空间小成本低，可以根据应用程序的需要增长和收缩。线程中大小必须指定且固定
2. goroutine被多路复用到较少的os线程上。一个程序中可能只有一个线程和数千个goroutines
3. 当使用goroutines访问共享内存时，通过设计的通道可以防止竞态条件发生。通道可以被认为是goroutines通信的管道

## goroutine并发控制和通信
- 全局共享变量
所有子goroutine共享这个变量，并不断轮询这个变量的变化，主进程更新该全局变量
- channel通信（csp模型）
- context包

