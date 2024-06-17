---
title: Prometheus监控k8s
date: 2024-06-1714:08:03
categories:
- 云原生
- Prometheus
tags:
---

# Prometheus监控k8s

![img](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/0d93944f9b945bc0b0bb41cb9d1b0f87.png)
左边是采集谁监控谁，一般是一些短周期的任务，比如cronjob任务，也可以是一些持久性的任务，一般主要是一些持久性的任务，比如web服务。

中间是Prometheus本身，内部是一个tsdb的数据库，但所有的被监控端暴露完指标之后，Prometheus会主动抓取这些指标，存储到tsdb数据库中，然后提供给web ui，或者api clients通过PromQL调用数据，PromQL就是查询这些数据的。

中间上方是服务发现，我们需要自动去发现新加入的节点时，或者以批量的节点加入监控中，在k8s中内置了k8s服务发现的机制，也就是他会连接k8s的api，去发现部署的应用，pod，然后暴露出去，进行监控。

![img](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/5faee1956c6b1331218588ff0e8a0690.png)

当需要监控node的资源，可以设置node_exporter，去采集机器的资源信息

当想要监控容器，k8s内置cAdvisor采集器，不需要单独部署，只要知道怎么去访问这个cAdvisor采集器就可以了

监控k8s资源对象，会部署一个kube-state-metrics这个服务，定时的从API中获取这些指标，存取到Prometheus中。

![img](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/1965c3cd5dd36e34e204ff3a557f760c.png)


## 采集数据
1. 主动拉取：
    主要通过scrape job进行，主动从目标实例中拉取实例的过程。主要通过http或者其他方式从目标实例中拉取数据，存储到本地的时间序列数据库中。
    拉取过程中需要指定目标实例的地址，端口，请求路径等信息，以及标签选择器，超时时间等参数。

  分两模块：

  discovery模块

  协程（discovery）-发现targets（chan []*targetgroup.Group）-协程（updater）-更新targets，触发信号（triggerSend）-协程（sender）拉取targets

  该模块利用各种服务发现协议发现目标采集点，通过channel通道将最新发现的目标采集点信息实时同步给scrape模型

  scrape模块

  负责使用http协议从目标采集点上抓取监控指标数据

  syncCh通道-协程（updateTsets）-更新写入到scrapeManager结构体中targetSets字段对应的map中-触发信号triggerSend-协程（reloader）加载targets-从scrapeManager中targetSets中拉取最新采集点进行加载

  

2. push gateway
    是一个临时组件，用于处理无法直接从实例拉取数据的场景，例如容器环境和短暂运行的任务。

## 怎么存储数据的
## 怎么暴露 接口的
## 存储和采集数据有哪些可配置的参数