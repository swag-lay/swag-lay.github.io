---
title: docker
date: 2023-07-16 11:57:41
categories:
- 云原生
- docker
tags:
---

docker是开源的应用容器引擎
开发者将他们的应用以依赖都打包到一个轻量级，可移植的容器中，然后实现虚拟化

## 什么是docker
docker是一个c/s架构，守护进程运行在主机上，然后通过socket连接从客户端访问docker守护进程。
守护进程从客户端接收命令，按照命令管理运行在主机上的容器
**一个docker容器，是一个运行时环境，可以简单理解为进程运行的集装箱**

## 应用场景
- web应用的自动化打包和发布
- 自动化测试和持续集成，发布
- 在服务器中部署和调整数据库或其他的后台应用

## docker和vm的区别
1. docker有着更少的抽象层

2. docker利用宿主机的内核，vm需要的是guest os
   ![image-20240716144051219](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240716144051219.png)
两者的区别：
- vm直接在宿主机，宿主机操作系统的基础上创建虚拟层，虚拟化的操作系统，虚拟化的仓库，然后再安装应用。
- docker容器再宿主机，宿主机操作系统上先创建docker引擎，在引擎的基础上安装应用。

3. docker启动速度块，磁盘使用一般在mb，性能接近原生，单机支持上千个容器

## docker的技术基础
### namespace
命名空间是容器隔离的基础，保持A容器看不到B容器
通常一个容器包含6个命名空间：User，Mnt，Network，UTS，IPC，Pid
```bash
docker inspect -f "{{.State.Pid}}" 容器id
ls -l /proc/xxxx/ns
```

![image-20240716150743606](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240716150743606.png)
cgroups（控制组）容器资源统计和隔离
主要用到的控制组包括：cpu，blkio，device，freezer，memory
因此docker的本质就是限制了的Namespaces，cgroup，具有逻辑上独立文件系统和网络的一个进程

**pid隔离**
因为存在命名空间 彼此隔离，所以docker中不同容器是否可以运行相同pid呢？
答案是可以
此处用到了pid namespace，其实是linux创建新进程时的一个可选参数，在linux系统中创建进程的系统调用是clone方法
```cpp
int clone(int (*fn) (void *)，void *child stack,
          int flags， void *arg， . . .
         /* pid_ t *ptid, void *newtls, pid_ t *ctid */ ) ;
```
通过调用这个方法，改进程会获取一个独立的进程空间，他看不到宿主机上的其他进程，这也就是在容器内执行ps命令的结果。

1.进程隔离
	![image-20240716162010825](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240716162010825.png)
在docker容器内部打印进程与宿主机的进程隔离

![image-20240716162250430](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240716162250430.png)

**网络隔离**
在容器之间虽然可以通过命名空间创建隔离的网络环境，但是docker中的服务也需要和外界相连。
docker有四种网络模式：host，container，none，bridge
默认bridge模式，此模式下除了分配隔离的网络命名空间，还会给容器设置ip，所有容器的网桥和主机上的虚拟网桥docker0相连。


### unionfs联合文件系统
**镜像**
image是docker部署的基本单位，一个image包含了我们的程序文件，以及这个程序依赖的资源的环境。

unionfs是一种为linux操作系统设计的用于把多个文件系统联合到同一个挂载点的文件系统服务。
AUFS是unionfs的升级版，将不同文件夹中的层联合到了同一个文件夹中，这些文件夹在AUFS中称作分支，整个联合的过程被称为联合挂载。
在linux中我们采用mount联合文件，比如我们将服务的源代码和一个存放代码修改记录的目录联合起来，前者设置只读权限，后者设置读写权限，那么一切对源代码目录的修改都只会影响那个存放修改的目录，不会污染原始的代码

**dockerde的镜像分层机制**

![image-20240716155805197](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240716155805197.png)

docker image是一个层级结构，最底层的layer为BaseImage（一般是操心系统的iso镜像），然后顺序执行每一条指令，生成的layer按照入栈的顺序逐渐累加，最终形成一个Image
layer和image的关系与AUFS中的联合目录和挂载点的关系比较相似

![image-20240716155718721](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240716155718721.png)

### CGroups物理资源分组
命名空间不能为我们提供物理资源上的隔离，比如CPU或内存。
比如某个容器运行cpu密集型任务，影响其他容器的执行性能和效率。因此Control Groups（简称CGroups）就是能够隔离主机器上的物理资源