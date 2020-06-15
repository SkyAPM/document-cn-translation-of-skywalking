# TTL
在 SkyWalkin 中, 除了元数据外, 有两种类型的观测数据
1. 记录, 包括 trace 和告警. 未来可能还包括日志.
2. 指标, 包括如 p99/p95/p90/p75/p50, 热点图, 成功率, cpm(rpm) 等.
指标又根据 分钟/小时/天/月 维度, 在存储中划分为不同索引或表. 

针对不同的类型, 你有以下设置:
```yaml
    # 设置指标数据的超时. 超时到期后，指标数据将自动删除.
    recordDataTTL: ${SW_CORE_RECORD_DATA_TTL:3} # Unit is day
    metricsDataTTL: ${SW_CORE_RECORD_DATA_TTL:7} # Unit is day
```

- `recordDataTTL` 影响的**记录**数据，包括跟踪和报警.
- `metricsDataTTL` 影响的所有指标，包括服务、实例、端点指标和拓扑映射指标.
