# 协议

有以下两种类型的协议.

- [**探针协议**](#probe-protocols). 包括了对打点代理如何发送所收集的指标数据和追踪的描述和定义, 以及各自实体的格式.

- [**查询协议**](#query-protocol). 服务后端给 SkyWalking 自有的 UI 和任何第三方 UI 提供了查询的能力. 这些查询都是基于 GraphQL 的.


## 探针协议

它们也是与探针的组相关的, 为了理解这一点, 请参考[概念和设计](../concepts-and-designs/README.md)一文.
这些探针组是**基于原生语言代理协议**, **服务网格协议**以及其他第三方打点协议.

## 注册协议

包含服务, 服务实例, 网络地址以及端点元数据注册.
注册的目的是:
1. 对于服务, 网络地址和端点, 注册之后将会返回注册对象的一个唯一 ID, 通常是一个整数. 探针可以使用这个 ID 来替代字符串文本. 达到数据压缩的功能. 进一步讲, 有些协议只接收 ID.
1. 对于服务实例, 注册之后将会为每个新的实例返回一个新的唯一 ID. 每个服务实例必须包含服务 ID.

### 基于语言的原生代理协议

有以下两种协议可以让基于语言的代理在分布式环境下工作.
1. **跨进程传播的头部协议**是一种有线数据格式, 代理和 SDK 通常使用 RPC 请求以及 HTTP/MQ/HTTP2 请求头来运载数据.
远程代理将在请求处理器中接收这些数据, 并将上下文绑定到该请求中.
1. **追踪数据协议**是一种离线数据格式, 代理和 SDK 使用这种数据格式来发送追踪数据和指标数据到 SkyWalking 或其他兼容的后端。

为了兼容性, 请求头有两种格式. 默认是使用 v2.
* [跨进程传播的请求头 v2](Skywalking-Cross-Process-Propagation-Headers-Protocol-v2.md) 是自 6.0.0-beta 版本开始, 针对在线上下文传播的一种新的协议.
它将在以后替代老的 **SW3** 协议, 目前来说它们二者都是支持的.
* [跨进程传播的请求头 v1](Skywalking-Cross-Process-Propagation-Headers-Protocol-v1.md) 是针对在线传播的协议.
遵循此协议的不同进程的追踪数据段可以被连接起来.

自 SkyWalking v6.0.0-beta 开始, SkyWalking 代理和后端都使用第二版的追踪数据协议(Trace Data Protocol v2), 后端仍然支持 v1 版本的协议.
* [SkyWalking 追踪数据协议 v2](Trace-Data-Protocol-v2.md) 定义了代理和后端之间通讯的方式和格式.
* [SkyWalking 追踪数据协议 v1](Trace-Data-Protocol.md). 该协议用于旧的版本中. 目前仍然支持.

### 服务网格探针协议

Sidecar 中的探针或代理可以使用该协议发送数据到后端. 该服务通过 gRPC 实现, 需要一下关键信息:

1. 在服务两侧都需要服务名或 ID
1. 在服务两侧都需要服务实例名称或 ID
1. 端点. HTTP 中的 URI, gRPC 中的方法完整签名.
1. 时延. 以毫秒为单位
1. HTTP 中的响应吗
1. 状态. 成功或失败
1. 协议. HTTP, gRPC 等
1. 监测点. 在服务网格 sidecar 中, `client` 或 `server`。 在普通的 L7 代理中, 值是 `proxy`.

### 第三方打点协议

SkyWalking 并不定义第三方打点协议. 它们只是协议和数据格式, SkyWalking 兼容这些协议和数据格式, 且可以接收它们. SkyWalking 一开始就支持 Zipkin v1, v2 数据格式.
后端遵循模块化原则, 所以要扩展新的接收器以支持新的协议和格式是非常容易的.

## 查询协议

查询协议遵循 GraphQL 语法, 提供了数据查询能力, 这都取决于你要分析的指标.

提供了以下 5 个纬度的数据:
1. 元数据. 元数据包括了底层监控的所有服务及其实例, 端点等的简要信息. 可以使用多种方式来查询这些元数据.
1. 拓扑图. 展示服务或端点的拓扑图和依赖图. 包括了直接关系或全局图.
1. 指标. 指标查询针对所有在 [OAL 脚本](../concepts-and-designs/oal.md) 中定义的对象. 你可以在脚本中基于聚合函数获得指标数据的线性矩阵或热力学矩阵格式.
1. 聚合. 聚合查询意味着在查询阶段需要对指标数据进行再次聚合, 这将使得查询接口需要一些不一样的参数. 例如 `TopN` 个服务就是一个非常典型的聚合查询,
指标流聚合仅仅计算每个服务的度量值, 但是期望的列表需要将指标数据进行排序.
1. 追踪. 查询分布式追踪
1. 报警. 通过报警查询, 你可以得到报警趋势和细节.

实际的查询 GraphQL 脚本可以在 `query-protocol` [文件夹](../../../oap-server/server-query-plugin/query-graphql-plugin/src/main/resources)内找到.

以下是目前已有的指标名称, 基于[official_analysis.oal](../../../oap-server/generated-analysis/src/main/resources/official_analysis.oal)

**全局指标**
- all_p99, 所有服务响应时间的 p99 值
- all_p95
- all_p90
- all_p75
- all_p70
- all_heatmap, 所有服务响应时间的热点图 

**服务指标**
- service_resp_time, 服务的平均响应时间
- service_sla, 服务的成功率
- service_cpm, 服务每分钟调用次数
- service_p99, 服务响应时间的 p99 值
- service_p95
- service_p90
- service_p75
- service_p50

**服务实例指标**
- service_instance_sla, 服务实例的成功率
- service_instance_resp_time, 服务实例的平均响应时间
- service_instance_cpm, 服务实例每分钟调用次数

**端点指标**
- endpoint_cpm, 端点每分钟调用次数
- endpoint_avg, 端点平均响应时间
- endpoint_sla, 端点成功率
- endpoint_p99, 端点响应时间的 p99 值
- endpoint_p95
- endpoint_p90
- endpoint_p75
- endpoint_p50

**JVM 指标**, JVM 相关的指标, 只有当 javaagent 启用时才有效
- instance_jvm_cpu
- instance_jvm_memory_heap
- instance_jvm_memory_noheap
- instance_jvm_memory_heap_max
- instance_jvm_memory_noheap_max
- instance_jvm_young_gc_time
- instance_jvm_old_gc_time
- instance_jvm_young_gc_count
- instance_jvm_old_gc_count

**服务关系指标**, 代表服务之间调用的指标
指标的 ID 只能在拓扑图查询中获取

- service_relation_client_cpm, 在客户端每分钟检测到的调用次数
- service_relation_server_cpm, 在服务端每分钟检测到的调用次数
- service_relation_client_call_sla, 在客户端检测到的成功率
- service_relation_server_call_sla, 在服务端检测到的成功率
- service_relation_client_resp_time, 在客户端检测到的平均响应时间
- service_relation_server_resp_time, 在服务端检测到的平均响应时间

**端点关系指标**, 代表相互依赖的端点之间的指标. 只有在追踪代理启用时有效.
指标 ID 只能在拓扑查询中获得.
- endpoint_relation_cpm
- endpoint_relation_resp_time
