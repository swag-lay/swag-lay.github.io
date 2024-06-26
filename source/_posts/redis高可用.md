---
title: redis高可用
date: 2024-04-19 13:55:40
categories:
- redis
tags:
---

# redis主从复制
主从服务器之间采用的是读写分离的方式
首先客户端可以读写到主服务器，也可以读从服务器，然后主服务器同步写操作给从服务器。

主从复制第一次同步的过程：
- 建立连接，协商同步
- 执行bgsave命令（开辟的新进程）生成rdb文件同步数据给从服务器
- 缓冲区执行新的写命令，发送新写命令给从服务器（这里不断传输写命令是建立在长连接上的命令传播）

为了减少主服务器的压力，我们可以设置经理服务器，也可以称为二级服务器，由主服务器依次到各级服务器的顺序进行同步操作

当网络中途断开的时候，复制的过程采用增量复制，根据redis中设置的参数将这段时间内的操作进行写入

## 哨兵机制
我们通过设置哨兵节点检测主从节点，不断向节点发送ping命令，判断其是否正常运行
为了排除主节点压力过大或网络原因没有响应，我们一般设置哨兵集群（最少三台）。