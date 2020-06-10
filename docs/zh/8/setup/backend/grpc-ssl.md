# OAP server 支持 gPRC SSL 传输

OAP 通信我们当前使用 gRPC，一个使用协议缓冲区的消息序列化并支持多平台 RPC 框架。gRPC 的好处在于它促进了使用 SSL/TLS 进行身份验证和加密
数据交换。现在 OAP 支持为 gRPC 接收器启用 SSL 传输。

您可以按照以下步骤来启用此特性

## Creating SSL/TLS Certificates 创建 SSL/TLS 证书

似乎第一步加密生成证书和密钥文件是为通信。我认为在命令行中使用 `openssl` 会非常简单。

如果您不熟悉如何生成密钥文件使用这个 [脚本](https://github.com/apache/skywalking/blob/master/tools/TLS/tls_key_generate.sh)。

我们需要一下文件：

- `server.pem` 用于签名和验证公钥的私有 RSA 密钥.
- `server.crt` 用于分发的自签名X.509公钥.
- `ca.crt` 客户端验证服务端证书的证书中心公钥.

## 配置 OAP 服务

你可以向 `application.yml/core/default` 添加以下配置启用 gRPC SSL

```json
gRPCSslEnabled: true
gRPCSslKeyPath: /path/to/server.pem
gRPCSslCertChainPath: /path/to/server.crt
gRPCSslTrustedCAPath: /path/to/ca.crt
```

OAP 服务加载 `gRPCSslKeyPath` 和 `gRPCSslCertChainPath` 来加密通信。 `gRPCSslTrustedCAPath` 帮助 gRPC 客户端在集群模式来验证服务端证书。

如果你启用 `sharding-server` 从外部摄取数据，在 `application.yml/receiver-sharing-server/default` 文件中添加如下内容：

```json
gRPCSslEnabled: true
gRPCSslKeyPath: /path/to/server.pem
gRPCSslCertChainPath: /path/to/server.crt
```

因为  `sharding-server`  只从外部接收数据，所以根本不需要 CA 证书。

如果你对口 java agent，参考[TLS.md](../service-agent/java-agent/TLS.md)配置 java agent 启用 TLS。
