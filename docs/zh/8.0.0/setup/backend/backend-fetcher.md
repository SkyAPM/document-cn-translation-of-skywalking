# Open Fetcher

Fetcher 是 SkyWalking 后台的一个概念。当从目标系统读数据时，使用拉模式相比 [receiver](backend-receivers.md) 更合适。这个模式典型的存在一些度量 SDK 中，例如 Prometheus。

## Prometheus Fetcher

```yaml
prometheus-fetcher:
  selector: ${SW_PROMETHEUS_FETCHER:default}
  default:
    active: ${SW_PROMETHEUS_FETCHER_ACTIVE:false}
```

### 配置文件

Prometheus fetcher 通过配置文件设置。配置文件定义了所有与 fetching 服务及实例关联的内容，以及要加载哪些规则文件。

OAP 可以在启动时加载配置。如果新的配置不是格式规范的，OAP 启动失败。这些文件在 `$CLASSPATH/fetcher-prom-rules` 目录下。

文件使用 YAML 格式编写，有下面描述方案定义。方括号表示参数是可选择的。

一个完整的样例可以查看 [此处](../../../../oap-server/server-bootstrap/src/main/resources/fetcher-prom-rules/self.yaml)

通用占位符如下：

- `<duration>`: 将被解析为持续时间的文本表示。The formats accepted are based on the ISO-8601 duration format `PnDTnHnMn.nS` with days considered to be exactly 24 hours.
- `<labelname>`: 匹配正则表达式的字符串 [a-zA-Z_][a-zA-Z0-9_]*
- `<labelvalue>`: unicode 字符的字符串
- `<host>`: 由主机名或IP加可选端口号组成的有效字符串
- `<path>`: 有效的 URL 路径
- `<string>`: 普通的字符串

```yaml
# How frequently to fetch targets.
fetcherInterval: <duration>
# Per-fetch timeout when fetching this target.
fetcherTimeout: <duration>
# The HTTP resource path on which to fetch metrics from targets.
metricsPath: <path>
#Statically configured targets.
staticConfig:
  # The targets specified by the static config.
  targets:
    [ - <host> ]
  # Labels assigned to all metrics fetched from the targets.
  labels:
    [ <labelname>: <labelvalue> ... ]
# Metrics rule allow you to recompute queries.
metricsRules:
   [ - <metric_rules> ]
```

#### <metric_rules>

```yaml
# The name of rule, which combinates with a prefix 'meter_' as the index/table name in storage.
name: <string>
# Scope should be one of SERVICE, INSTANCE and ENDPOINT.
scope: <string>
# The transformation operation from prometheus metrics to skywalking ones. 
operation: <operation>
# The prometheus sources of the transformation operation.
sources:
  # The prometheus metric family name 
  <string>:
    # Function for counter, one of INCREASE, RATE, and IRATE.
    [counterFunction: <string> ]
    # The range of a counterFunction.
    [range: <duration>]
    # The percentile rank of percentile operation
    [percentiles: [<rank>,...]]
    # Relabel prometheus labels to skywalking dimensions.
    relabel:
      service: [<labelname>, ...]
      [instance: [<labelname>, ...]]
      [endpoint: [<labelname>, ...]]
```

#### <operation>

可用的操作有 `avg`, `avgHistogram` 和 `avgHistogramPercentile`。`avg` 和 `avgXXX` 表示将原始获得的度量指标或者高比率指标平均为低比率度量指标。该过程是 Skywalking 的向下采样的延伸，这就增加了从原始数据到分钟比率的过程。

当你指定 `avgHistogram` 和 `avgHistogramPercentile` 时，度量指标 source 必须是 `histogram` 类型。由于直方图的 `bucket`, `sum` 和 `count` 都是计数器，所以也需要一个统计函数。
