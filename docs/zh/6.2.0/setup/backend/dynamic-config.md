# 动态配置

SkyWalking 配置大部分通过 `application.yml` 和系统环境变量进行设置.
但其中有一些支持从上游管理系统进行获取.

到目前为止, SkyWalking 支持以下动态配置.

| Config Key | Value 描述 | Value 格式示例 |
|:----:|:----:|:----:|
|receiver-trace.default.slowDBAccessThreshold| 数据库慢语句阀值, 覆盖 `applciation.yml` 中的 `receiver-trace/default/slowDBAccessThreshold`. | default:200,mongodb:50|


此功能依赖上游服务, 因此默认是关闭的.

```yaml
configuration:
  none:
```

## 动态配置服务 (Dynamic Configuration Service, DCS)
[动态配置服务](../../../../oap-server/server-configuration/grpc-configuration-sync/src/main/proto/configuration-service.proto) 
是一个 gRPC 服务, 需要上游系统进行实现.
在你按照如下方式配置开启此功能之后, SkyWalking OAP 就可以从该服务的实现获取配置信息

```yaml
configuration:
  grpc:
    # 上游系统主机名
    host: 127.0.0.1
    # 上游系统端口
    port: 9555
    #period : 60 # Unit seconds, sync period. Default fetch every 60 seconds.
    #clusterName: "default" # the name of current cluster, set the name if you want to upstream system known.  
```

## 动态配置 Apollo 实现

动态配置服务也支持 [Apollo](https://github.com/ctripcorp/apollo/), 只需按如下配置:

```yaml
configuration:
  apollo:
    apolloMeta: <your apollo meta address>
    apolloCluster: default
    # apolloEnv: # defaults to null
    appId: skywalking
    period: 5
```

## 动态配置 Nacos 实现

动态配置服务也支持 [Nacos](https://github.com/alibaba/nacos), 只需按如下配置:

```yaml
configuration:
  nacos:
    # Nacos Server Host
    serverAddr: 127.0.0.1
    # Nacos Server Port
    port: 8848
    # Nacos Configuration Group
    group: 'skywalking'
    # Unit seconds, sync period. Default fetch every 60 seconds.
    period : 60
    # the name of current cluster, set the name if you want to upstream system known.
    clusterName: "default"
```

## Dynamic Configuration Zookeeper Implementation

动态配置也支持 [Zookeeper](https://github.com/apache/zookeeper) 作为配置中心, 如需启用, 只需按如下配置:

```yaml
configuration:
  zookeeper:
    period : 60 # Unit seconds, sync period. Default fetch every 60 seconds.
    nameSpace: /default
    hostPort: localhost:2181
    #Retry Policy
    baseSleepTimeMs: 1000 # initial amount of time to wait between retries
    maxRetries: 3 # max number of times to retry
```

## 第三方配置中心
欢迎贡献此模块的实现来支持一些流行的配置中心, 如 etcd, Consul. 提交 issue 进行讨论。


