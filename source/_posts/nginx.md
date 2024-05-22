---
title: nginx
date: 2024-03-14 13:09:09
categories:
- 网络编程
- nginx
tags:
---

# nginx是如何处理http头部的过程
请求之前，将nginx和客户端建立连接，
然后接收用户发送的http请求，
然后接收所有header，
然后决定哪些http模块处理请求

![image-20240314131424122](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240314131424122.png)
建立连接-分配内存池-http设置请求超时时间

![image-20240314131919911](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240314131919911.png)
当用户发送请到来之后，发送ack回复，事件模块拿到请求，回调ngx_http_wait_request_handler，请求变为用户态，分配内存池（这段内存是从连接内存池分配的）

![image-20240314132417225](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240314132417225.png)
处理请求只需要放入nginx内存中就行，同时需要做大量的上下文分析，所以许哟啊分配请求内存池。
在状态机解析请求行时，不断分批大内存。
分批完大内存之后，开始识别header，确定哪一个server块去处理请求，然后移除超时定时器，接下来开始核心的11个阶段http请求处理阶段

1. Read Request Headers：解析请求头。
2. Identify Configuration Block：识别由哪一个 location 进行处理，匹配 URL。
3. Apply Rate Limits：判断是否限速。例如可能这个请求并发的连接数太多超过了限制，或者 QPS 太高。
4. Perform Authentication：连接控制，验证请求。例如可能根据 Referrer 头部做一些防盗链的设置，或者验证用户的权限。
5. Generate Content：生成返回给用户的响应。为了生成这个响应，做反向代理的时候可能会和上游服务（Upstream Services）进行通信，然后这个过程中还可能会有些子请求或者重定向，那么还会走一下这个过程（Internal redirects and subrequests）。
6. Response Filters：过滤返回给用户的响应。比如压缩响应，或者对图片进行处理。
7. Log：记录日志。
1. POST_READ：在 read 完请求的头部之后，在没有对头部做任何处理之前，想要获取到一些原始的值，就应该在这个阶段进行处理。这里面会涉及到一个 realip 模块。
2. SERVER_REWRITE：和下面的 REWRITE 阶段一样，都只有一个模块叫 rewrite 模块，一般没有第三方模块会处理这个阶段。
3. FIND_CONFIG：做 location 的匹配，暂时没有模块会用到。
4. REWRITE：对 URL 做一些处理。
5. POST_WRITE：处于 REWRITE 之后，也是暂时没有模块会在这个阶段出现。
接下来是确认用户访问权限的三个模块：
6. PREACCESS：是在 ACCESS 之前要做一些工作，例如并发连接和 QPS 需要进行限制，涉及到两个模块：limt_conn 和 limit_req
7. ACCESS：核心要解决的是用户能不能访问的问题，例如 auth_basic 是用户名和密码，access 是用户访问 IP，auth_request 根据第三方服务返回是否可以去访问。
8. POST_ACCESS：是在 ACCESS 之后会做一些事情，同样暂时没有模块会用到。
最后的三个阶段处理响应和日志：
9. PRECONTENT：在处理 CONTENT 之前会做一些事情，例如会把子请求发送给第三方的服务去处理，try_files 模块也是在这个阶段中。
10. CONTENT：这个阶段涉及到的模块就非常多了，例如 index, autoindex, concat 等都是在这个阶段生效的。
11. LOG：记录日志 access_log 模块。

![image-20240314140800523](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240314140800523.png)