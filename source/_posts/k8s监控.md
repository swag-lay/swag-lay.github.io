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
存储时间序列数据的是他自带的时间序列数据库，称为prometheus存储引擎

特点：
1. 单机存储
2. 持久化存储，机器重启之后仍让可用
3. 可压缩：压缩时间序列数据，节约磁盘空间
4. 支持快照

存储引擎底层采用一种称为WAL（write-ahead-logging）的机制确保数据的可靠性和一致性。该机制基于日志文件，当收集到新的指标数据时，将数据写入wal文件中，然后异步的将数据写入本地磁盘的时序数据库中。
存储底层用的存储格式为TSDB，基于时间的块存储方式，即将每个时间序列按照时间戳划分成一系列固定大小的块，并对每个块进行压缩存储。这种方式可以大幅减少存储空间，提高查询效率。

首先同样采用了内存缓存，基于哈希表的数据结构

存储数据的目录结构
```javacript
data/
  |- alerts/
  |- chunks/
  |- head/
  |- index/
  |- rules/
  |- snapshots/
  |- wal/
  
alerts/: 存储警报规则的状态信息
chunks/: 存储时序数据的块文件，每个块文件存储一段时间内的时序数据。块文件名由一组标签（label）组成，用于标识这段时间内的时序数据，例如：01D3EVB6S8SJP91GZM0RZP4YJF。
head/: 存储最近的时序数据，用于快速查询。与块文件不同，head目录存储的是最近的时序数据，而不是按时间切分的块文件。
index/: 存储指向块文件的索引信息，以便能够快速查找特定标签（label）的时序数据。
rules/: 存储告警规则的定义信息
snapshots/: 存储快照数据，用于在Prometheus重启时快速恢复数据。
wal/: 存储写入预写日志（write-ahead log）的数据。

```


## 怎么暴露 接口的
默认对每一个被监控的目标（target）使用/metrics路径获取指标。我们在配置文件中设置了目标地址，然后Prometheus就会从这个endpoint中收集指标数据

## 存储和采集数据有哪些可配置的参数
存储数据：
```javascript
--storage.tsdb.path: 存储数据的目录，默认为data/，如果要挂外部存储，可以指定该目录
--storage.tsdb.retention.time: 数据过期清理时间，默认保存15天
--storage.tsdb.retention.size: 实验性质，声明数据块的最大值，不包括wal文件，如512MB
--storage.tsdb.retention: 已被废弃，改为使用storage.tsdb.retention.time
```