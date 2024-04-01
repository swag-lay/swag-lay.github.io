---
title: go-context
date: 2024-04-01 17:29:31
categories:
- golang
tags:
---

# context详解
go1.7中引入的context类型。系统中可以使用context传递请求范围的元数据，例如不同函数，线程甚至进程的请求ID。
通过携带键值对用于传输过程和API请求相关的数据

通过http传递当前context，则需要把自己序列化为context。在接收端，解析传入的请求并将值放入当前context中。一旦我们需要跨越线程传播context，需要进行手动操作以将context置于同一条线上。同时，将其从传播线路上解析到接收端的context。
在http中，我们可以将请求ID转储为header，大多数context元数据可以作为header传输。