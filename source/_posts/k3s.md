---
title: k3s
date: 2024-11-25 10:39:16
categories:
- 云原生
- k3s
tags:
---

# k3s单机
所有kubernetes control plane组件的操作都封装在单个二进制文件和进程中，所以k3s支持自动化和管理复杂的集群操作。
k3s减少了外部依赖，仅需要现代内核和cgroup挂载。

## 个人注册表
可以将容器配置为连接到个人注册表，在启动时，k3s会检测是否存储/etc/rancher/k3s/registries.yaml，如果有，则在生成容器配置时使用此文件中包含的注册表配置。

## 嵌入注册镜像
内嵌Spegel，一种无状态的分布式OCI注册源，允许集群中的节点之间点对点共享容器映射。默认情况禁用了。
启动：以-embedded-registry标志或嵌入式registry:true在配置文件中启动服务器节点。

## 部署单机版流程
配置要求：1cpus/512mb，推荐2cpus/1gb

docker安装 

单机安装命令：
使用官网安装脚本，默认使用containerd运行容器，需要在后面加上-s --docker参数选择使用docker
```shell
curl -sfL https://rancher-mirror.rancher.cn/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn sh -s - --docker
```
安装完成之后：
- 默认k3s服务将被配置为在节点重启后/进程崩溃，杀死时自动重启
- 将安装其他实用程序，包括kubectl，crictl，ctr等
- kubeconfig文件将被写入到/etc/rancher/k3s/k3s.yaml中，由k3s安装的kubectl自动使用该文件

除非希望向集群添加容量或冗余，否则没有必要添加额外的server或agent节点
