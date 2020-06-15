# 协议

有以下两种类型的协议.

- [**探针协议**](#探针协议). 包括对探针如何发送收集到的度量数据、跟踪信息以及涉及到的每个实体格式的描述和定义。

- [**查询协议**](#查询协议). 服务后端给 SkyWalking 自有的 UI 和任何第三方 UI 提供了查询的能力. 这些查询都是基于 GraphQL 的.


## 探针协议

它们也是与探针的组相关的, 为了理解这一点, 请参考[概念和设计](../concepts-and-designs/README.md)一文.
这些探针组是**基于原生语言代理协议**, **服务网格协议**以及其他第三方打点协议.

## 注册协议

包含服务, 服务实例, 网络地址以及端点元数据注册.
注册的目的是:
1. 对于服务, 网络地址和端点, 注册之后将会返回注册对象的一个唯一 ID, 通常是一个整数. 探针可以使用这个 ID 来替代字符串文本, 达到数据压缩的功能. 进一步讲, 有些协议只接收 ID.
2. 对于服务实例, 注册之后将会为每个新的实例返回一个新的唯一 ID. 每个服务实例必须包含服务 ID.

### 基于语言的原生代理协议

有以下两种协议可以让基于语言的代理在分布式环境下工作.
1. **跨进程传播的头部协议**是一种有线数据格式, 代理和 SDK 通常使用 RPC 请求以及 HTTP/MQ/HTTP2 请求头来运载数据.
远程代理将在请求处理器中接收这些数据, 并将上下文绑定到该请求中.
2. **追踪数据协议**是一种离线数据格式, 代理和 SDK 使用这种数据格式来发送追踪数据和指标数据到 SkyWalking 或其他兼容的后端。

为了兼容性, 请求头有两种格式. 默认是使用 v2.
* [跨进程传播的请求头 v2](Skywalking-Cross-Process-Propagation-Headers-Protocol-v2.md) 是自 6.0.0-beta 版本开始, 针对在线上下文传播的一种新的协议.
它将在以后替代老的 **SW3** 协议, 目前来说它们二者都是支持的.
* [跨进程传播的请求头 v1](Skywalking-Cross-Process-Propagation-Headers-Protocol-v1.md) 是针对在线传播的协议.
遵循此协议的不同进程的追踪数据段可以被连接起来.

自 SkyWalking v6.0.0-beta 开始, SkyWalking 代理和后端都使用第二版的追踪数据协议(Trace Data Protocol v2), 后端仍然支持 v1 版本的协议.
* [SkyWalking 追踪数据协议 v2](Trace-Data-Protocol-v2.md) 定义了代理和后端之间通讯的方式和格式.
* [SkyWalking 追踪数据协议 v1](Trace-Data-Protocol.md). 该协议用于旧的版本中. 目前仍然支持.

### 服务网格探针协议

Sidecar 中的探针或代理可以使用该协议发送数据到后端. 该服务通过 gRPC 实现, 需要以下关键信息:

1. 在服务两侧都需要服务名或 ID
2. 在服务两侧都需要服务实例名称或 ID
3. 端点. HTTP 中的 URI, gRPC 中的方法完整签名.
4. 时延. 以毫秒为单位
5. HTTP 中的响应码
6. 状态. 成功或失败
7. 协议. HTTP, gRPC 等
8. 监测点. 在服务网格 sidecar 中, `client` 或 `server`。 在普通的 L7 代理中, 值是 `proxy`.

### 第三方打点协议

SkyWalking 并不定义第三方打点协议. 它们只是协议和数据格式, SkyWalking 兼容这些协议和数据格式, 且可以接收它们. SkyWalking 一开始就支持 Zipkin v1, v2 数据格式.
后端遵循模块化原则, 所以要扩展新的接收器以支持新的协议和格式是非常容易的.

## 查询协议

查询协议遵循 GraphQL 语法, 提供了数据查询能力, 这都取决于你要分析的指标.

实际的查询 GraphQL 脚本可以在 `query-protocol` [文件夹](https://github.com/apache/skywalking/tree/master/oap-server/server-query-plugin/query-graphql-plugin/src/main/resources)内找到.