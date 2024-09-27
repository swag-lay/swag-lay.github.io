---
title: k8s-apiserver
date: 2024-09-27 15:38:53
categories:
- 云原生
- k8s
tags:
---

# k8s-apiserver的设计
core groups会调用/api下面的接口，name groups会调用/apis/$NAME/$VERSION
还有一些暴露系统状态的 /metrics /healthz

当一个post请求到达apiserver之后，kube-apisever会手下能执行在http filter chain中注册的过滤器链，包括认证，鉴权。然后通过route进入handler中，进行etcd的交互。
这里因为不同的资源可能对应多个version，那我需要知道传过来的的资源对应的version，所以会先解码body，转成internal version，然后变成etcd的storege version。

kube-apiserver 中包含三个 server，分别为 KubeAPIServer、APIExtensionsServer 以及 AggregatorServer，三个 server 是通过委托模式连接在一起的，初始化过程都是类似的，首先为每个 server 创建对应的 config，然后初始化 http server，http server 的初始化过程为首先初始化 GoRestfulContainer，然后安装 server 所包含的 API，安装 API 时首先为每个 API Resource 创建对应的后端存储 RESTStorage，再为每个 API Resource 支持的 verbs 添加对应的 handler，并将 handler 注册到 route 中，最后将 route 注册到 webservice 中。