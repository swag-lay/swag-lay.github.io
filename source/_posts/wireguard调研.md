# wireguard调研
说白了开源vpn协议
## 底层原理解析

**1.中继服务器原理**
中继服务器就是作为流量转发节点（网关），允许其他客户端通过此设备的nat转发访问外网。例如实验那种启用的PostUp和PostDown。
在wireguard中，并不关心流量怎么转发，由系统内核和iptables规则处理。其中客户端和服务端是对等的，不区分谁是客户端谁是服务端，双方都会监听upd端口，只看谁主动连接，谁就是客户端。
**2.流量转发**
针对不同网络拓扑的流量路由
1）端到端直接连接
双方都在一个局域网内，或者直接通过公网访问，不需要中继跳转
2）一端位于NAT后面，一端通过公网暴露
方案：通过公网暴露的一端作为服务端，另一端指定服务端的公网地址和端口，然后通过persistent-keepalive维持长连接，这里因为wireguard支持漫游，也就是说，无论双方谁的地址变动了，wireguard在看到对方从新地址通信的时候，就会记住他的新地址。所以双方要是一直保持在线，并且通信足够频繁的话（也就是配置persistent-keepalive），两边的ip都不固定也没有影响。
3）两端都位于NAT后面，需要中继服务器连接
NAT会做源端口随机化处理，直接连接可能比较困难，因此可以加一个中继服务器，双方都将中继服务器作为对端，然后维持长连接，流量就会通过中继服务器进行转发。
4）两端都位于NAT后面，通过UDP NAT打洞

发送的报文udp测试

![image-20250108132250641](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20250108132250641.png)

**3.peer**

![image-20250108134306209](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20250108134306209.png)

实际的流量发送函数，在每个endpoint结构体为

```go
endpoint struct{
    sync.Mutex
    // 这里val具有ClearSrc的方法，可以进行清除源ip或其他信息，和下面的字段实现了数据漫游
    val conn.Endpoint
    // 其中clearSrcOnTx就是用于标准源地址的ip是否发生变化，
    clearSrcOnTx   bool 
    // 启动数据漫游
	disableRoaming bool
}
```

**4.组网实现**
wireguard通过nat使得没有公网ip的设备也可以通过udp直接通信
1）endpoint定义设备：
在上面的源码中我们可以看到endpoint就是远程设备，保存了设备的地址和端口，地址也就是可能是公网地址也可能是nat后的私网地址
2）udp穿透机制：
客户端定期向发送keepalive包，确保nat机制不会超时。两个位于nat之后的设备在收到彼此的keepalive后，nat建立映射，进行点对点通信。

![image-20250108135600688](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20250108135600688.png)

## 关键概念

**peer/node/device**
连接vpn为自己注册一个vpn子网地址的主机。可以通过使用逗号分隔的CIDR指定子网范围，为其自身地址以外的ip地址选择路由
**中继服务器**
一个公网可达的节点，可以将流量中继到NAT后面的其他对等节点。
**子网**
私有IP，例如192.168.1.1/24，一般在NAT后面。
**NAT**
动态地址路由转换，子网的私有IP地址由路由器提供，通过公网无法直接访问私有子网设备，通过NAT做地址转换，路由器会跟踪发出的连接，并将响应转发到正确的内部ip
**public endpoint（公开断点）**
节点的公网ip+端口
如果peer不在同一子网内，那么节点endpoint必须使用公网ip地址
**公私钥**
单个节点的wireguard公私钥
**DNS**
域名服务器，将域名解析为vpn客户端的ip