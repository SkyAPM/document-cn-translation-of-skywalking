# 版本 3.x -> 5.0.0-alpha 升级的常见问题解答

## Collector

### 现象

UI 上没数据。

### 原因

从 3.2.6 升级到 5.0.0，Elasticsearch 的索引由于已经存在所以没有重新创建，但是它们和 5.0.0-alpha 不兼容。注册服务名时，ES 将按默认类型字符串创建此列，这是错误的。

### 解决方法

清理 elasticsearch 中的数据文件夹，重新启动 elasticsearch、collector 和监控应用程序。
