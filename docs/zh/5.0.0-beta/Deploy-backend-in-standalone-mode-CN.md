# 用途说明
单机模式默认使用本地H2数据库，不支持集群部署。主要用于：预览、功能测试、演示和低压力系统。

如果使用单机collector用于非演示环境，你可选择使用Elasticsearch作为存储实现。

**在5.0.0-alpha版本中，暂不提供H2实现, 所以在启动之前,必须先部署ElasticSearch**

## 所需的第三方软件
- JDK8+

## 下载发布版本
- 前向[发布页面](https://github.com/apache/incubator-skywalking/releases)

## Quick Start
Collector单机模拟启动简单，提供和集群模式相同的功能，单机模式下除端口(8080, 10800, 11800, 12800)被占用的情况下，直接启动即可。

## 部署后台服务
1. 解压安装包`tar -xvf skywalking-collector.tar.gz`，windows用户可以选择zip包
1. 运行`bin/startup.sh`启动。windows用户为.bat文件。

- **注意：在5.0.0-alpha 版本中,startup.sh将会启动collector和UI两个进程，UI通过127.0.0.1:10800访问本地collector，无需额外配置。**

## 使用Elastic Search代替H2存储
- 在单机模式下，`collector`也支持其他的存储（当前已支持的ElasticSearch 5.3），取消  `application.yml`配置文件中的`storage` 相关配置节的注释，并修改配置,默认的配置是`collector`
和 `Elasticsearch` 运行在同一台机器上。

## 部署Elasticsearch
- 修改`elasticsearch.yml`文件
  - 设置 `cluster.name: CollectorDBCluster`。此名称需要和collector配置文件一致。
  - 设置 `node.name: anyname`, 可以设置为任意名字，如Elasticsearch为集群模式，则每个节点名称需要不同。
  - 增加如下配置

```
# ES监听的ip地址
network.host: 0.0.0.0
thread_pool.bulk.queue_size: 1000
```

- 启动Elasticsearch
