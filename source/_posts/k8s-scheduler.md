---
title: k8s-scheduler
date: 2024-07-21 16:54:18
categories:
- 云原生
tags:
---

## Kube-scheduler
起承上启下的作用，首先负责接收controller manager创建新的pod，为其安排node，其次安排工作结束之后，目标node上的kubelet服务进程接管后续工作。

![image-20240721165644256](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240721165644256.png)