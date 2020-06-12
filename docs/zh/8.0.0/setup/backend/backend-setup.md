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
1. **Level 1**, 模块名。模块定义项。
2. **Level 2**, 模块类型。 设置模块类型。
3. **Level 3**. 类型属性。

例如:

```yaml
storage:
  selector: mysql # the mysql storage will actually be activated, while the h2 storage takes no effect
  h2:
    driver: ${SW_STORAGE_H2_DRIVER:org.h2.jdbcx.JdbcDataSource}
    url: ${SW_STORAGE_H2_URL:jdbc:h2:mem:skywalking-oap-db}
    user: ${SW_STORAGE_H2_USER:sa}
    metadataQueryMaxSize: ${SW_STORAGE_H2_QUERY_MAX_SIZE:5000}
  mysql:
    properties:
      jdbcUrl: ${SW_JDBC_URL:"jdbc:mysql://localhost:3306/swtest"}
      dataSource.user: ${SW_DATA_SOURCE_USER:root}
      dataSource.password: ${SW_DATA_SOURCE_PASSWORD:root@1234}
      dataSource.cachePrepStmts: ${SW_DATA_SOURCE_CACHE_PREP_STMTS:true}
      dataSource.prepStmtCacheSize: ${SW_DATA_SOURCE_PREP_STMT_CACHE_SQL_SIZE:250}
      dataSource.prepStmtCacheSqlLimit: ${SW_DATA_SOURCE_PREP_STMT_CACHE_SQL_LIMIT:2048}
      dataSource.useServerPrepStmts: ${SW_DATA_SOURCE_USE_SERVER_PREP_STMTS:true}
    metadataQueryMaxSize: ${SW_STORAGE_MYSQL_QUERY_MAX_SIZE:5000}
  # other configurations
```

1. **`storage`** is the 模块名.
2. **`selector`** 模块类型.
3. **`default`** 模块默认属性.
4. `driver`, `url`, ... `metadataQueryMaxSize` 类型属性.


同时，模块包括必修模块和可选模块，必修模块提供后端框架，
即使模块化支持可插拔，删除这些模块是没有意义的，对于可选模块，其中一些有
一个名为“none”的提供程序实现，这意味着它只提供一个没有实际逻辑的shell，典型的例子是telemetry。
将“-”设置为“selector”意味着在运行时将排除整个模块。
我们强烈建议您不要尝试更改这些模块的api，除非你非常了解SkyWalking项目及其代码。

必须的模块列表
1. **Core**。做所有数据分析和流调度的基本和主要框架。
2. **Cluster**。管理集群中的多个后端实例，这可以提供高吞吐量的处理能力。
3. **Storage**。持久化分析结果。
4. **Query**。提供查询接口给UI。

对于**Cluster** 和**Storage** 有多个实现者(提供者), 查看 **Cluster management**
和 **Choose storage** 的[link list](#advanced-feature-document-link-list)文档。

一些**receiver** 模块也提供了。
Receiver是一个模块，负责接受后端的传入数据请求。大多数（所有）通过一些rpc协议，如GRPC和HTTPrestful提供。
Receiver有许多不同的模块名，你可以阅读[link list](#advanced-feature-document-link-list)中的**Set receivers**文档。

## 高级特性的文档链接列表
在理解了设置文件结构之后，您可以选择您感兴趣的功能文档。
我们建议您按照以下顺序阅读功能文档。

1. [Overriding settings](backend-setting-override.md)。支持重写application.yml中的设置。
2. [IP and port setting](backend-ip-port.md)。介绍如何设置和使用IP和端口。
3. [Backend init mode startup](backend-init-mode.md)。如何初始化环境并优雅地退出。
在尝试初始化新群集之前，请阅读此内容。
4. [Cluster management](backend-cluster.md)。指导您将backend服务器设置为群集模式。
5. [Deploy in kubernetes](backend-k8s.md).。指导您构建和使用Skywalking图像，并在K8S中部署。
6. [Choose storage](backend-storage.md).如我们所知，在默认的快速启动中，backend 使用h2数据库运行。但显然，它不符合产品环境。在这里，你可以找到其他的选择。
选择你喜欢的，我们也欢迎任何人贡献新的存储实施者，
7. [Set receivers](backend-receivers.md)。 你可以根据你的要求选择接收者，大多数接收者是无害的，至少我们的默认接收器是无害的。您可以设置并激活所有提供的接收器。
8. [Open fetchers](backend-fetcher.md). 你可以从目标应用程序读取指标。它们的工作原理就像接收器，但在拉取模式下，就像典型的普罗米修斯.
9. [Token authentication](backend-token-auth.md). 你可以添加令牌身份验证机制，以避免“OAP”接收不可信的数据.  
10. 在backend进行[trace sampling](trace-sampling.md) 。 此示例保持度量的准确性，但不保存某些存储中依赖于速率的traces。
11. 根据 [slow DB statement threshold](slow-db-statement.md) 配置文档可了解如何检测系统中的慢速数据库语句（包括SQL语句）。 
12. 官方 [OAL scripts](../../guides/backend-oal-scripts.md)。正如您从我们的[OAL introduction](../../concepts-and-designs/oal.md)中所知，
大多数backend分析功能都基于脚本。这是对官方手稿的描述，这有助于您了解正在处理的度量数据，也可以用于警报。
13. [Alarm](backend-alarm.md). 告警提供基于时间序列的检查机制。你可以设置针对分析OAL度量对象的告警规则。
14. [Advanced deployment options](advanced-deployment.md)。如果你需要部署的backend有很大的扩展性并支持高负载，你需要看看这个。
15. [Metrics exporter](metrics-exporter.md). 使用metrics data导出器将metric数据转发给第三方系统。
16. [Time To Live (TTL)](ttl.md). Metrics 和 trace 是时间序列数据, 因此可能会被永久存储, 你可以设置各个维度的过期时间。 
17. [Dynamic Configuration](dynamic-config.md). 从远程服务拉取动态改变OAP配置或第三方配置。
18. [Uninstrumented Gateways](uninstrumented-gateways.md). 配置SkyWalking代理插件不支持的网关/代理，在拓扑图中反映委托信息。
19. [Apdex threshold](apdex-threshold.md). 如果在OAL中激活Apdex计算，则需要为不同的服务配置阈值。
20. [Group Parameterized Endpoints](endpoint-grouping-rules.md). 为参数化端点配置分组规则，以改进指标的意义。

## backend 遥测
OAP后端集群本身的底层是一个分布式流处理系统。为了帮助运营团队，我们为OAP后端本身提供遥测。按照[document](backend-telemetry.md)使用它。

## FAQs
#### 何时以及为何我们需要设置时区

SkyWalking 提供了降取样时间序列指标的功能.
根据各个时间维度(如分钟, 小时, 天, 月)进行查询和存储时的时间格式化会涉及到时区。

比如说, 按分钟维度的指标时间将会被格式化成 YYYYMMDDHHmm 格式, 格式化过程是和时区相关的。

默认情况下, SkyWalking OAP 后端选择操作系统的默认时区, 如果你想覆盖这个行为, 请按照普通 Java 程序和操作系统设置的相关文档进行设置。

#### 如何通过第三方工具查询存储信息?
SkyWalking提供了浏览器UI、CLI和GraphQL方式来支持扩展. 但有些用户可能会想到直接从存储中查询数据。
比如在ElasticSearch的例子中，Kibana就是一个很好的工具。

在默认情况下，由于减少内存、网络和存储空间的使用, skywalk只在实体中保存id，而元数据只保存在“*_inventory”实体中。
在这种特殊情况下这些工具通常不支持嵌套查询，或者工作起来不方便。
SkyWalking提供了一个配置，将所有必要的name列添加到最终的度量实体中，以ID作为权衡。

看一下'application.yaml'中配置“core/default/activeExtraModelColumns”，并将其设置为true以打开该特性。

这个特性不会为本地的skywalk场景提供任何新特性，只用于第三方集成。.
