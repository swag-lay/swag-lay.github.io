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

## 主要特性
- 互动性
- 即时更新
- 低延迟
- 高兼容性
- 加密传输
- 跨平台

## websocket和tcp的区别和关联
websocket作为运行在tcp之上的高效的通信协议，提供了长期开启客户端和服务器之间双向通信通道，与tcp三次握手过程类似。websocket的握手通过http完成，建立后，通道保持开放以进行数据交换，确保了tcp协议的可靠传输和流量控制

核心区别：
- 建立连接：除了经典的tcp三次握手之外，需要http协议头中的upgrade字段以升级至websocket连接
- 数据传输单位：倾向与以消息为单位传输数据，tcp是字节流
- 数据处理：websocket相对于tcp，添加了数据压缩和消息分片的额外处理
- 实时性：实时交互能力

## websocket怎么保证安全性
- 使用CSRF token，请求令牌等方案保护websocket握手过程，防止websocket握手过程被CSRF攻击所利用
- 使用wss协议，基于tls的websocket
- 硬编码的websocket的url接口，以保证用户的输入无法篡改此url


