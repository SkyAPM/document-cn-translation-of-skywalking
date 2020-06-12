# TTL Time To Live

在 SkyWalkin 中, 除了元数据外, 有两种类型的观测数据
1. 记录, 包括 trace 和告警. 未来可能还包括日志.
1. 指标, 包括如 p99/p95/p90/p75/p50, 热点图, 成功率, cpm(rpm) 等.
指标又根据 分钟/小时/天/月 维度, 在存储中划分为不同索引或表. 

针对不同的类型, 你有以下设置:
```yaml
    # 设置指标数据的超时. 超时到期后，指标数据将自动删除.
    enableDataKeeperExecutor: ${SW_CORE_ENABLE_DATA_KEEPER_EXECUTOR:true} # 关闭后，指标数据自动删除也将随之关闭.
    recordDataTTL: ${SW_CORE_RECORD_DATA_TTL:90} # 单位为分钟
    minuteMetricsDataTTL: ${SW_CORE_MINUTE_METRIC_DATA_TTL:90} # 单位为分钟
    hourMetricsDataTTL: ${SW_CORE_HOUR_METRIC_DATA_TTL:36} # 单位为小时
    dayMetricsDataTTL: ${SW_CORE_DAY_METRIC_DATA_TTL:45} # 单位为天
    monthMetricsDataTTL: ${SW_CORE_MONTH_METRIC_DATA_TTL:18} # 单位为月
```

- `recordDataTTL` 影响 **记录** 数据.
- `minuteMetricsDataTTL`, `hourMetricsDataTTL`, `dayMetricsDataTTL` 和 `monthMetricsDataTTL` 影响
分钟/小时/天/月维度的指标数据.

## ElasticSearch 6 存储 TTL 
**特别注意:**  
由于 ElasticSearch 的特性, 它会在每次通过查询条件进行删除之后重建索引.
这是一个非常重的操作, 会导致 ElasticSearch 服务挂起数秒钟.
现在 ElasticSearch 中有上百个索引, 在上述情况下, 会导致 ElasticSearch 意外地停止响应.
因此我们按天创建索引来避免执行按查询条件删除数据,
然后直接删除整个过期的索引, 这是个高性能的操作, 向挂起说再见.

在 ElasticSearch 存储中, 有以下的配置.
```yaml
    # 这些TTL配置将覆盖核心模块中的相同配置。
    recordDataTTL: ${SW_STORAGE_ES_RECORD_DATA_TTL:7} # 单位为天
    otherMetricsDataTTL: ${SW_STORAGE_ES_OTHER_METRIC_DATA_TTL:45} # 单位为天
    monthMetricsDataTTL: ${SW_STORAGE_ES_MONTH_METRIC_DATA_TTL:18} # 单位为月
``` 

- `recordDataTTL` 影响 **记录** 数据.
- `otherMetricsDataTTL` 影响 分钟/小时/天 维度的指标. `minuteMetricsDataTTL`, `hourMetricsDataTTL` 和 `dayMetricsDataTTL` 还保留, 但他们的单位改为了 **天**. 如果你想手动设置他们, 请删除掉 `otherMetricsDataTTL`.
- `monthMetricsDataTTL` 影响 月 维度的指标.
