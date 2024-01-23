---
title: tomcat+nginx
date: 2023-12-26 13:28:01
categories:
- 服务器
tags:
---
# Tomcat介绍
## web容器
早期web应用主要用于浏览静态页面，http服务器（apache，nginx）向浏览器返回静态html，浏览器解析html，将结构呈现给用户
servlet技术简单理解为运行在服务端的java小程序，但是servlet没有main方法，不能独立运行，因此必须把它部署到servlet容器中，由容器来实例化并调用 servlet
tomcat就是一个servlet容器，为了方便使用，tomcat同时具有http服务器的功能
因此tomcat就是一个http服务器+servlet容器，我们叫它web容器
**http服务器**：处理http请求并响应结果
![image-20231226133449527](..\images\image-20231226133449527.png)
**servlet容器**：http服务器将请求交给servlet容器处理，servlet容器会将请求转发到具体的servlet（servlet容器用来加载和管理业务类）
servlet容器会根据web.xml文件中的映射关系，调用相应的servlet，servlet将处理的结果返回给servlet容器，并通过http服务器将响应传输给客户端。
![image-20231226133749901](..\images\image-20231226133749901.png)

# Nginx
## 静态资源服务器
将服务器上的静态文件缓存下来，通过http协议展现给客户端。
## 反向代理
客服端将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器，获取数据后再返回给客户端。对外暴露的是反向代理服务器地址，隐藏了真实服务器ip地址。反向代理”代理“的是目标服务器，这一个过程对于客户端而言是透明的。
代理服务器
## 正向代理
代理客户端 如vpn
## 负载均衡
nginx可以将接收到的客户端请求以一定的规则（负载均衡策略）均匀的分配到这个服务器集群中所有的服务器上，这个就叫做**负载均衡**。其中nginx充当的就是反向代理服务器的作用，负载均衡正是nginx作为反向代理服务器最常见的一个应用。
## 负载均衡的策略
* 轮询（Round Robin 默认）
* IP哈希
* 最小连接数