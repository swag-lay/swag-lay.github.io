---
title: yarn
date: 2023-09-12 09:47:14
categories:
- 大数据开发
- hadoop
tags:
---

# yarn

管控整个集群的资源进行调度

## yarn架构

主从结构：

主角色（master）：resourcemanager

从角色（slave）：nodemanager

## yarn辅助角色

搭配两个辅助角色使得yarn更加稳定

代理服务器（proxyserver）：web application proxy web应用程序代理，使用代理的原因是为了减少通过yarn进行基于网络的攻击的可能性

历史服务器（jobhistoryserver）：应用程序历史信息记录服务



## yarn容器

![image-20230912100208820](..\images\image-20230912100208820.png)

![image-20230912100236664](..\images\image-20230912100236664.png)

## 常见命令

```sh
start-yarn.sh
stop-yarn.sh

yarn --daemon start|stop resourcemanager|nodemanager|proxyserver
mapred --damon start|stop historyserver
```

## 提交mapreduce示例程序到yarn运行

yarn作为资源调度管理框架，其自身提供资源供许多程序运行，常见的有：

mapreduce程序

spark程序

flink程序

### mapreduce示例程序

内置示例代码在：

$HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-xxx.jar

通过hadoop jar运行

```sh
hadoop jar 程序文件 java类名 [程序参数] ...
```

