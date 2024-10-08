---
title: golang-net
date: 2024-10-08 10:30:28
categories:
- 计算机网络
tags:
---

# golang中的网络编程

当大量的连接同时到达服务器，怎么进行网络编程
方案：
1.异步io编程 非阻塞 io多路复用
2.多线程/多进程/多协程

优化：
tcp参数，keep alive，连接数目，
传输的文件协议 压缩
减少心跳时间

## golang内置包net
当我们监听一个socket之后，传输数据采用conn.Write(buffer []byte)的时候，底层代码
```golang
func (fd *FD) Write(p []byte) (int, error) {
    ...
    var nn int
    for {
        max := len(p)
        if fd.IsStream && max-nn > maxRW {
            max = nn + maxRW
        }
        n, err := syscall.Write(fd.Sysfd, p[nn:max])
        if n > 0 {
            nn += n
        }
        // 如果全部写入kernel了，返回成功
        if nn == len(p) {
            return nn, err
        }
        // 重点：还没写完，挂起goroutine等待通知
        if err == syscall.EAGAIN && fd.pd.pollable() {
            if err = fd.pd.waitWrite(fd.isFile); err == nil {
                continue
            }
        }
        if err != nil {
            return nn, err
        }
        if n == 0 {
            return nn, io.ErrUnexpectedEOF
        }
    }
}
//每次创建完新连接之后，该连接都会被注册到事件列表，runtime包调度协程底层依然采用epoll/select等底层接口进行io多路复用。也就是说协程并不会阻塞系统线程，对于用户来说代码依然是同步的。
```

golang官网推崇使用goroutinue-per-connection，对于数据一个负责写，一个负责读。但是这样创建的goroutinue一旦太多依然会影响性能。

### 造轮子解决-gnet
直接使用epool和kqueue系统调用，不用net包，类似netty

同样的主从多reactors模型

注意使用gnet的原则：永远不能把业务逻辑写入EventHandler.React中，调度是调度，业务是业务，才不会阻塞event-loop线程

通过我们可以基于gnet加入协程池进一步改善，把阻塞的业务代码放入协程池中执行
https://zhuanlan.zhihu.com/p/82758065