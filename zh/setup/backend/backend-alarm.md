# 告警
告警的核心由一组规则驱动，这些规则定义在`config/ Alarm -settings.yml`文件中。
告警规则的定义分为两部分。
1. 告警规则。它们定义了应该如何触发度量警报，应该考虑什么条件。
2. 网络钩子。当警告触发时，哪些服务终端需要被告知。

## 规则
报警规则主要有以下几点：
- **Rule name**。在告警信息中显示的唯一名称。必须以`_rule`结尾。
- **Indicator name**。 也是oal脚本中的度量名。只支持long,double和int类型。详情见
[所有可能的度量名称列表](#所有可能的度量名称列表).
- **Include names**。其下的实体名称都在此规则中。比如服务名，终端名。
- **Threshold**。阈值。
- **OP**。 操作符, 支持 `>`, `<`, `=`。欢迎贡献所有的操作符。
- **Period**.。多久告警规则需要被核实一下。这是一个时间窗口，与后端部署环境时间相匹配。                     
- **Count**。 在一个Period窗口中，如果**value**s超过Threshold值（按op），达到Count值，需要发送警报。
- **Silence period**。在时间N中触发报警后，在**TN -> TN + period**这个阶段不告警。 默认情况下，它和**Period**一样，这意味着相同的告警（在同一个Indicator name拥有相同的Id）在同一个Period内只会触发一次。


```yaml
rules:
  # Rule unique name, must be ended with `_rule`.
  endpoint_percent_rule:
    # Indicator value need to be long, double or int
    indicator-name: endpoint_percent
    threshold: 75
    op: <
    # The length of time to evaluate the metric
    period: 10
    # How many times after the metric match the condition, will trigger alarm
    count: 3
    # How many times of checks, the alarm keeps silence after alarm triggered, default as same as period.
    silence-period: 10
    
  service_percent_rule:
    indicator-name: service_percent
    # [Optional] Default, match all services in this indicator
    include-names:
      - service_a
      - service_b
    threshold: 85
    op: <
    period: 10
    count: 4
```

## 默认告警规则
为了方便，我们在发行版中提供了默认的`alarm setting.yml`文件，包括以下规则
1.过去3分钟内服务平均响应时间超过1秒。
1.服务成功率在过去2分钟内低于80%。
1.服务90%响应时间在过去3分钟内低于1000毫秒.
1.服务实例在过去2分钟内的平均响应时间超过1秒。
1.端点平均响应时间过去2分钟超过1秒。
 


## 所有可能的度量名称列表
这些度量名称定义在 [OAL 脚本](../../guides/backend-oal-scripts.md)中, 现在，来自**Service**, **Service Instance**, **Endpoint**的度量可以用于告警，我们会在后期版本中进行扩展。

如果你希望有其它的告警场景，请提交issue或pr。

