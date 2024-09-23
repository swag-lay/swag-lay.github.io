---
title: grpc-服务发现
date: 2023-09-22 11:10:05
categories:
- 中间件源码
- grpc
tags:
---

# 服务发现
服务发现一般有客户端路由和代理层路由两种

## 客户端路由
由调用方负责直接获取被调用方的地址，并使用load balance算法发送请求。
所有的server启动都会向注册中心注册自身的服务地址，同时可以通过心跳检测保障服务是否可用。
优点：减少代理层带来的网络通信，开发者更灵活控制路由。
缺点：客户端复杂了。
grpc官方采用的服务发现流程采用客户端路由的方式

![image-20240922111935430](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20240922111935430.png)

## 代理层路由
不由调用方去获取，通过代理去获取。

优点：简化客户端，其中管理，可以进行统一的监控和日志管理。需要统一的通信机制。


## 源码解析
在源码中怎么获取到服务地址的。
resolver类其实就是通过名字去取服务的一个接口。所有的放入一个map种通过shceme进行映射builder。
其中主要有builder这个接口
```golang
type Builder interface {
	//所有的子类都需要实现这个 最后返回一个有地址的resolver，也就是server的地址列表
	Build(target Target, cc ClientConn, opts BuildOptions) (Resolver, error)
	// Scheme returns the scheme supported by this resolver.  Scheme is defined
	// at https://github.com/grpc/grpc/blob/master/doc/naming.md.  The returned
	// string should not contain uppercase characters, as they will not match
	// the parsed target's scheme as defined in RFC 3986.
	Scheme() string
}
```

我们看一下通过的resolver实现的Builder接口
```golang
func (r *Resolver) Build(target resolver.Target, cc resolver.ClientConn, opts resolver.BuildOptions) (resolver.Resolver, error) {
	r.mu.Lock()
	defer r.mu.Unlock()
	// Call BuildCallback after locking to avoid a race when UpdateState
	// or ReportError is called before Build returns.
	//这里target里面就存放的url，服务地址/服务名，cc是客户端的网络连接
	//这里可以自己设计回调函数，例如日志，资源初始化等。
	r.BuildCallback(target, cc, opts)
	r.CC = cc
	if r.lastSeenState != nil {
		err := r.CC.UpdateState(*r.lastSeenState)
		go r.UpdateStateCallback(err)
	}
	return r, nil
}
```

我们可以自己设计resovler和builder完成一些操作
比如我们将服务放到公司的注册中心里面，然后设置build接口，因为这里我们不依赖与DNS，而是基于服务注册中心。

```golang
func (b *OcrResolverBuilder) Build(target resolver.Target, cc resolver.ClientConn, opts resolver.BuildOptions) (resolver.Resolver, error) {
	// 执行回调
	if b.callback != nil {
		b.callback(target, cc, opts)
	}

	// 假设这里从数据库中获取服务地址
	// 在这里因为grpc的默认解析方式只能解析一个ip地址，所以采用自定义resovler可以解决这个问题
	cfg := ocrbase.GetOcrConfig()
	addresses := cfg.PPConfig.ApiUrls

	// 初始化自定义解析器
	resolver := &OcrResolver{
		target: target,
		cc:     cc,
		addrs:  addresses,
	}

	// 向 gRPC 客户端更新地址
	resolver.updateClientConnState()

	return resolver, nil
}
```

在这里更新的时候 最后就到了UpdateState接口
```golang
func (ccr *ccResolverWrapper) UpdateState(s resolver.State) error {
	ccr.cc.mu.Lock()
	ccr.mu.Lock()
	if ccr.closed {
		ccr.mu.Unlock()
		ccr.cc.mu.Unlock()
		return nil
	}
	if s.Endpoints == nil {
		s.Endpoints = make([]resolver.Endpoint, 0, len(s.Addresses))
		for _, a := range s.Addresses {
			ep := resolver.Endpoint{Addresses: []resolver.Address{a}, Attributes: a.BalancerAttributes}
			ep.Addresses[0].BalancerAttributes = nil
			s.Endpoints = append(s.Endpoints, ep)
		}
	}
	//第一个是用于记录解析器状态的变化。检测resovler的state是否变化，生成日志或者跟踪事件。这里主要是帮助调试
	ccr.addChannelzTraceEvent(s)
	ccr.curState = s
	ccr.mu.Unlock()
	//这里是才是实际系统的状态进行更新
	return ccr.cc.updateResolverStateAndUnlock(s, nil)
}
```

## 开发中的两个问题

这里更新ip会不会出现滞后问题，导致ip不对

一般采用watch机制，通常采用etcd或者zookeeper都有watch机制，当发现配置被修改的时候，会发送给订阅的客户

比如etcd，他的watch机制允许客户端注册一个监听器，当被监听的键值对发生变化时，主动通知客户端。
etcd的事件通知系统是实现watch机制的关键，该系统采用发布-订阅模式。也就是leader节点会发布一个事件，所有订阅了该事件的watch请求（watch请求由客户端发起，然后加入了watch队列）。
对于服务端来说有核心的几点：
- watch 功能通过 etcd 客户端与 etcd 服务端特定节点间建立的 grpc 长连接实现
- etcd 服务端模块分为上层的 serverWatchStream 和底层的 watchableStore 模块两部分
- serverWatchStream 承上启下，一方面和 etcd 客户端通过长连接通信，一方面和 watchableStore 通过 watchStream 通信交互
- 在 serverWatchStream 中会异步启动读协程 recvLoop 和写协程 sendLoop，分别负责读写客户端的请求/响应数据
- watchableStore 中会完全基于内存存储 watcher 的存储，根据是否需要回溯历史变更记录，watcher 会被分为 syncd、uysynced 两部分. 此外还有一个 victims 部分用于暂存因容量不足而发送阻塞的回调事件
- 在每个 etcd 服务端节点将数据写入状态机的位置存在一个公共的切面，在这个切面上 etcd 执行了 notify 操作，基于变更的数据记录和当前节点中存在的 watchers 进行 join，生成一个批次的 watch 回调事件，然后向上传递，完成通知回调

**对于我们的注册中心其实是数据库，那这里我们相当于需要对mysql的数据变化做监听**
一般包括这几种方法
- 轮询方式
- 使用触发器
- redis的pub和sub机制
- binlog方式
- canal工具

binlog方式：
1. 开启binlog：在MySQL配置文件中，将`log_bin`参数设置为ON。

2. 使用MySQL的`mysqlbinlog`命令行工具将binlog中的内容读取出来，并进行解析。

3. 解析binlog中的内容，判断数据变更的类型，如果是插入、更新或删除，则触发对应的回调函数，通知后端服务进行相应的处理。

