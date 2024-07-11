---
title: network-spdy
date: 2024-07-11 15:17:09
categories:
- 计算机网络
- spdy
tags:
---

# SPDY
google开发的基于tcp的应用层协议，在于通过压缩，多路复用和优先级来缩短网页的加载时间和提高安全性。

![image-20240711151904587](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240711151904587.png)
在性能上对http进行优化，核心在于尽量减少连接个数，对于http头部并没有太大的修改。具体，spdy使用http的方法和页眉，但是删除了一些头并重写了http中管理连接和数据转移格式的部分，所以基本上是兼容http的。

目前google只是在ssl之上增加会话层来实现spdy，http的请求格式保持不变，即现有的所有服务端应用均不用做任何修改。因此，目前spdy目的是为了加强http。默认规定建立在tls上个，即url secheme为https

优点
1. 多路复用 请求优化
在一个spdy连接中可以无限个并发请求，允许多个并发http请求共用一个tcp会话。
多路复用可以设置优先级，不像传统http那样严格按照先入先出一个个处理。
2. 支持服务器推送技术
服务器可以主动向客户端发起通信 向客户端推送数据，这样整个网络一直保持一个快速状态。
3. spdy压缩了http头
舍去不必要的头信息。
4. 强制使用ssl

![spdy_session](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/spdy_session.png)