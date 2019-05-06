# 域(Scopes) 和 字段(Fields)

使用聚合函数时, 请求将会在每个域内以时间和**分组键(Group Key)**进行分组.

### `All` 所有域

| 名称 | 备注 | 分组键 | 类型 | 
|------|------|--------|------|
| endpoint | 表示每个请求的端路径 |   | string |
| latency | 表示每个请求的耗时 |   |  int(单位: 毫秒)  |
| status | 表示请求成功还是失败 | bool(true 表示成功)  |
| responseCode | 如果该请求是 HTTP 请求, 则表示 HTTP 响应的状态码. 如 200, 404, 302| | int |

### `Service` 服务域

计算服务每个请求的度量指标.

| 名称 | 备注 | 分组键 | 类型 | 
|------|------|--------|------|
| id | 表示服务的唯一标识 | 是 | int |
| name | 表示服务的名称 | | string |
| serviceInstanceName | 表示服务实例 id 所引用的名称 | | string |
| endpointName | 表示端点的名称, 如 HTTP URI 的完整路径 | | string |
| latency | 表示每个请求的耗时 | | int |
| status | 表示请求成功还是失败 | | bool(true 表示成功)  |
| responseCode | 如果该请求是 HTTP 请求, 表示 HTTP 请求的响应码 | | int|
| type | 表示每个请求的类型, 如: Database(数据库), HTTP, RPC, gRPC | | enum |

### `ServiceInstance` 服务实例域

计算服务实例每个请求的度量指标数据.

| 名称 | 备注 | 分组键 | 类型 | 
|------|------|--------|------|
| id | 表示服务实例的唯一标识, 通常是一个数值 | 是 | int |
| name |  表示服务实例的名称, 如 `ip:端口@服务名`. **注意**: 当前的原生代理使用 `进程id@服务名` 作为实例名称, 当你想设置一个过滤器来进行聚合时这个实例名称可能没有什么意义 | | string |
| serviceName | 表示服务的名称 | | string |
| endpointName | 表示端点的名称, 如 HTTP URI 的完整路径 | | string|
| latency | 表示每个请求的耗时 | | int |
| status | 表示请求成功还是失败 | | bool(true 表示成功) |
| responseCode | 如果该请求是 HTTP 请求, 表示 HTTP 请求的响应码 | | int|
| type | 表示每个请求的类型, 如: Database(数据库), HTTP, RPC, gRPC | | enum |

#### `ServiceInstance` 的二级域

当服务实例是一个 JVM 并且通过 javaagent 收集时, 计算服务实例的度量指标.

1. `ServiceInstanceJVMCPU` 域

| 名称 | 备注 | 分组键 | 类型 | 
|------|------|--------|------|
| id | 表示服务实例的唯一标识, 通常是一个数值 | 是 | int |
| name |  表示服务实例的名称, 如 `ip:端口@服务名`. **注意**: 当前的原生代理使用 `进程id@服务名` 作为实例名称, 当你想设置一个过滤器来进行聚合时这个实例名称可能没有什么意义 | | string |
| serviceName | 表示服务的名称 | | string |
| usePercent | 表示 CPU 耗时百分比 | | double|

2. `ServiceInstanceJVMMemory` 域 (服务实例 JVM 内存)

| 名称 | 备注 | 分组键 | 类型 | 
|---|---|---|---|
| id | 表示服务实例的唯一标识, 通常是一个数值 | 是 | int |
| name |  表示服务实例的名称, 如 `ip:端口@服务名`. **注意**: 当前的原生代理使用 `进程id@服务名` 作为实例名称, 当你想设置一个过滤器来进行聚合时这个实例名称可能没有什么意义 | | string |
| serviceName | 表示服务的名称 | | string |
| heapStatus | 表示该指标是否是堆的指标 | | bool |
| init      | 参考 JVM 文档 | | long |
| max       | 参考 JVM 文档 | | long |
| used      | 参考 JVM 文档 | | long |
| committed | 参考 JVM 文档 | | long |

3. `ServiceInstanceJVMMemoryPool` 域 (服务实例 JVM 内存池)

| 名称 | 备注 | 分组键 | 类型 | 
|---|---|---|---|
| id | 表示服务实例的唯一标识, 通常是一个数值 | 是 | int |
| name |  表示服务实例的名称, 如 `ip:端口@服务名`. **注意**: 当前的原生代理使用 `进程id@服务名` 作为实例名称, 当你想设置一个过滤器来进行聚合时这个实例名称可能没有什么意义 | | string |
| serviceName | 表示服务的名称 | | string |
| poolType | 根据不同的 JVM 版本, 可能包括 CODE\_CACHE\_USAGE, NEWGEN\_USAGE, OLDGEN\_USAGE, SURVIVOR\_USAGE, PERMGEN\_USAGE, METASPACE\_USAGE . | | enum |
| init      | 参考 JVM 文档 | | long |
| max       | 参考 JVM 文档 | | long |
| used      | 参考 JVM 文档 | | long |
| committed | 参考 JVM 文档 | | long |

4. `ServiceInstanceJVMGC` 域 (服务实例 JVM 垃圾回收)

| 名称 | 备注 | 分组键 | 类型 | 
|---|---|---|---|
| id | 表示服务实例的唯一标识, 通常是一个数值 | 是 | int |
| name |  表示服务实例的名称, 如 `ip:端口@服务名`. **注意**: 当前的原生代理使用 `进程id@服务名` 作为实例名称, 当你想设置一个过滤器来进行聚合时这个实例名称可能没有什么意义 | | string |
| serviceName | 表示服务的名称 | | string |
| phrase | 包括 NEW/OLD | | Enum |
| time | GC 耗时 | | long |
| count | GC 次数 | | long |

### `Endpoint` 端点域

计算服务中每个端点请求的度量指标.

| 名称 | 备注 | 分组键 | 类型 | 
|---|---|---|---|
| id | 表示端点的唯一标识, 通常是一个数值 | 是 | int |
| name | 表示每个请求的端路径, 如 HTTP URI 的完整路径 | | string |
| serviceName | 表示服务的名称 | | string |
| serviceInstanceName | 表示服务实例 id 所引用的名称 | | string |
| latency | 表示每个请求的耗时 | | int |
| status | 表示请求成功还是失败 | | bool(true 表示成功) |
| responseCode | 如果该请求是 HTTP 请求, 表示 HTTP 请求的响应码 | | int|
| type | 表示每个请求的类型, 如: Database(数据库), HTTP, RPC, gRPC | | enum |

### `ServiceRelation` 服务关系域

计算一个请求在两个服务之间的度量指标.

| 名称 | 备注 | 分组键 | 类型 | 
|---|---|---|---|
| sourceServiceId | 表示源服务的 id | yes | int |
| sourceServiceName | 表示源服务的名称 | | string |
| sourceServiceInstanceName | 表示源服务实例的名称 | | string |
| destServiceId | 表示目标服务的 id | yes | string |
| destServiceName | 表示目标服务的名称 | | string |
| destServiceInstanceName | 表示目标服务实例的名称 | | string|
| endpoint | 表示本次调用中使用的端点 | | string |
| componentId | 表示本次调用中使用到的组件 id | yes | string |
| latency | 表示每个请求的耗时 | | int |
| status | 表示请求成功还是失败 | | bool(true 表示成功) |
| responseCode | 如果该请求是 HTTP 请求, 表示 HTTP 请求的响应码 | | int|
| type | 表示每个请求的类型, 如: Database(数据库), HTTP, RPC, gRPC | | enum |
| detectPoint | 监测点, 表示服务关系在哪里监测到. 取值: client(客户端), server(服务端), proxy(代理端) | yes | enum|

### `ServiceInstanceRelation` 服务实例关系域

计算一个请求在两个服务实例之间的度量指标

| 名称 | 备注 | 分组键 | 类型 | 
|---|---|---|---|
| sourceServiceInstanceId | 表示源服务实例的 id | yes | int|
| sourceServiceName | 表示源服务的名称 | | string |
| sourceServiceInstanceName | 表示源服务实例的名称 | | string |
| destServiceInstanceId | 表示目标服务实例 id | yes | int| 
| destServiceName | 表示目标服务的名称 | | string |
| destServiceInstanceName | 表示目标服务实例的名称 | | string|
| endpoint | 表示本次调用中使用的端点 | | string |
| componentId | 表示本次调用中使用到的组件 id | yes | string |
| latency | 表示每个请求的耗时 | | int |
| status | 表示请求成功还是失败 | | bool(true 表示成功) |
| responseCode | 如果该请求是 HTTP 请求, 表示 HTTP 请求的响应码 | | int|
| type | 表示每个请求的类型, 如: Database(数据库), HTTP, RPC, gRPC | | enum |
| detectPoint | 监测点, 表示服务关系在哪里监测到. 取值: client(客户端), server(服务端), proxy(代理端) | yes | enum|

### `EndpointRelation` 端点关系域

计算两个端点之间的度量指标数据, 这种关系通常难以探测, 而且还要依赖于追踪库来传播上游端点.
因此端点关系域(`EndpointRelation`)仅仅在使用 SkyWalking 原生代理进行追踪的情况下才有效,
包括自动打点代理(如 Java, .NET), OpenCensus SkyWalking 导出器以及其他传播追踪上下文的实现.

| 名称 | 备注 | 分组键 | 类型 | 
|---|---|---|---|
| endpointId | 表示此关系中的父端点 id | yes | int |
| childEndpointId | 表示此关系中, 被父端点(即上一行)使用的端点的 ID | yes | int| 
| endpoint | 表示此关系中的父端点 | | string| 
| childEndpoint| 表示在此关系中, 被父端点(即上一行)使用的端点 | | string |
| rpcLatency | 表示从父端点到子端点之间 RPC 的耗时, 并不包括父端点本身的耗时. | | int |
| componentId | 表示本次调用中使用到的组件 id | yes | string |
| status | 表示请求成功还是失败 | | bool(true 表示成功) |
| responseCode | 如果该请求是 HTTP 请求, 表示 HTTP 请求的响应码 | | int|
| type | 表示每个请求的类型, 如: Database(数据库), HTTP, RPC, gRPC | | enum |
| detectPoint | 监测点, 表示服务关系在哪里监测到. 取值: client(客户端), server(服务端), proxy(代理端) | yes | enum |
