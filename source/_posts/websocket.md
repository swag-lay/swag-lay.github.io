---
title: websocket
date: 2024-05-21 11:13:45
categories:
- 计算机网络
- websocket
tags:
---

# WebSocket
一次握手就能使客户端和服务端建立长连接，并进行双向传输
与http相比websocket协议每次数据传输的头信息都比较小，节约带宽

在springboot项目中，一般采用@ServerEndPoint+@Component
@ServerEndpoint 该注解可以将类定义成一个WebSocket服务器端
@OnOpen 表示有浏览器链接过来的时候被调用
一般将建立的session信息存入会话对象中进行保存。
@OnClose 表示浏览器发出关闭请求的时候被调用
@OnMessage 表示浏览器发消息的时候被调用
@OnError 表示报错了