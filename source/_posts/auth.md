---
title: auth
date: 2024-03-05 16:42:08
categories:
- 系统设计
- 认证授权
tags:
---
# 认证授权基础
## 认证和授权的区别
认证：你是谁？验证身份的凭证，通过凭证，系统就知道你是你，存在你这个用户
授权：你有权限干什么？发生在认证之后，掌管我们访问系统的权限。

## cookie
cookie存在客户端，一般用来保存用户信息。
**cookie和session有什么区别？**
session的作用是通过服务端记录用户的状态。http是无状态的，系统不知道用户的操作情况，服务端给特定的用户创建特定的session之后就可以标识这个用户并且跟踪这个用户。
cookie数据保存在客户端（浏览器端），session数据保存在服务器端。相对来说session安全性更高。如果使用cookie的一些铭感信息不要写入cookie中，最好能够将cookie信息加密然后使用到的时候再去服务端解密。
使用session的时候要注意以下几个点：
- 依赖session的关键业务一定要确保客户端开启了cookie
- 注意session的过期时间

# jwt
基于token的认知授权机制，是一种规范化之后的json结构的token。
jwt本质上是一组字串，通过(.)切分成三个为Base64编码的部分：
- header：描述jwt的元数据，定义了生成签名的算法以及token的类型
- payload：用来存放实际需要传递的数据（部分默认是不加密的，**不要将隐私信息存放在其中**）
- signature（签名）：服务器通过payload，header和一个密钥（secret）使用header里面指定的签名算法（默认是hmac sha256）生成。
**jwt安全的核心在于签名，签名安全的核心在密钥**