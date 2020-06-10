# 问题

控制台中太多 GRPC 日志

## 原因

Skywalking 采用了 GRPC 框架发送数据，GRPC 框架读取日志配置文件进行日志输出。

## 解决方法

在日志配置文件中添加对 `org.apache.skywalking.apm.dependencies` 包的过滤
