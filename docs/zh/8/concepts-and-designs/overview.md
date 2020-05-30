# 概念与设计总览

SkyWalking: 一个开源的可观测平台, 用于从服务和云原生基础设施收集, 分析, 聚合及可视化数据。SkyWalking 提供了一种简便的方式来清晰地观测分布式系统, 甚至横跨多个云平台。SkyWalking 更是一个现代化的应用程序性能监控(Application Performance Monitoring)系统, 尤其专为云原生、基于容器的分布式系统设计.

## 为什么使用 SkyWalking

在许多不同的场景下, SkyWalking 为观察和监控分布式系统提供了解决方案。首先是像传统的方式那样, SkyWalking 为服务提供了自动打点的代理, 如 Java, C# , Node.js , Go , PHP 以及 Nginx LUA（包括 Python 和 C++ 调用的 SDK 捐献）。

对于多数语言，持续部署环境，云原生基础设施正变得更加强大，但也更加复杂。

Skywalking 的服务网格接收器可以让 Skywalking 接收来自服务网格框架（例如 Istio , Linkerd）的遥测数据，以帮助用户理解整个分布式系统。

总之, SkyWalking 为 **服务(service)**, **服务实例(service instance)**, 以及 **端点(endpoint)** 提供了可观测能力。服务(Service), 实例(Instance) 以及 端点(Endpoint) 等概念在如今随处可见, 所以让我们先了解一下他们在 SkyWalking 中都表示什么意思：

- **服务(Service)**. 表示对请求提供相同行为的一组工作负载. 在使用打点代理或 SDK 的时候,你可以定义服务的名字. SkyWalking 还可以使用在 Istio 等平台中定义的名称。
- **服务实例(Service Instance)**. 上述的一组工作负载中的每一个工作负载称为一个实例. 就像 Kubernetes 中的 `pods` 一样,服务实例未必就是操作系统上的一个进程. 但当你在使用打点代理的时候, 一个服务实例实际就是操作系统上的一个真实进程.
- **端点(Endpoint)**. 对于特定服务所接收的请求路径, 如 HTTP 的 URI 路径和 gRPC 服务的类名 + 方法签名。

使用 SkyWalking 时, 用户可以看到服务与端点之间的拓扑结构, 每个服务/服务实例/端点的性能指标, 还可以设置报警规则。

除此之外, 你还可以通过一下方式集成

1. 其他分布式追踪使用 Skywalking 原生代理和Zipkin , Jaeger 和 OpenCensus 的 SDK；
2. 其他度量指标系统，例如 Prometheus , Sleuth(Micrometer。

## 架构

SkyWalking 逻辑上分为四部分: 探针, 平台后端, 存储和用户界面.

![img 架构图](http://skywalking.apache.org/assets/frame-v8.jpg?u=20200423)

- **探针** 基于不同的来源可能是不一样的, 但作用都是收集数据, 将数据格式化为 SkyWalking 适用的格式.
- **平台后端**, 支持数据聚合, 数据分析以及驱动数据流从探针到用户界面的流程。分析包括 Skywalking 原生追踪和性能指标以及第三方来源，包括 Istio 及 Envoy  telemetry , Zipkin 追踪格式化等。 你甚至可以使用  [Observability Analysis Language 对原生度量指标](oal.md) 和 [用于扩展度量的计量系统](meter.md) 自定义聚合分析。
- **存储** 通过开放的插件话的接口存放 SkyWalking 数据. 你可以选择一个既有的存储系统, 如 ElasticSearch, H2 或 MySQL 集群(Sharding-Sphere 管理),也可以选择自己实现一个存储系统. 当然, 我们非常欢迎你贡献新的存储系统实现。
- **UI** 一个基于接口高度定制化的Web系统，用户可以可视化查看和管理 SkyWalking 数据。

## What is next

- 学习了解 SkyWalking 的 [项目目标](project-goals.md)
- 问题, [为什么 SkyWalking 架构不涉及 MQ?](../FAQ/why_mq_not_involved.md)
