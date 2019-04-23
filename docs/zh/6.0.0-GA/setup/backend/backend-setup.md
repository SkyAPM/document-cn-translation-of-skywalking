# Backend启动
首先也是最重要的是，Skywalking backend启动行为由`config/application.yml`驱动。
了解这个设置文件将帮助您阅读此文档。

## 启动脚本
默认的启动脚本是`/bin/oapService.sh`(.bat)。
阅读[start up mode](backend-start-up-mode.md) 可了解启动backend的其它选项。


## application.yml
这个设置文件背后的核心概念是，Skywalking的Collector基于纯模块化设计。
最终用户可以根据自己的需求切换或组装收集器功能。

所有, 在`application.yml`里有三个级别。
1. **Level 1**, 模块名。说明该模块在运行模式可用。
1. **Level 2**, 提供者名。 设置模块的提供者名。
1. **Level 3**. 提供者的设置。

事例:
```yaml
core:
  default:
    restHost: 0.0.0.0
    restPort: 12800
    restContextPath: /
    gRPCHost: 0.0.0.0
    gRPCPort: 11800
```
1. **core** 就是模块。.
1. **default** 是core模块的默认实现。.
1. `restHost`, `restPort`, ... `gRPCHost` 是实现者的所有设置项。

同时，模块包括必选模块和可选模块，所需模块提供了backend的框架。即使模块化支持可插拔，删除这些模块也是没有意义的。我们强烈建议你不要试图更改这些模块的API，除非您非常了解Skywalking项目及其代码。

必须的模块列表
1. **Core**。做所有数据分析和流调度的基本和主要框架。
1. **Cluster**。管理集群中的多个后端实例，这可以提供高吞吐量的处理能力。
1. **Storage**。持久化分析结果。
1. **Query**。提供查询接口给UI。

对于**Cluster** 和**Storage** 有多个实现者(提供者), 查看 [link list](#advanced-feature-document-link-list)下的 **Cluster management**
和 **Choose storage** 的文档。

一些**receiver** 模块也提供了。
Receiver是一个模块，负责接受后端的传入数据请求。大多数（所有）通过一些rpc协议，如GRPC和HTTPrestful提供。
Receiver有许多不同的模块名，你可以阅读[link list](#advanced-feature-document-link-list)中的**Set receivers**文档。

## 高级特性的文档链接列表
在理解了设置文件结构之后，您可以选择您感兴趣的功能文档。
我们建议您按照以下顺序阅读功能文档。

1. [Overriding settings](backend-setting-override.md)。支持重写application.yml中的设置。
1. [IP and port setting](backend-ip-port.md)。介绍如何设置和使用IP和端口。
1. [Backend init mode startup](backend-init-mode.md)。如何初始化环境并优雅地退出。
在尝试初始化新群集之前，请阅读此内容。
1. [Cluster management](backend-cluster.md)。指导您将backend服务器设置为群集模式。
1. [Deploy in kubernetes](backend-k8s.md).。指导您构建和使用Skywalking图像，并在K8S中部署。
1. [Choose storage](backend-storage.md).如我们所知，在默认的快速启动中，backend 使用h2数据库运行。但显然，它不符合产品环境。在这里，你可以找到其他的选择。
选择你喜欢的，我们也欢迎任何人贡献新的存储实施者，
1. [Set receivers](backend-receivers.md)。 你可以根据你的要求选择接收者，大多数接收者是无害的，至少我们的默认接收器是无害的。您可以设置并激活所有提供的接收器。
1. 在backend进行[trace sampling](trace-sampling.md) 。 此示例保持度量的准确性，但不保存某些存储中依赖于速率的traces。
1. 根据 [slow DB statement threshold](slow-db-statement.md) 配置文档可了解如何检测系统中的慢速数据库语句（包括SQL语句）。 
1. 官方 [OAL scripts](../../guides/backend-oal-scripts.md)。正如您从我们的[OAL introduction](../../concepts-and-designs/oal.md)中所知，大多数backend分析功能都基于脚本。这是对官方手稿的描述，这有助于您了解正在处理的度量数据，也可以用于警报。
1. [Alarm](backend-alarm.md). 告警提供基于时间序列的检查机制。你可以设置针对分析OAL度量对象的告警规则。
1. [Advanced deployment options](advanced-deployment.md)。如果你需要部署的backend有很大的扩展性并支持高负载，你需要看看这个。
1. [Metric exporter](metric-exporter.md). 使用metric data导出器将metric数据转发给第三方系统。

## backend遥测
OAP后端集群本身的底层是一个分布式流处理系统。为了帮助运营团队，我们为OAP后端本身提供遥测。按照[document](backend-telemetry.md)使用它。
