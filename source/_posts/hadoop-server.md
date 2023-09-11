---
title: hadoop_server
date: 2023-09-07 13:38:52
categories:
- 大数据开发
- hadoop
tags:
---

# Hadoop的部署

大规模数据的存储，计算，传输

1.上传，配置软链接

2.修改配置文件

workers

hadoop-env.sh

core-site.xml

hdfs-site.xml

3.分发到node2，node3，配置环境变量

4.创建数据目录，并修改文件权限归属hadoop账户

5.启动。



namenode：master主角色，写edits

secondarynamenode：master的辅助，通过http从namenode拉去数据（edits和fsimage），然后合并完成后提供给namenode使用

## 一键启停脚本

$HADOOP_HOME/sbin/start-dfs.sh，一键启动

原理：

在执行此脚本的机器上，启动SecondaryNameNode

读取core-site.xml内容（fs.defaultFS项），确认NameNode所在机器并启动

读取workers内容，确认DataNode所在机器，启动所有DataNode

$HADOOP_HOME/sbin/stop-dfs.sh，一键关闭



$HADOOP_HOME/SBIN/hadoop-daemon.sh 单独控制所在机器的进程的启停

```sh
hadoop-daemon.sh (start|status|stop) (namenode|secondarynamenode|datanode)
```

$HADOOP_HOME/bin/hdfs 单独控制所在机器的进程的启停

```sh
hdfs --daemon (start|status|stop) (namenode|secondarynamenode|datanode)
```



hadoop命令

老版本：hadoop fs [generic options]

新版本：hdfs dfs [generic options]

# HDFS存储原理

分布式存储：每个节点存储数据的一部分

问题：文件大小不一样，不利于统一管理

解决：设定统一的管理单位block块

## 数据写入流程

![image-20230911190443515](..\images\namenode写入流程.png)

关键信息点：

​	namenode不负责数据写入，只负责数据记录和权限审批

​	客户端直接向一台datanode写数据，这个datanode一般是离客户端最近（网络距离）的那一个

​	数据块副本的复制工作，由datanode之间自行完成（构建一个pipline，按顺序复制分发）

## 数据读取流程

![image-20230911190900682](F:\swag-lay.github.io\source\images\namenode数据读取.png)

关键信息点：

​	数据同样不通过namenode提供

​	namenode提供的block列表，会基于网络距离计算尽量提供离客户端最近的

​	这是因为1个block默认有3份，会尽量找离客户端最近的那一份让其读取