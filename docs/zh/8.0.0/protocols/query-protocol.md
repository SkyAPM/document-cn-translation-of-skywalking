# 查询协议
查询协议在 GraphQL 语法中定义了一组 API，通过 SkyWalking 本机可视化工具或第三方系统(包括Web UI、CLI或私有系统)提供数据查询和交互功能。
查询协议官方库, https://github.com/apache/skywalking-query-protocol.

### 元数据  
元数据包括整个监控服务及其实例、端点等简要信息。
可以使用多种方法查询元数据。
```graphql
extend type Query {
    getGlobalBrief(duration: Duration!): ClusterBrief

    # Normal service related metainfo 
    getAllServices(duration: Duration!): [Service!]!
    searchServices(duration: Duration!, keyword: String!): [Service!]!
    searchService(serviceCode: String!): Service
    
    # Fetch all services of Browser type
    getAllBrowserServices(duration: Duration!): [Service!]!

    # Service intance query
    getServiceInstances(duration: Duration!, serviceId: ID!): [ServiceInstance!]!

    # Endpoint query
    # Consider there are huge numbers of endpoint,
    # must use endpoint owner's service id, keyword and limit filter to do query.
    searchEndpoint(keyword: String!, serviceId: ID!, limit: Int!): [Endpoint!]!
    getEndpointInfo(endpointId: ID!): EndpointInfo

    # Database related meta info.
    getAllDatabases(duration: Duration!): [Database!]!
    getTimeInfo: TimeInfo
}
```

### 拓扑
显示服务或端点的拓扑和依赖关系图。包括直接关系图或全局关系图。

```graphql
extend type Query {
    # Query the global topology
    getGlobalTopology(duration: Duration!): Topology
    # Query the topology, based on the given service
    getServiceTopology(serviceId: ID!, duration: Duration!): Topology
    # Query the instance topology, based on the given clientServiceId and serverServiceId
    getServiceInstanceTopology(clientServiceId: ID!, serverServiceId: ID!, duration: Duration!): ServiceInstanceTopology
    # Query the topology, based on the given endpoint
    getEndpointTopology(endpointId: ID!, duration: Duration!): Topology
}
```

### 指标
指标查询以定义的所有对象为目标 [OAL script](../concepts-and-designs/oal.md). 
您可以根据脚本中的聚合函数以线性或热力学矩阵格式获得指标数据. 

3种类型的指标可以是查询
1. 单一值. 大多数默认指标的类型是单一值，将其视为默认值. `getValues` 和 `getLinearIntValues` 适合此类型.
1. 多值. OAL中定义的一个指标包括多个值计算. 使用 `getMultipleLinearIntValues` 获取所有值. `percentile` 是 OAL 中一个典型的多值函数.
1. Heatmap value. 阅读 [Heatmap in WIKI](https://en.wikipedia.org/wiki/Heat_map) 获取详细信息. `thermodynamic` 仅仅是 OAL 的一个函数. 使用 `getThermodynamic` 获取值.
```graphql
extend type Query {
    getValues(metric: BatchMetricConditions!, duration: Duration!): IntValues
    getLinearIntValues(metric: MetricCondition!, duration: Duration!): IntValues
    # 查询包含多个值的指标类型，并将它们格式化为多个线性值.
    # 这些多个值的序列是在 OAL 计算函数的基础上确定的
    # 例如，我们是否应该查询OAL中func百分位数(50、75、90、95、99)的结果，
    # 然后5行被响应，p50是返回值的第一个元素。
    getMultipleLinearIntValues(metric: MetricCondition!, numOfLinear: Int!, duration: Duration!): [IntValues!]!
    getThermodynamic(metric: MetricCondition!, duration: Duration!): Thermodynamic
}
```

指标定义在 `config/oal/*.oal` 文件中.

### 聚合
聚集查询意味着指标数据在查询阶段需要二次聚集，这使得查询接口具有一些不同的参数.
例如，“TopN”服务列表是一个非常典型的聚合查询，metrics流聚合只计算每个服务的metrics值，但是预期的列表需要按这些值排序metrics数据.

聚合查询只适用于单个值指标.

```graphql
# 聚合查询与指标查询不同.
# 所有聚合查询都需要查询时在后端或存储进行聚合.
extend type Query {
    # TopN是一个聚合查询.
    getServiceTopN(name: String!, topN: Int!, duration: Duration!, order: Order!): [TopNEntity!]!
    getAllServiceInstanceTopN(name: String!, topN: Int!, duration: Duration!, order: Order!): [TopNEntity!]!
    getServiceInstanceTopN(serviceId: ID!, name: String!, topN: Int!, duration: Duration!, order: Order!): [TopNEntity!]!
    getAllEndpointTopN(name: String!, topN: Int!, duration: Duration!, order: Order!): [TopNEntity!]!
    getEndpointTopN(serviceId: ID!, name: String!, topN: Int!, duration: Duration!, order: Order!): [TopNEntity!]!
}
```

### 其它
以下查询针对特定功能，包括跟踪、警报或配置文件.
1. 追踪. 分布式追踪查询.
1. 告警. 通过告警查询，可以了解告警趋势和详细信息.

可以在以下文件找到实际的 GraphQL 查询脚本 `query-protocol` ,[此处](../../../oap-server/server-query-plugin/query-graphql-plugin/src/main/resources).

## 条件
### Duration
Duration 是一种广泛使用的参数类型，因为 APM 数据与时间有关. 解释如下.
步长与精度有关. 
```graphql
# Duration 定义了每个查询操作的开始和结束时间.
# Fields: `start` and `end`
#   表示时间跨度. 每一个都符合这个步骤.
#   ref https://www.ietf.org/rfc/rfc3339.txt
#   时间格式如下
#       `SECOND` step: yyyy-MM-dd HHmmss
#       `MINUTE` step: yyyy-MM-dd HHmm
#       `HOUR` step: yyyy-MM-dd HH
#       `DAY` step: yyyy-MM-dd
#       `MONTH` step: yyyy-MM
# Field: `step`
#   表示精确的时间点
# e.g.
#   if step==HOUR , start=2017-11-08 09, end=2017-11-08 19
#   then
#       指标取自以下时间点
#       2017-11-08 9:00 -> 2017-11-08 19:00
#       在时间跨度中有11个时间点(小时)
input Duration {
    start: String!
    end: String!
    step: Step!
}

enum Step {
    MONTH
    DAY
    HOUR
    MINUTE
    SECOND
}
```
