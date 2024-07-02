---
title: network-http&https
date: 2024-03-25 16:42:30
categories:
- 计算机网络
- http&https
tags:
---

# http & https
## http是如何保存用户的状态
http本身是一种无状态协议，也就是说http协议自身不对请求和响应之间的通信状态进行保存。而session机制就是为了保存用户状态，session通过服务端记录用户的状态。常见保存session的方法有内存和数据库（比如redis），如何session进行跟踪，一般是通过是在cookie中附加一个session id。当cookie被禁用时，通常利用url重写把session id直接附加在url路径后面

## 从输入url到页面展示的过程

![image-20240325164540430](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240325164540430.png)
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

## http标头
http1.1的标头有四种，**通用标头，实体标头，请求标头，响应标头**
- 通用标头
传递有关消息本身的信息，通用标头不会限制于是请求还是响应报文，但是某些通用标头大部分或全部用于一种特定类型的请求中。
1. Cache-Control
2. Connection：keep-alive 持久性连接
3. Date：比北京时间慢八小时
。。。

- 请求标头
1. Accept 告知客户端能够接受的MME类型
2. Accpet-Charset 客户端接受的字符编码
3. Accpet-Encoding 客户端希望服务端返回的内容编码
4. Authorization

- 响应标头
1. Accept-Ranges bytes表示可以处理 none表示不能处理
2. Age 告诉客户端源服务器在多久之前创建了响应，单位秒，通常接近0
3. ETag 对于条件请求非常重要，因为条件请求是根据ETag的值进行匹配的
4. Location 表示url需要重定向页面，仅仅与3xx（重定向）和201（已创建）一起使用
5. Access-Contrl-Allow-Origin 指定一个来源，告诉浏览器允许该来源进行资源访问

- 实体标头
用于http请求和http响应中
实体标头不局限于请求标头或者响应标头
一般Content-开头都是实体标头

## https流程

![image-20240410144518114](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240410144518114.png)

http三次tcp握手
ClientHello：客户端通过服务器发送hello消息来发起握手过程
ServerHello
认证：客户端的证书颁发机构会认证SSL证书，然后发送Certificate报文，报文中包含公开密钥证书。最后服务器发送ServerHelloDone作为hello请求的响应。第一部分握手阶段结束
加密阶段：第一个阶段握手完成后，客户端会发送ClientKeyExchange作为响应，这个响应中包含一种称为The premaster secret的密钥字符串，这个字符串就是使用上面公开密钥证书进行加密的字符串。随后客户端发送ChangeCipherSpec，告诉服务端使用私钥解密这个premaster secret的字符串，然后客户端发送finished告诉服务端自己发送完成
实现安全的非对称加密：然后，服务器发送ChangeCipherSpec和Finished告诉客户端解密完成，至此实现RSA的非对称加密。

## http1.0 1.1 2.0 3.0

![image-20240702120528390](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240702120528390.png)

- http1.0
	无连接，可以使用keep-alive建立长连接，但需要手动
	无法复用连接，每次发送请求都需要进行tcp连接
	队头阻塞，下一个请求必须在前一个请求响应达到之后才能发送
    其他：认为每个服务器绑定唯一一个ip地址，因此在请求消息的url中没有主机名，http1.0没有host域。
    不支持断点续传功能，每次都需要传送全部的页面和数据。
- http1.1
	简单：基本的报文格式是header+body，头部信息是key-value的形式
	灵活易扩展，请求方法url状态码没有固定死
	无状态
	明文传输
	不安全
	长连接
	管道传输：在一个tcp连接中，用户可以发送多个请求，但是服务器必须按照接收请求的顺序发送对这些管道化请求的响应。http1.1虽然解决了请求的队头阻塞（应用层），但是没有解决响应的队头阻塞。但是在传输层（tcp）仍存在队头阻塞，tcp中途有数据断开，丢包，导致http无法从缓冲区中得到数据，依然会队头阻塞，当丢包会触发tcp的超时重传。

- http2.0
	头部压缩
	二进制格式：在1.0和1.1中报文都是以纯文本的格式简单易读，在2.0中采用了二进制格式。报头和数据体称为：帧-》头信息帧和数据帧
	多路复用：能够在一个tcp上进行任意数量的http请求，彻底解决了队头阻塞的问题。
	服务端推送：服务端不再是被动响应，也可以主动向客户端发送消息，推送额外的资源。
## http和https的区别
https就是http+tsl/ssl组合
端口号：http是80，https是443
url前缀：http://,https://
安全性和资源消耗：http运行在tcp上，所有传输内容为明文，客户端和用户端都无法验证对方的身份。https是运行在ssl/tls上的，ssl/tls又是运行在tcp上的。所有传输的内容都经过加密，加密采用对称加密，但对称加密的密钥用服务器的证书进行了非对称加密
搜索引擎优化：搜索引擎通常会更青睐使用https协议的网站，因为https能够提供更高的安全性和用户隐私保护。使用https协议的网站在搜索结果中可能会被优先显示，从而对seo产生影响。

## TLS
由记录协议，握手协议，警告协议，变更密码规范协议，扩展协议等几个子协议组成
比如ECDHE-ECDSA-AES256-GCM-SHA384
基本格式**密钥交换算法-签名算法-对称加密算法-摘要算法**组成，有时候还有分组模式
TLS从根本上使用对称加密和非对称加密两种形式


