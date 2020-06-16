# 告警
告警的核心由一组规则驱动，这些规则定义在`config/ Alarm -settings.yml`文件中。
告警规则的定义分为三部分。
1. 告警规则。它们定义了应该如何触发度量警报，应该考虑什么条件。
2. [网络钩子](#Webhook}。当警告触发时，哪些服务终端需要被告知。
3. [gRPC钩子](#gRPCHook). 远程gRPC方法的主机和端口，告警触发后调用.

## 实体名称
定义范围和实体名称之间的关系.
- **服务**: 服务名称
- **实例**: {服务名称} 的 {实例名称}
- **端点**: {服务名称} 中的 {端点名称}
- **数据库**: 数据库服务名
- **服务关系**: {源服务名称} 到 {目标服务名称}
- **实例关系**: {源服务名称} 的 {源实例名称} 到 {目标服务名称} 的 {目标实例名称}
- **端点关系**: {源服务名称} 中的 {源端点名称} 到 {目标服务名称} 中的 {目标端点名称}

## 规则
报警规则主要有以下几点：
- **规则名称**。在告警信息中显示的唯一名称。必须以`_rule`结尾。
- **度量名称**。 也是oal脚本中的度量名。只支持long,double和int类型。详情见 [所有可能的度量名称列表](#所有可能的度量名称列表).
- **包含 名称**。其下的实体名称都在此规则中. 请遵循[实体名称定义](#实体名称)。
- **排除 名称**. 以下实体名称不在此规则中. 请遵循[实体名称定义](#实体名称).
- **包含 名称正则表达式**.提供一个正则表达式来包含实体名称. 如果同时设置包含名称的列表和包含名称的正则表达式，则两个规则都将生效.
- **排除 名称正则表达式**. 提供一个正则表达式来排除需要排除的名称. 如果同时设置排除的名称列表和排除名称的正则表达式，则两个规则都将生效.
- **阈值**。阈值。
对于多个值指标，例如**percentile**，阈值是一个数组。像`value1` `value2` `value3` `value4` `value5`这样描述.
每个值可以作为度量中每个值的阈值。如果不想通过此值或某些值触发警报，则将值设置为 `-`.  
例如在**百分比**中，`value1`是P50的阈值,且 `-，-，value3, value4, value5`的意思是，没有阈值的P50和P75百分位告警规则.
- **操作符**。 操作符, 支持 `>`, `<`, `=`。欢迎贡献所有的操作符。
- **周期**.。多久告警规则需要被核实一下。这是一个时间窗口，与后端部署环境时间相匹配。                     
- **计数**。 在一个周期窗口中，如果**值**秒超过阈值（按周期统计），达到计数值，需要发送警报。
- **静默时间**。在时间N中触发报警后，在**TN -> TN + period**这个阶段不告警。 默认情况下，它和**周期**一样，这意味着相同的告警（同一个度量名称拥有相同的Id）在同一个周期内只会触发一次。


```yaml
rules:
  # 规则唯一名称，必须以'_rule'结尾.
  endpoint_percent_rule:
    # 度量值为 long、double或int 类型
    metrics-name: endpoint_percent
    threshold: 75
    op: <
    # 评估度量标准的时间长度
    period: 10
    # 度量有多少次符合告警条件后，才会触发告警
    count: 3
    #检查多少次，告警触发后保持沉默，默认周期相同
    silence-period: 10
    
  service_percent_rule:
    metrics-name: service_percent
    # [可选]默认，匹配此指标中的所有服务
    include-names:
      - service_a
      - service_b
    exclude-names:
      - service_c
    threshold: 85
    op: <
    period: 10
    count: 4
  service_resp_time_percentile_rule:
    # 度量值为 long、double或int 类型
    metrics-name: service_percentile
    op: ">"
    # 多种指标值的阈值。P50、P75、P90、P95、P99的阈值
    threshold: 1000,1000,1000,1000,1000
    period: 10
    count: 3
    silence-period: 5
    message: Percentile response time of service {name} alarm in 3 minutes of last 10 minutes, due to more than one condition of p50 > 1000, p75 > 1000, p90 > 1000, p95 > 1000, p99 > 1000
```

## 默认告警规则
为了方便，我们在发行版中提供了默认的`alarm setting.yml`文件，包括以下规则
1.最近 3 分钟内服务平均响应时间超过 1 秒。
2.服务成功率在最近 2 分钟内低于80%。
3.服务响应时间在最近 3 分钟内低于 1000 毫秒.
4.服务实例在最近 2 分钟内的平均响应时间超过 1 秒。
5.端点平均响应时间在最近 2 分钟内超过1秒。
6.数据库访问平均响应时间在过去 2 分钟内超过 1 秒。
7.端点之间平均响应时间在最近 2 分钟内超过 1 秒。


## 所有可能的度量名称列表
这些度量名称定义在 [OAL 脚本](../../guides/backend-oal-scripts.md)中, 现在，来自**Service**, **Service Instance**, **Endpoint**的度量可以用于告警，我们会在后期版本中进行扩展。

如果你希望有其它的告警场景，请提交issue或pr。


## Webhook
SkyWalking 的告警 Webhook 要求对等方是一个 Web 容器. 告警的消息会通过 HTTP 请求进行发送, 请求方法为 `POST`, `Content-Type` 为 `application/json`,
JSON 格式基于 `List<org.apache.skywalking.oap.server.core.alarm.AlarmMessage`, 包含以下信息.
- **scopeId**. 所有可用的 Scope 请查阅 `org.apache.skywalking.oap.server.core.source.DefaultScopeDefine`.
- **name**. 目标 Scope 的实体名称.
- **id0**. Scope 实体的 ID.
- **id1**. 未使用.
- **ruleName**. 您在 `alarm-settings.yml` 中配置的规则名.
- **alarmMessage**. 报警消息内容.
- **startTime**. 告警时间, 位于当前时间与 UTC 1970/1/1 之间.

以下是一个样例
```json
[{
	"scopeId": 1, 
        "name": "serviceA", 
	"id0": 12,  
	"id1": 0,  
    "ruleName": "service_resp_time_rule",
	"alarmMessage": "alarmMessage xxxx",
	"startTime": 1560524171000
}, {
	"scopeId": 1,
        "name": "serviceB",
	"id0": 23,
	"id1": 0,
    "ruleName": "service_resp_time_rule",
	"alarmMessage": "alarmMessage yyy",
	"startTime": 1560524171000
}]
```

## gRPCHook
告警消息将使用 `Protobuf` 类型通过gRPC远程方法发送. 
消息格式，以下文件定义了关键信息 `oap-server/server-alarm-plugin/src/main/proto/alarm-hook.proto`.

协议的部分内容如下:
```protobuf
message AlarmMessage {
    int64 scopeId = 1;
    string scope = 2;
    string name = 3;
    string id0 = 4;
    string id1 = 5;
    string ruleName = 6;
    string alarmMessage = 7;
    int64 startTime = 8;
}
```

## 动态更新配置
从6.5.0开始，可以通过[动态配置](dynamic-config.md)在运行时动态更新报警设置 ,
它将覆盖 `alarm-settings.yml` 中的设置.

为了确定是否触发告警规则, SkyWalking 需要为每个告警规则缓存时间窗的指标, 如果任何属性 (`metrics-name`, `op`, `threshold`, `period`, `count`, etc.) 的规则改变,
滑动窗口将会毁坏和重建, 导致此告警规则重新启动.