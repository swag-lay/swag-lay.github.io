---
title: network-http
date: 2024-03-25 16:42:30
categories:
- 计算机网络
- http
tags:
---

# http & https
## http是如何保存用户的状态
http本身是一种无状态协议，也就是说http协议自身不对请求和响应之间的通信状态进行保存。而session机制就是为了保存用户状态，session通过服务端记录用户的状态。常见保存session的方法有内存和数据库（比如redis），如何session进行跟踪，一般是通过是在cookie中附加一个session id。当cookie被禁用时，通常利用url重写把session id直接附加在url路径后面

## 从输入url到页面展示的过程

![image-20240325164540430](..\images\image-20240325164540430.png)
1. 通过dns协议，获取域名对应ip
2. 根据ip和端口，向目标服务器发送tcp请求
3. 在tcp连接上，向服务器发送http请求报文，获取网页内容
4. 服务器接收http请求报文，处理请求，返回响应报文
5. 浏览器收到http响应报文，解析响应体的html代码，渲染网页，如果有需要加载的其他资源再次发送http请求，获取资源，直到网页全部加载
6. 不需要通信时，关闭tcp连接

http状态码

|状态码|类别|原因|
|---|---|
|1xx |information |接收请求正在处理|
|2xx|success|完成|
|3xx|redirection|重定向，需要附加操作以完成|
|4xx|client error|服务器无法处理|
|5xx|server error|服务器请求错误|

## http和https的区别
端口号：http是80，https是443
url前缀：http://,https://
安全性和资源消耗：http运行在tcp上，所有传输内容为明文，客户端和用户端都无法验证对方的身份。https是运行在ssl/tls上的，ssl/tls又是运行在tcp上的。所有传输的内容都经过加密，加密采用对称加密，但对称加密的密钥用服务器的证书进行了非对称加密
搜索引擎优化：搜索引擎通常会更青睐使用https协议的网站，因为https能够提供更高的安全性和用户隐私保护。使用https协议的网站在搜索结果中可能会被优先显示，从而对seo产生影响。
