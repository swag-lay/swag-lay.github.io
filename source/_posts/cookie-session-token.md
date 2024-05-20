---
title: cookie-session-token
date: 2024-04-10 15:27:24
categories:
- 计算机网络
- cookie&session&token
tags:
---

# Cookie和Session
因为http是无状态的协议，所以服务器不知道客户端的历史请求记录，session和cookie的目的是为了弥补http的无状态特性

## session
保存会话
客户端请求服务器，服务器会开辟一块内存空间，这个对象便是session对象，存储结构是ConcurrentHashMap，弥补了http的无状态特性，服务器可以利用session存储客户端在同一个会话期间的一些操作记录

session判断是否是同一会话

![image-20240410171934809](..\images\image-20240410171934809.png)
第一次请求开辟内存空间，设置sessionId，通过响应头的Set-Cookie：JSESSIONID=xxxxxx，向客户端发送设置cookie的响应，客户端设置cookie，该cookie的过期时间为浏览器会话结束
接下来每一次请求都会携带cookie（包含sessionId）

缺点：当一台服务器A存储Session，做了负载均衡之后，加入一段时间内A服务器的访问量激增，转发到B访问，此时B里面并没有Session
## cookie
http协议的cookie包括Web Cookie和浏览器Cookie，他是服务器发送到web浏览器的一小块数据。服务器发送到浏览器的Cookie，浏览器会存储，并与下一个请求一起发送到服务器。通常用于判断两个请求是否来自同一个浏览器，例如用户保持登录状态
三个目的：
- 会话管理
- 个性化
- 跟踪
两种类型cookies，一种session cookies，一种persistent cookies
session cookies：会话cookies，不包含到期时间，存储在内存中，不会写入磁盘，浏览器关闭cookie消失

## jwt
**用于json对象在各方之间安全的传输信息**，此信息可以进行验证和信任，因为他是经过数字签名的。jwt可以使用机密（HMAC算法）或使用RSA或ECDSA的公钥/私钥对进行签名。虽然可以对jwt进行加密，以便在各方之间提供保密性，但是我们将关注已签名的token，签名token可以验证其中包含的声明的完整性，而加密token可以向其他方隐藏这些声明。当使用公钥/私钥对对令牌进行签名时，该签名还证明只有持有私钥的一方才是对其进行签名的一方（**签名技术是保证传输的信息不被篡改，并不能保证信息传输的安全**）
作用
- 认证
- 信息交换

Header：令牌类型和签名算法
```json
{
	"alg":"HS256",
	"typ":"JWT"
}
```
Payload：有关实体（通常是用户）和其他数据的声明
Signature：签证信息包括
- header（base64之后）
- payload（base64之后）
- secret

### jwt如何工作的

![image-20240520155050919](F:\swag-lay.github.io\source\_posts\cookie-session-token\image-20240520155050919.png)
- 客户端向服务器请求授权
- 授予授权后，服务器向应用程序返回访问令牌
- 应用程序使用访问令牌访问

jwt具有加密签名，session cookies没有；
无状态，声明被存储在客户端，而不是服务器内存
可扩展性更强
支持跨越认知