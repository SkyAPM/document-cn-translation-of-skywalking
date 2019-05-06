# 支持传输层安全性协议（TLS）
在互联网中传输数据时，传输层安全性协议（TLS）是一种很常见的安全解决方案。
在一些SkyWalking使用案例中，用户有如下使用背景：

> 目标应用在一个区域（也称为VPC），同时，SkyWalking后端在另一个区域（VPC）。
>
> 因此，安全性需求是很有必要的。

## 鉴权模式 Mode
只支持 **no mutual auth**.
- 如果你对如何产生秘钥文件不了解的话，使用这个[脚本](../../../../../tools/TLS/tls_key_generate.sh)。 
- 找到`ca.crt`，在客户端使用。
- 找到`server.crt`和`server.pem`，在服务器端使用。

## 开启并配置TLS

### agent配置
- 将`ca.crt`放到agent包的`/ca`文件夹下。注意，`/ca`文件夹并没有在分发包中创建，请自己创建它。

当agent检测到`/ca/ca.crt`后，会自动开启TLS。

请确保不会访问到区域（VPC）外的其他端口，比如防火墙、代理。
