# 升级的常见问题解答
## 版本 3.x -> 5.0.0-alpha
### Collector
### 现象
UI上没数据。

### 原因
从3.2.6升级到5.0.0，Elasticsearch的索引由于已经存在所以没有重新创建，但是它们和5.0.0-alpha不兼容。
注册服务名时，ES将按默认类型字符串创建此列，这是错误的。

### 解决方法
清理elasticsearch中的数据文件夹，重新启动elasticsearch、collector和监控应用程序。