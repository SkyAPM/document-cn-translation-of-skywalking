# 支持传输层安全性协议（TLS）

在互联网中传输数据时，传输层安全性协议（TLS）是一种很常见的安全解决方案。

在一些 SkyWalking 使用案例中，用户有如下使用背景：

Transport Layer Security (TLS) is a very common security way when transport data through Internet.
In some use cases, end users report the background:

> 目标(正在被监控)应用在一个区域（也称为VPC），同时，SkyWalking 后端在另一个区域（VPC）。
> 因此，安全性需求是很有必要的。

## 认证模式

只支持 **no mutual auth**.

- 如果你对如何产生秘钥文件不了解的话，使用这个[脚本](../../../../../tools/TLS/tls_key_generate.sh)。
- 找到 `ca.crt` ，在客户端使用。
- 找到 `server.crt` ,`server.pem` 和 `ca.crt`，在服务器端使用。 阅读[gRPC SSL](../../backend/grpc-ssl.md) 了解更多。

## 开启并配置 TLS

### Agent 配置

- 将 `ca.crt` 放到 agent 包的 `/ca` 文件夹下. 注意, `/ca` 文件夹并没有在分发包中创建，请自行创建它。

当 agent 检测到 `/ca/ca.crt` 后，会自动开启 TLS。

请确保不会访问到区域（VPC）外的其他端口，比如防火墙、代理。
