# wireguard组网实验

## 实验可行性
机器：一台阿里云（带公网ip）+windows本地wsl（无公网ip）
阿里云：ubuntu20.04 内核5.4.0-73-generic
wsl：ubuntu22.04 内核5.15.167.4-microsoft-standard-WSL2
组网设置：

| 机器   | 公网            | 组网                 |
| ------ | --------------- | -------------------- |
| wsl    | 无              | 组网ip：10.10.0.1/24 |
| 阿里云 | 116.198.246.XXX | 组网ip：10.10.0.0/24 |

1.两台机器均需要下载wireguard，官网要求内核保证在5.6以上，否则需要自行编译升级。实验证明5以上可行。

2.开启ipv4流量转发
```bash
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
echo "net.ipv4.conf.all.proxy_arp = 1" >> /etc/sysctl.conf
sysctl -p

echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p
```

3.关闭防火墙或者开放端口

4.安装
ubuntu下：

```bash
sudo apt install wireguard
```

wireguard采用interface（本端）和peer（对端），通常不区分客户端和服务端，一台机器既可以生成客户端密钥也可以生成服务端密钥。
此处我们将带公网ip的机器定义为本端（服务端）

### intreface
生成服务端的公私钥
```bash
sudo mkdir -p /etc/wireguard && sudo chmod 0777 /etc/wireguard && cd /etc/wireguard
umask 077 
wg genkey | tee private.key | wg pubkey > public.key
```
通常我们需要在wireguard文件夹下创建对应wireguard网卡的配置文件，然后启动网卡，对于一个组网需要一个配置文件即可
此处我们使用wg0.conf配置
```bash
[Interface]
Address = 10.10.0.0/24 #组网ip
ListenPort = 7777 #wireguard监听端口
PrivateKey = XXXXX # 本机wireguard生成私钥
# 连接启动执行命令：
# 1：iptables -A FORWARD -i wg0 -j ACCEPT 允许wg0的入站转发
# 2：iptables -A FORWARD -o wg0 -j ACCEPT 允许wg0的出战转发
# 3：iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE 对出站数据包启动nat，利用eth0访问外网
PostUp   = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
# 断开执行命令 取消连接的所有iptables规则
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
PublicKey = XXXXX # 对端公钥
AllowedIPs = 10.10.0.1/24 # 对端组网ip
```

### peer
同样生成公私钥
配置client.conf到/etc/wireguard
```bash
[Interface]
PrivateKey = XXXXX # 本端私钥
Address = 10.10.0.1/24 # 组网ip

[Peer]
PublicKey = XXXXX # 对端公钥
Endpoint = 116.198.246.XXX:7777 # 对端公网ip+监听端口
AllowedIPs  = 10.10.0.0/24 # 对端组网ip
PersistentKeepalive = 25 
```
####  启动
```bash
wg-quick up wg0 #最后的wg0就是conf的文件名，例如wsl端文件为client.conf，就启动client网卡
```
#### 停止
```bash
wg-qucik down wg0
```

### 测试
启动两端wireguard网卡

interface

![image-20250108104138763](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20250108104138763.png)

![image-20250108104324979](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20250108104324979.png)

peer

![image-20250108104347535](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20250108104347535.png)

![image-20250108104420816](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/image-20250108104420816.png)

直接互相ping组网ip即可测试

或者nc监听端口发送信息

