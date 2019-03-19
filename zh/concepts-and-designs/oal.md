# 观测分析语言(Observability Analysis Language, OAL)

在流模式(Streaming mode)下, SkyWalking 提供了 OAL 来分析流入的数据.

OAL 聚焦于服务, 服务实例以及端点的度量指标, 因此 OAL 非常易于学习和使用.

考虑到性能, 可读性以及可调试性, OAL 被定义成一种编译语言.
OAL 脚本会在打包阶段被编译为正常的 Java 代码.

## 语法

OAL 脚本文件应该以 `.oal` 为后缀.

```
METRIC_NAME = from(SCOPE.(* | [FIELD][,FIELD ...]))
[.filter(FIELD OP [INT | STRING])]
.FUNCTION([PARAM][, PARAM ...])
```

## 域(Scope)

主要的**域(Scope)**包括 `All`, `Service`, `ServiceInstance`, `Endpoint`, `ServiceRelation`, ServiceInstanceRelation`, `EndpointRelation`.
当然还有一些二级域, 他们都属于以上某个一级域.

阅读[Scope Definitions](scope-definitions.md)一文, 你可以在该文中找到所有域和字段.

## 过滤器(Filter)

使用在使用过滤器的时候, 通过指定字段名或表达式来构建字段值的过滤条件.

表达式可以使用 `and`, `or` 和 `()` 进行组合.
操作符包含 `=`, `!=`, `>`, `<`, `in (v1, v2, ...`, `like "%..."`, 他们可以基于字段类型进行类型检测,
		如果类型不兼容会在编译/代码生成期间报错.

## 聚合函数(Aggregation Function)

默认的聚合函数有 SkyWalking OAP 核心实现, 并可自由扩展更多函数.

提供的函数

- `longAvg`. 某个域实体所有输入的平均值. 输入字段必须是 `long` 类型.
> instance_jvm_memory_max = from(ServiceInstanceJVMMemory.max).longAvg();

在这个例子中, 输入是 `ServiceInstanceJVMMemory` 域的每个请求, 平均值是基于字段 `max` 进行求值的.

- `doubleAvg`. 某个域实体的所有输入的平均值. 输入的字段必须是 `double` 类型.
> instance_jvm_cpu = from(ServiceInstanceJVMCPU.usePercent).doubleAvg();

在这个例子中, 输入是 `ServiceInstanceJVMCPU` 域的每个请求, 平均值是基于 `usePercent` 字段进行求值的.

- `percent`. 对于输入中匹配指定条件的百分比数.
> endpoint_percent = from(Endpoint.*).percent(status == true);

在这个例子中, 输入是每个端点的请求, 条件是 `endpoint.status == true`.

- `sum`. 某个域实体的调用总数.
> Service_Calls_Sum = from(Service.*).sum();

本例统计每个服务的调用数.

- `p99`, `p95`, `p90`, `p75`, `p50`. 参考[Wiki 上对 p99 的解释](https://en.wikipedia.org/wiki/Percentile)
> All_p99 = from(All.latency).p99(10);

本例中统计所有接入请求的 p99 值.

- `thermodynamic`. 参考[Wiki 上对 HeatMap 的解释](https://en.wikipedia.org/wiki/Heat_map))
> All_heatmap = from(All.latency).thermodynamic(100, 20);

本例中统计所有进入请求的热力学热点图.

## 度量指标名称

存储实现, 报警以及查询模块的度量指标名称. SkyWalking 内核支持自动类型推断.

## 组(Group)

所有度量指标数据都会使用 Scope.ID 和最小时间桶(min-level time bucket) 进行分组.

- 在端点(Endpoint)域中, Scope.ID = Endpoint 的 ID(基于服务及其端点的唯一标志).

## 示例

```
// 计算 Endpoint1 和 Endpoint2 的 p99 值
Endpoint_p99 = from(Endpoint.latency).filter(name in ("Endpoint1", "Endpoint2")).summary(0.99)

// 计算端点名以 `serv` 开头的端点的 p99 值
serv_Endpoint_p99 = from(Endpoint.latency).filter(name like ("serv%")).summary(0.99)

// 计算每个端点的响应平均时长
Endpoint_avg = from(Endpoint.latency).avg()

// 计算每个端点的延迟柱状图, 每隔 50 毫秒一条柱
// 在用户界面中, 匹配此度量指标的都会显示热力学图
Endpoint_histogram = from(Endpoint.latency).histogram(50)

// 统计每个服务响应状态为 true 的百分比
Endpoint_success = from(Endpoint.*).filter(status = "true").percent()

// 统计每个服务响应码在 [200, 299] 之间的百分比
Endpoint_200 = from(Endpoint.*).filter(responseCode like "2%").percent()

// 统计每个服务响应码在 [500, 599] 之间的百分比
Endpoint_500 = from(Endpoint.*).filter(responseCode like "5%").percent()

// 统计每个服务的调用总量
EndpointCalls = from(Endpoint.*).sum()
```
