# 观测分析语言(Observability Analysis Language, OAL)

在流模式(Streaming mode)下, SkyWalking 提供了 OAL 来分析流入的数据.

OAL 聚焦于服务, 服务实例以及端点的度量指标, 因此 OAL 非常易于学习和使用.

6.3版本以后, OAL引擎嵌入在OAP服务器运行时中，称为“OAL -rt”(OAL运行时)。
OAL脚本现在位于' /config '文件夹，用户可以简单地改变和重新启动服务器，使其有效。
但是，OAL脚本仍然是编译语言，OAL运行时动态生成java代码。
您可以在system env上打开set ' SW_OAL_ENGINE_DEBUG=Y '，查看生成了哪些类。

## 语法

OAL 脚本文件应该以 `.oal` 为后缀.
```
// Declare the metrics.
METRICS_NAME = from(SCOPE.(* | [FIELD][,FIELD ...]))
[.filter(FIELD OP [INT | STRING])]
.FUNCTION([PARAM][, PARAM ...])

// Disable hard code 
disable(METRICS_NAME);
```

## 域(Scope)

分析对象包括所有、服务、服务实例、端点、服务关系、服务实例关系、端点关系`. 
当然还有一些二级域, 他们都属于以上某个一级域.

阅读 [Scope Definitions](scope-definitions.md), 你可以在该文中找到所有域和字段定义.

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

- `histogram`. Read [Heatmap in WIKI](https://en.wikipedia.org/wiki/Heat_map)
> All_heatmap = from(All.latency).histogram(100, 20);
在这个例子中 传入请求的热力图参数描述如下
参数(1)是计算延迟的精度，例如在上面的例子中，在101-200ms组中，113ms和193ms被认为是相同的.
参数(2)是分组数量。在上述情况下，一共有21组数据分别为0-100ms, 101-200ms，…1901 - 2000ms, 2000ms以上.

- `apdex`. Read [Apdex in WIKI](https://en.wikipedia.org/wiki/Apdex)
> service_apdex = from(Service.latency).apdex(name, status);
在本例中，该属性描述的是服务的应用性能指数评分.
参数(1)是服务名称，该阈值可以在配置文件service-apdex-threshold.yml中定义.
The parameter (2) is the status of this request. The status(success/failure) effects the Apdex calculation.

- `p99`, `p95`, `p90`, `p75`, `p50`. Read [percentile in WIKI](https://en.wikipedia.org/wiki/Percentile)
> all_percentile = from(All.latency).percentile(10);


**百分位**是自7.0版本引入的第一个多值度量。由于有多个值，可以通过' getMultipleLinearIntValues ' GraphQL查询进行查询。
在本例中，所有传入请求的“p99”、“p95”、“p90”、“p75”、“p50”。参数是p99延迟计算的精度，如在上述情况下，120ms和124被认为是相同的。
在7.0.0之前，使用' p99 '、' p95 '、' p90 '、' p75 '、' p50 '函数分别计算指标。在7.x版本仍然支持，但不推荐且不包括在正式脚本。
> All_p99 = from(All.latency).p99(10);

在本例中，p99为所有传入请求的值。参数是p99延迟计算的精度，如在上述情况下，120ms和124被认为是相同的。


## 度量指标名称

存储实现, 报警以及查询模块的度量指标名称. SkyWalking 内核支持自动类型推断.

## 组(Group)

所有度量指标数据都会使用 Scope.ID 和最小时间桶(min-level time bucket) 进行分组.

- 在端点(Endpoint)域中, Scope.ID = Endpoint 的 ID(基于服务及其端点的唯一标志).


# #禁用
“Disable”是OAL中的高级语句，只在特定情况下使用。
一些聚合和度量是通过核心硬代码定义的，
这个“禁用”语句是设计用来让它们停止活动的，
比如' segment '， ' top_n_database_statement '。
在默认情况下，没有人是禁用的。

## 示例

```
// 计算 Endpoint1 和 Endpoint2 的 p99 值
Endpoint_p99 = from(Endpoint.latency).filter(name in ("Endpoint1", "Endpoint2")).summary(0.99)

// 计算端点名以 `serv` 开头的端点的 p99 值
serv_Endpoint_p99 = from(Endpoint.latency).filter(name like ("serv%")).summary(0.99)

// 计算每个端点的响应平均时长
Endpoint_avg = from(Endpoint.latency).avg()

// 计算每个端点 p50, p75, p90, p95 and p99 的延迟柱状图, 每隔 50 毫秒一条柱
Endpoint_percentile = from(Endpoint.latency).percentile(10)

// 统计每个服务响应状态为 true 的百分比
Endpoint_success = from(Endpoint.*).filter(status = "true").percent()

// 统计每个服务响应码在 [200, 299] 之间的百分比
Endpoint_200 = from(Endpoint.*).filter(responseCode like "2%").percent()

// 统计每个服务响应码在 [500, 599] 之间的百分比
Endpoint_500 = from(Endpoint.*).filter(responseCode like "5%").percent()

// 统计每个服务的调用总量
EndpointCalls = from(Endpoint.*).sum()

disable(segment);
disable(endpoint_relation_server_side);
disable(top_n_database_statement);
```
