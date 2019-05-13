# 观测分析平台(Observability Analysis Platform, OAP)

观测分析平台(Observability Analysis Platform, OAP)是一个从 SkyWalking 6.x 开始出现的新概念.
OAP 取代了整个旧的 SkyWalking 后端. OAP 的能力如下所述.

## OAP 的能力

OAP 从多种数据源接收数据, 这些数据分为两大类, **链路追踪**和**度量指标**.

- **链路追踪**. 包括 SkyWalking 原生数据格式, Zipkin V1 和 V2 数据格式, 以及 Jaeger 数据格式.
- **度量指标**. SkyWalking 集成了服务网格平台, 如 Istio, Envoy 和 Linkerd, 并在数据面板和控制面板进行观测.
	此外, SkyWalking 原生代理还可以运行在度量模式, 这极大提升了性能.

使用集成方案的同时, SkyWalking 还提供了可视化集成来对追踪和日志进行绑定, 这是通过使用 trace id 和 span id 实现的.

所有的服务都是通过 gRPC 和 HTTP 协议实现, 使得未来集成那些尚未支持的生态系统更加容易.

## OAP 中的链路追踪

OAP 中的链路追踪有两种处理.
1. 在 SkyWalking 5.x 中传统的方式. 格式化追踪数据. OAP 通过分析数据段获得度量指标, 并将度量数据推送到聚合流.
2. 仅仅将追踪视为某种日志, 只提供存储和可视化能力.

同样的, SkyWalking 接收来自其他系统的追踪数据格式, 如 Zipkin, Jeager, OpenCensus.
这些格式也可以由以上两种方式进行处理.

## OAP 中的度量指标

OAP 中的度量指标是 6.x 版本中全新的功能. 通过连接的节点之间的度量, 构建分布式系统的观测数据, 且不需要追踪数据.

度量数据在 OAP 集群中以流的模式进行聚合. 参考[观测分析语言](oal.md), 该文介绍了如何使用简单的脚本形式进行数据聚合和分析.
