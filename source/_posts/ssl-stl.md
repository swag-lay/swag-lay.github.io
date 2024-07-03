---
title: ssl/stl
date: 2024-06-20 15:59:28
categories:
- 计算机网络
- https—ssl
tags:
---

# SSL&STL
ssl握手使用三个密钥来建立私密连接：
- 私有密钥
- 公共密钥
- 会话密钥
当数据用私钥加密的时候，只能用公钥才能解密。大通过私钥也是唯一能够解密公钥加密信息的东西。

![img](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/9d30270e7f052b054fb074b81decb44a.png)

1. 客户端连接受ssl保护的服务端
2. 发送ssl证书副本和公钥给客户端
3. 检查证书
4. 客户端用公钥加密信息，创建对称会话密钥
5. 服务端用私钥解密会话密钥，发送加密确定以启动会话
6. 之后的所有数据都用会话密钥加密

现在往往采用tls
gin对tls启动的实现
```go
func (engine *Engine) RunTLS(addr, certFile, keyFile string) (err error) {
	debugPrint("Listening and serving HTTPS on %s\n", addr)
	defer func() { debugPrintError(err) }()

	if engine.isUnsafeTrustedProxies() {
		debugPrint("[WARNING] You trusted all proxies, this is NOT safe. We recommend you to set a value.\n" +
			"Please check https://pkg.go.dev/github.com/gin-gonic/gin#readme-don-t-trust-all-proxies for details.")
	}

	err = http.ListenAndServeTLS(addr, certFile, keyFile, engine.Handler())
	return
}
```

**证书验证阶段（使用非对称加密）**
当服务器将报文发送给客户端，第一个报文中包含证书信息，第二个报文包含公钥信息。
客户端根据自己的证书列表来确认服务器是否可信。
用CA机构的公钥对数字签名解密，用证书的签名算法对签名进行hash
比较解密的数字签名和证书内容的hash值。

## 服务器安装ssl

![img](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/a2e2486cabbefc6f3b867ccb78b7d243.png)
1. 创建密钥和发送证书签名请求
服务器需要生成私钥和公钥，然后发送签名请求
2. 将CSR（签名请求）发送到证书颁布机构
将加密的数据文件提交给CA获取证书，需要选择一个受网络浏览器（DigiCert/SSL.com）公开信任的CA
3. 在服务器上安装SSL证书
通常需要安装一个中间证书，它通过将证书连接到CA的根证书来确认证书的真实性
中间的安装步骤根据具体说明

### 为什么整个SSL的连接过程不全部采用非对称加密
非对称加密算法复杂，加密时间长，效率低，但是安全性高，所以我们只在tsl连接阶段使用，后面的数据传输同一使用对称加密。
### 假设这个证书不是由一个可信的人发的，但是这个证书本身是合法的，客户端怎么知道哪些证书可以信任，哪些不能信任？
证书的信任不仅却决于技术上是否是合法的，不仅仅基于证书自身的有效性，而是基于谁发布了这个证书。
- 受信任的CA列表：内置的根信任列表。
- 合法但不受信任：这种情况下客户端是不会信任的，浏览器会提示警告。
- 手动导入证书：在开发或者测试中开发者手动导入的信任。
- 私有CA/企业CA：仅在组织内部受信任，通过为了保障员工的设备信任这些证书，会部署到设备上。
