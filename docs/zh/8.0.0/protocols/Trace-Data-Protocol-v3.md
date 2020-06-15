# 跟踪数据协议v3
跟踪数据协议描述了SkyWalking代理/探针与后端之间的数据格式。 

## 总览
跟踪数据协议是由[gRPC format](https://github.com/apache/skywalking-data-collect-protocol)定义并提供的,
并且是由 [HTTP 1.1](HTTP-API-Protocol.md)实现的。

### 发送服务实例状态
1. 服务实例属性 
服务实例具有比名称更多的信息, 一旦探针想要发送它,使用 `ManagementService#reportInstanceProperties` 提供字符串键/字符串值对列表作为参数的服务。
 至少发送目标实例的`language`。

2. 服务连通
服务实例应该在后端保持存活。探针应当设置一个任务每分钟使用一次`ManagementService#keepAlive`服务。

### 发送跟踪和指标
拥有服务ID和服务实例ID后，可以发送跟踪和指标。现在我们有
1. `TraceSegmentReportService#collect` 用于 skywalking 的跟踪格式
2. `JVMMetricReportService#collect` 用于 skywalking 的JVM格式

对于跟踪格式，有一些注意事项
1. 追踪段是SkyWalking中的一个概念，它应包括单个OS进程中每个请求的所有跨度，通常是基于语言的单个线程。
2. 跨度有3个不同的类型。

* 入口跨度(EntrySpan)
EntrySpan代表服务提供商，也是服务器端的端点。作为APM系统，我们的目标是应用服务器。因此，几乎所有服务和MQ消费者都是EntrySpan。

* 本地跨度(LocalSpan)
LocalSpan代表一种普通的Java方法，与MQ服务生产者/消费者或服务（例如HTTP服务）提供者/消费者都不与远程服务无关。

* 出口跨度(ExitSpan)
ExitSpan代表服务的客户或MQ生产者，在SkyWalking的早期被称为`LeafSpan`。
例如：通过JDBC访问DB，读取Redis / Memcached被归类为ExitSpan。

3. 跨线程或进程 父信息的跨度称为依赖(Reference)。 依赖 携带 父级请求中的链路ID(trace id), 
追踪段ID(segment id), 跨度ID(span id), 服务名称(service name),服务实例名称(service instance name),
端点名称(endpoint name) 和 客户端使用的目标地址(target address used at client side)(跨线程不需要) 。 
点击查看 [SkyWalking 跨进程传播的头部协议 V3](Skywalking-Cross-Process-Propagation-Headers-Protocol-v3.md) to get more details.

4. `Span#skipAnalysis` 如果此跨度不需要后端分析，则可以为TRUE。

