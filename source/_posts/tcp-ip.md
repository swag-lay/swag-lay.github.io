---
title: tcp/ip
date: 2024-03-26 13:26:23
categories:
- 计算机网络
- tcp/ip协议
tags:
---

# tcp三次握手
**刚开始客户端处于closed的状态，服务端处于listen状态**
第一次握手，客户端发送syn报文，指明序列号为lsn，此时客户端处于syn_send状态
第二次握手，服务器收到syn报文，发送syn报文给客户端，序列号为lsn，将客户端你的lsn+1作为ack，此时服务器处于syn_recv状态
第三次握手，客户端收到syn报文，将lsn+1作为ack发送给服务端，此时客户端处于establised状态
服务器收到ack报文，处于establised状态，双方建立连接

# tcp四次挥手
**刚开始双方都处于establised状态**
第一次挥手，客户端发送fin报文，指定序列号，此时客户端处于fin_wait1状态
第二次挥手，服务端接收fin之后，发送ack报文，序列号为客户端序列号+1，此时服务端处于close_wait状态
第三次挥手，此时服务端也想要断开连接，发送fin报文，指定序列号，此时服务端处于last_ack状态
第四次挥手，客户端收到fin之后，将序列号+1作为ack发送给服务端，客户端此时处于time_wait状态，需要过2msl之后确保服务端收到ack报文才进入closed状态
服务端接收ack报文之后关闭连接进入closed状态

其中第四次挥手为什么要等待2msl
第四次挥手时，客户端发送给服务器的ack有可能丢失，如果服务端因为某些原因而没有收到ack的话，服务端就会重发fin，如果客户端在2msl时间内收到了fin，就会重新发送ack并再次等待2msl，防止server没有收到ack而不断重发fin