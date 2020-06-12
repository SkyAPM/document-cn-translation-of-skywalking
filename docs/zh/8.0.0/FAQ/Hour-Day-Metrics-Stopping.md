# 为什么小时和日指标在升级到7.x后停止更新?

这是6.x->7.x升级时遇到的情况
阅读 ElasticSearch存储的[采样数据封装特性](../setup/backend/backend-storage.md#downsampling-data-packing)

用户可以简单地删除所有过期的' *-day_xxxxx '和' *-hour_xxxxx ' (' xxxxx '是时间戳)索引。
skywalk只使用“metrics name-xxxxx”和“metrics name-month_xxxxx”索引.