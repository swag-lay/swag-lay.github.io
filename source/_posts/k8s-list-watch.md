---
title: k8s-list-watch
date: 2023-09-27 16:37:52
categories:
- 云原生
- k8s
tags:
---

k8s的内部通信机制，所有组件的通信都采用http协议通信，不依赖中间件。
在内部通信中，为了保证消息的实时性，如果采用轮询会大大加大api-server的负担，同时实效性不高。那么如果apiserver发起http请求，怎么保证消息可靠呢，端口又怎么解决大量占用问题呢。归结于k8s的list-watch机制。

# list-watch
客户端（controller/scheduler）通过list-watch监听apiserver中资源的crud，并对事件调用回调函数处理。
list部分：罗列资源，基于http短链接
watch：监听资源的变更事件，采用http长连接。采用分块传输编码，首次出现在http/1.1中，对于动态生成的内容来说，在内容创建完之前是不可知的，使用分块传输编码，数据分解成一系列数据块，并以一个或多个块发送，这样服务器可以发送数据而不需要预先知道发送内容的总大小。

## informer模块
比如从statefulset的创建中，所有的资源都会放到informer中监控，而informer正好封装了list-watch api，informer会通过list罗列资源，并且调用watch监听资源变更，将结果放入一个fifo队列中，协程会从队列中取出事件，调用注册函数处理。