---
title: network-websocket
date: 2024-03-25 18:21:21
categories:
- 计算机网络
- WebSocket
tags:
---

# websocket
基于tcp连接的全双工通信协议，也就是客户端和服务器可以同时发送和接收数据
用于弥补HTTP协议在持久通信上的不足，客户端和服务器仅需握手一次，两者就可以创建持久性的连接，进行双向数据传输
常用于实时任务：聊天，弹幕，协同编辑

## 工程流程
1. 客户端向服务器端发送http请求，请求头中包含**Upgrade:websocket**，和**Sec-WebSocket-Key**等字段。
2. 服务器接收到请求后，进行协议的升级，如果支持WebSocket，回复http 101，响应头包含**Connection:Upgrade**和**Sec-WebSocket-Accept:xxx**等字段，表示成功升级到WebSocket协议
3. 建立双向连接，数据以帧的形式进行传送。（在连接过程中，通过心跳机制保持连接的稳定性和活跃性）
4. 客户端或服务器主动发送一个关闭帧，表示要断开连接，另一方收到后，也会回复一个关闭帧，然后关闭tcp连接。
