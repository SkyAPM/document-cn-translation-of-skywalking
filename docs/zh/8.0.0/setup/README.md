# 设置
安装过程基于你想要使用哪种探针。如果你对此不了解，请首先阅读[概念与设计](../concepts-and-designs/README.md)


**重要：请确保被监控的服务器上的系统时间和OAP服务器上的系统时间是相同的。**

如果你有任何问题，请检查你的问题是否在issue的[常见问题](../常见问题/帮助.md)中描述。

## 下载官方发行版
- 后端和UI、Java agent都是Apache官方发行，你可以在[Apache SkyWalking 下载页](http://skywalking.apache.org/downloads/)找到它们。

## 各语言agent

- [Java agent](service-agent/java-agent/README.md). Introduces how to install java agent to your service, without any impact in your code.

- [LUA agent](https://github.com/apache/skywalking-nginx-lua). Introduce how to install the lua agent in Nginx + LUA module or OpenResty.

- [Python Agent](https://github.com/apache/skywalking-python). Introduce how to install the Python Agent in a Python service.

- [Java agent](service-agent/java-agent/README.md)。介绍了如何将java agent安装到你的服务中，不需要修改任何代码。

下面的agent和SDK都与SkyWalking的数据格式、传输协议兼容，但是是由第三方维护。你可以到它们的项目仓库中找到对应的发行版，以及如何使用它们的说明。
- [SkyAPM .NET Core agent](https://github.com/SkyAPM/SkyAPM-dotnet)。可以通过 .NET Core agent的项目文档查看到更详细的信息。
- [SkyAPM Node.js agent](https://github.com/SkyAPM/SkyAPM-nodejs)。可以通过Node.js服务端agent的项目文档查看到更详细的信息。
- [SkyAPM PHP SDK](https://github.com/SkyAPM/SkyAPM-php-sdk)。可以通过PHP agent项目文档查看到更详细的信息。
- [SkyAPM GO2Sky](https://github.com/SkyAPM/go2sky). 参考 GO2Sky 项目文档获得更多信息.

## Service Mesh
  - Istio
    - [SkyWalking on Istio](istio/README.md)。介绍了如何使用Istio Mixer，并通过适配与SkyWalking一起工作。
  - Envoy
    - 使用 [ALS(访问日志服务)](https://www.envoyproxy.io/docs/envoy/latest/api-v2/service/accesslog/v2/als.proto) 来观测服务网格, 
    无需使用 Mixer. 按照 [文档](envoy/als_setting.md) 来开启此功能.
    
## Proxy
  - [Envoy Proxy](https://www.envoyproxy.io/)
    - [从 Envoy 向 SkyWalking 发送指标数据](envoy/metrics_service_setting.md)。介绍了如何使用[Metrics 服务](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/metrics/v2/metrics_service.proto.html) 
      将指标项从Envoy发到SkyWalking。
    - [从 Envoy 发送指标到 Skywalking](envoy/metrics_service_setting.md). 介绍了如何使用 [Metrics service](https://www.envoyproxy.io/docs/envoy/latest/api-v2/config/metrics/v2/metrics_service.proto.html) 
      向 SkyWalking 发送指标.

## 安装后端
从 [后端和 UI 设置文档](backend/backend-ui-setup.md)中可以了解到在不同的场景下如何对后端服务进行配置，以及如何开启高级特性。

## 更新日志
后端、UI和Java代理的更新日志查看 [更新日志](../../../CHANGES.md).

## 升级相关问题
[6.x 升级方法](../FAQ/v6-version-upgrade.md) 介绍了进行SkyWalking升级的推荐方法.
