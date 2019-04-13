# 追踪数据协议 v2 版本

追踪数据协议描述了数据在 SkyWalking 代理/sniffer 和后端之间进行传输的格式。

## 摘要

追踪数据协议在 [gRPC 格式](https://github.com/apache/incubator-skywalking-data-collect-protocol) 中定义和提供。

对于每个代理/SDK，在报告任何追踪/指标数据之前，它都需要注册服务 ID 和服务实例 ID。

### 第 1 步. 注册

[注册服务](https://github.com/apache/incubator-skywalking-data-collect-protocol/tree/master/register/Register.proto)掌管了所有注册方法。
第一步我们需要调用 `doServiceRegister`，然后调用`doServiceInstanceRegister`.

1. 首先，调用 `doServiceRegister`，输入是**服务名称（serviceName）**，这可以定义为任何 UTF-8 编码的字符串。
返回的结果是一个键值对（KeyValue），**服务名称（serviceName）** 作为键，**服务 ID（service id）** 作为值。支持批量注册。
1. 拥有了**服务 ID**之后，使用 `doServiceInstanceRegister` 来进行实例注册。输入是**服务 ID（service id）**，**全剧唯一标识（UUID）**，
和**注册时间（register time）**. 在整个分布式环境下，UUID 都应该保持唯一。返回的值仍然是一个键值对，键为**UUID**，值为**服务实例 ID（service instance id）**。
同样支持批量操作。

对于注册来说，最重要的是要注意这个过程应该是在后台异步进行的，所以返回的值可能是 **null**。在大多数情况下，
你需要设置一个定时器来循环地调用这些服务，直到你得到一个非空的返回值。建议循环周期为 10 秒。

该操作还支持批量进行，尽管对于大多数语言的代理/SDK 来说，没有什么批量注册的使用场景。
我们建议在返回结果中提取服务名 `serviceName` 和唯一键 `UUID`，对比你所期望获得的服务和 UUID，避免混淆同一个批次中的不同注册结果。

### 第 2 步. 发送追踪数据和指标

一旦你有了追踪 ID 和追踪实例 ID，你就可以发送追踪数据和指标数据了。目前：

1. 对于 SkyWalking 原生追踪数据格式，我们有 `TraceSegmentReportService#collect` 方法
2. 对于 SkyWalking 原生 JVM 格式，我们有 `JVMMetricReportService#collect` 方法

对于追踪格式来说，有以下几点需要注意：

1. 段（Segment）是 SkyWalking 中的概念，它需要包括在某个操作系统进程（根据语言不同，通常也是编程语言中的一个线程）中的每个请求的所有 span。
2. Span 有 3 中不同的组。

* EntrySpan
EntrySpan 代表一个服务提供者，也是服务端的一个端点。对于一个 APM 系统来说，我们更关注应用服务。所以几乎所有服务和消息队列消费者都是 EntrySpan。

* LocalSpan
LocalSpan 代表一个普通的 Java 方法，与远程服务无关，也不是一个消息队列的生产者/消费者，也不是一个服务（如 HTTP 服务）提供者和消费者。

* ExitSpan
ExitSpan 代表服务的一个客户端或一个消息队列的生产者，与 SkyWalking 早期的 `LeafSpan` 类似。
如：通过 JDBC 访问数据库时，从 Redis/Memcached 读取数据都被归类到 ExitSpan 中。

3. Span 父信息称为引用（Reference），其被包含在 Span 之中。Reference 运载了许多的字段，
包括但不限于追踪 ID（trace id），父段 ID（parent segment id），span id。其他还包括**入口服务实例 ID（entry service instance id）**，
**父服务实例 ID（parent service instance id）**，**入口端点（entry endpoint）**，**父端点（parent endpoint）**和**网络地址（network address）**。
参考[跨进程传播的头部协议 v2](Skywalking-Cross-Process-Propagation-Headers-Protocol-v2.md)可以了解到如何获取全部的这些字段。

4. 上行数据中的 `segment` 是 TraceSegmentObject 的二进制字节流。

### 第 3 步. 保活

`ServiceInstancePing#doPing` 方法必须每隔几秒钟就被调用一次，使得后端知道这个服务实例还存活着。
此方法需要 `doServiceInstanceRegister` 中使用的**服务实例 ID（service instance id）** 和 **UUID**。
