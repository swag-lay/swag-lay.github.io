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

## 服务器安装ssl

![img](https://web-mhe.oss-cn-beijing.aliyuncs.com/hexo/a2e2486cabbefc6f3b867ccb78b7d243.png)
1. 创建密钥和发送证书签名请求
服务器需要生成私钥和公钥，然后发送签名请求
2. 将CSR（签名请求）发送到证书颁布机构
将加密的数据文件提交给CA获取证书，需要选择一个受网络浏览器（DigiCert/SSL.com）公开信任的CA
3. 在服务器上安装SSL证书
通常需要安装一个中间证书，它通过将证书连接到CA的根证书来确认证书的真实性
中间的安装步骤根据具体说明



