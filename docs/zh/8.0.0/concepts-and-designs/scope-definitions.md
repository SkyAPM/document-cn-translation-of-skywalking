# 作用域(Scopes)和字段(Fields)

通过使用聚合函数，请求将会在每个域内以时间和 **是否分组键(Group Key)** 进行分组。

## `全局`作用域

| 名称 | 备注 | 是否分组键 | 数据类型 |
|---|---|---|---|
| name  | 表示每个请求对应的服务名称  |   | string |
| serviceInstanceName  | 表示引用的服务实例 id 的名称  |   | string |
| endpoint  | 表示每个请求的端路径  |   | string |
| latency  | 表示每个请求的耗时 |   |  int(单位：毫秒)  |
| status  | 表示请求成功还是失败的状态  |   | bool(true 表示成功)  |
| responseCode | 如果该请求是 HTTP 请求, 则表示 HTTP 响应的状态码. 如 200, 404, 302 | | int |
| type | 表示请求类型，例如: Database, HTTP, RPC, gRPC | | enum |

## `服务`作用域

从服务的每个请求计算度量数据。

| 名称 | 备注 | 是否分组键 | 数据类型 |
|---|---|---|---|
| id | 表示服务的唯一标识 | 是 | int |
| name | 表示服务的名称 | | string |
| serviceInstanceName | 表示引用的服务实例 id 的名称 | | string |
| endpointName | 表示端点的名称, 如 HTTP URI 的完整路径 | | string |
| latency | 表示每个请求的耗时 | | int |
| status | 表示请求成功还是失败的状态 | | bool(true 表示成功)  |
| responseCode | 如果该请求是 HTTP 请求, 表示 HTTP 请求的响应码 | | int|
| type | 表示每个请求的类型, 如: Database(数据库), HTTP, RPC, gRPC | | enum |

## `服务实例`作用域

从服务实例的每个请求计算度量数据。

| 名称 | 备注 | 是否分组键 | 数据类型 |
|---|---|---|---|
| id | 表示服务实例的唯一标识，通常是一个数值 | 是 | int |
| name |  表示服务实例名称。 例如 `ip:port@Service Name`.  **提醒**: 现在原生代理使用 `processId@Service name` 作为服务实例名称, 当需要在聚合中设置筛选器时，这个毫无用处。 | | string|
| serviceName | 表示服务的名称 | | string |
| endpointName | 表示端点的名称, 如 HTTP URI 的完整路径. | | string|
| latency | 表示每个请求的耗时 | | int |
| status | 表示请求成功还是失败的状态 | | bool(true 表示成功) |
| responseCode | 如果该请求是 HTTP 请求, 则表示 HTTP 响应的状态码. 如 200, 404, 302. | | int |
| type | 表示请求类型，例如: Database, HTTP, RPC, gRPC | | enum |

### `服务实例`二级作用域

当服务实例是一个 JVM 并且通过 javaagent 收集时, 计算服务实例的度量指标.

#### 1. `ServiceInstanceJVMCPU` 作用域

| 名称 | 备注 | 是否分组键 | 数据类型 |
|---|---|---|---|
| id | 表示服务实例的唯一标识，通常是一个数值 | 是 | int |
| name |  表示服务实例名称。 例如 `ip:port@Service Name`.  **提醒**: 现在原生代理使用 `processId@Service name` 作为服务实例名称, 当需要在聚合中设置筛选器时，这个毫无用处。 | | string|
| serviceName | 表示服务的名称 | | string |
| usePercent | 表示 CPU 耗时百分比 | | double|

#### 2. `ServiceInstanceJVMMemory` 作用域

| 名称 | 备注 | 是否分组键 | 数据类型 |
|---|---|---|---|
| id | 表示服务实例的唯一标识，通常是一个数值 | 是 | int |
| name |  表示服务实例名称。 例如 `ip:port@Service Name`.  **提醒**: 现在原生代理使用 `processId@Service name` 作为服务实例名称, 当需要在聚合中设置筛选器时，这个毫无用处。 | | string|
| serviceName | 表示服务的名称 | | string |
| heapStatus | 表示该指标是否是堆的指标 | | bool |
| init | 参考 JVM 文档 | | long |
| max | 参考 JVM 文档 | | long |
| used | 参考 JVM 文档 | | long |
| committed | 参考 JVM 文档 | | long |

#### 3. `ServiceInstanceJVMMemoryPool` 作用域

| 名称 | 备注 | 是否分组键 | 数据类型 |
|---|---|---|---|
| id | 表示服务实例的唯一标识，通常是一个数值 | 是 | int |
| name |  表示服务实例名称。 例如 `ip:port@Service Name`.  **提醒**: 现在原生代理使用 `processId@Service name` 作为服务实例名称, 当需要在聚合中设置筛选器时，这个毫无用处。 | | string|
| serviceName | 表示服务的名称 | | string |
| poolType | 根据不同的 JVM 版本, 可能包括 CODE_CACHE_USAGE, NEWGEN_USAGE, OLDGEN_USAGE, SURVIVOR_USAGE, PERMGEN_USAGE, METASPACE_USAGE | | enum |
| init | 参考 JVM 文档 | | long |
| max | 参考 JVM 文档 | | long |
| used | 参考 JVM 文档 | | long |
| committed | 参考 JVM 文档 | | long |

#### 4. `ServiceInstanceJVMGC` 作用域

| 名称 | 备注 | 是否分组键 | 数据类型 |
|---|---|---|---|
| id | 表示服务实例的唯一标识，通常是一个数值 | 是 | int |
| name |  表示服务实例名称。 例如 `ip:port@Service Name`.  **提醒**: 现在原生代理使用 `processId@Service name` 作为服务实例名称, 当需要在聚合中设置筛选器时，这个毫无用处。 | | string|
| serviceName | 表示服务的名称 | | string |
| phrase | 包括 NEW/OLD | | Enum |
| time | GC 耗时 | | long |
| count | GC 次数 | | long |

## `端点` 作用域

计算服务中每个端点请求的度量指标。

| 名称 | 备注 | 是否分组键 | 数据类型 |
|---|---|---|---|
| id | 表示端点的唯一标识，通常是一个数值 | 是 | int |
| name | 表示端点的名称, 如 HTTP URI 的完整路径. | | string |
| serviceName | 表示服务的名称 | | string |
| serviceInstanceName | 表示引用的服务实例 id 的名称. | | string |
| latency | 表示每个请求的耗时 | | int |
| status | 表示请求成功还是失败的状态| | bool(true 表示成功) |
| responseCode | 如果该请求是 HTTP 请求, 则表示 HTTP 响应的状态码. 如 200, 404, 302. | | int |
| type | 表示请求类型，例如: Database, HTTP, RPC, gRPC | | enum |

## `服务关系` 作用域

计算服务与服务之间每个请求的度量指标。

| 名称 | 备注 | 是否分组键 | 数据类型 |
|---|---|---|---|
| sourceServiceId | 表示源服务的 ID | 是 | int |
| sourceServiceName | 表示源服务的名称 | | string |
| sourceServiceInstanceName | 表示源服务实例名称 | | string |
| destServiceId | 表示目标服务的 ID | 是 | string |
| destServiceName | 表示目标服务的名称 | | string |
| destServiceInstanceName | 表示目标服务实例名称 | | string|
| endpoint | 表示本次调用中使用的端点 | | string
| componentId | 表示本次调用中使用到的组件 ID | 是 | string
| latency | 表示每个请求的耗时 | | int |
| status | 表示请求成功还是失败的状态| | bool(true 表示成功) |
| responseCode | 如果该请求是 HTTP 请求, 则表示 HTTP 响应的状态码. 如 200, 404, 302. | | int |
| type | 表示请求类型，例如: Database, HTTP, RPC, gRPC | | enum |
| detectPoint | Represent where is the relation detected. Values: client, server, proxy. | 是 | enum|

## `服务实例关系` 作用域

计算服务实例与服务实例之间每个请求的度量指标。

| 名称 | 备注 | 是否分组键 | 数据类型 |
|---|---|---|---|
| sourceServiceInstanceId | 表示源服务实例的 ID | 是 | int|
| sourceServiceName | 表示源服务的名称 | | string |
| sourceServiceInstanceName | 表示源服务实例名称 | | string |
| destServiceName | 表示目标服务的名称 | | |
| destServiceInstanceId | 表示目标服务实例 ID | 是 | int|
| destServiceInstanceName | 表示目标服务实例名称 | | string |
| endpoint | 表示本次调用中使用的端点 | | string
| componentId | 表示本次调用中使用到的组件 ID | 是 | string
| latency | 表示每个请求的耗时 | | int |
| status | 表示请求成功还是失败的状态| | bool(true 表示成功) |
| responseCode | 如果该请求是 HTTP 请求, 则表示 HTTP 响应的状态码. 如 200, 404, 302. | | int |
| type | 表示请求类型，例如: Database, HTTP, RPC, gRPC | | enum |
| detectPoint | Represent where is the relation detected. Values: client, server, proxy. | 是 | enum|

## `端点关系` 作用域

计算端点之间依赖的度量指标。这种关系通常难以探测，, 而且还要依赖于追踪库来传播上个端点。所以 `EndpointRelation` 范围聚合，仅仅在使用 SkyWalking 原生代理进行追踪的情况下才有效，包括自动打点代理(如 Java, .NET), OpenCensus SkyWalking exporter 以及其他传播追踪上下文的实现.

| 名称 | 备注 | 是否分组键 | 数据类型 |
|---|---|---|---|
| endpointId | 表示此依赖中的父端点 ID | 是 | int |
| endpoint | 表示此依赖中的父端点 | | string|
| childEndpointId | 表示此关系中, 被父端点(第一行)使用的端点的 ID | 是 | int|
| childEndpoint| 表示此关系中, 被父端点(第二行)使用的端点 | | string |
| rpcLatency | 表示从父端点到子端点之间 RPC 的耗时, 不包括父端点本身的耗时 | | int |
| componentId | 表示本次调用中使用到的组件 ID | 是 | string
| status | 表示请求成功还是失败的状态| | bool(true 表示成功) |
| responseCode | 如果该请求是 HTTP 请求, 则表示 HTTP 响应的状态码. 如 200, 404, 302. | | int |
| type | 表示请求类型，例如: Database, HTTP, RPC, gRPC | | enum |
| detectPoint | Represent where is the relation detected. Values: client, server, proxy. | 是 | enum|
