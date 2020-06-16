# 动态配置

SkyWalking 配置大部分通过 `application.yml` 和系统环境变量进行设置.
但其中有一些支持从上游管理系统进行获取.

到目前为止, SkyWalking 支持以下动态配置.

| Config Key | Value 描述 | Value 格式示例 |
|:----:|:----:|:----:|
|receiver-trace.default.slowDBAccessThreshold| 数据库慢语句阀值, 覆盖 `applciation.yml` 中的 `receiver-trace/default/slowDBAccessThreshold`. | default:200,mongodb:50|
|receiver-trace.default.uninstrumentedGateways| 需要网关，则重写 gateways.yml 文件. | 与 gateways.yml 文件一致|
|alarm.default.alarm-settings| 告警设置, 需要重写 alarm-settings.yml 文件. | 与 alarm-settings.yml 文件一致|
|core.default.apdexThreshold| apdex 阈值设置, 需要重写 service-apdex-threshold.yml 文件.|与 service-apdex-threshold.yml 文件一致|
|core.default.endpoint-name-grouping| 端点名称分组设置, 需要重写 endpoint_name_grouping.yml 文件.| 与 endpoint_name_grouping.yml 文件一致|


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
    #period : 60 # 单位秒，同步周期。默认每60秒取回一次.
    #clusterName: "default" # 当前群集的名称，如果想要上游系统的名称，请设置.  
```

## 动态配置 Apollo 实现

动态配置服务也支持 [Apollo](https://github.com/ctripcorp/apollo/), 只需按如下配置:

```yaml
configuration:
  apollo:
    apolloMeta: <your apollo meta address>
    apolloCluster: default
    # apolloEnv: # 默认为null
    appId: skywalking
    period: 5
```

## 动态配置 Nacos 实现

动态配置服务也支持 [Nacos](https://github.com/alibaba/nacos), 只需按如下配置:

```yaml
configuration:
  nacos:
    # Nacos 服务器主机
    serverAddr: 127.0.0.1
    # Nacos 服务器端口
    port: 8848
    # Nacos 配置分组
    group: 'skywalking'
    # 单位秒，同步周期。默认每60秒提取一次。
    period : 60
    # 当前群集的名称，如果想要上游系统的名称，请设置。
    clusterName: "default"
```

## 实现动态配置 Zookeeper

动态配置也支持 [Zookeeper](https://github.com/apache/zookeeper) 作为配置中心, 如需启用, 只需按如下配置:

```yaml
configuration:
  zookeeper:
    period : 60 # 单位秒，同步周期。默认每60秒提取一次。
    nameSpace: /default
    hostPort: localhost:2181
    #重试策略
    baseSleepTimeMs: 1000 # 重试等待的初始时间
    maxRetries: 3 # 重试的最大次数
```

## 第三方配置中心
欢迎贡献此模块的实现来支持一些流行的配置中心, 如 etcd, Consul. 提交 issue 进行讨论。


