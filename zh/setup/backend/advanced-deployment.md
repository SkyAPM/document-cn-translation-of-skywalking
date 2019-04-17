# 高级部署
OAP服务器在集群环境中相互通信。
在集群模式中，可通过不同的角色运行。
- Mixed(默认)
- Receiver
- Aggregator

有时，用户希望集群中的节点承担明确的角色。可以参考以下：
## Mixed
默认角色，OAP需承担以下责任：
1. 接收agent的跟踪或指标。
1. 做L1聚合
1. 内部通信(发送/接收)
1. 做L2聚合
1. 持久化数据
1. 报警
## Receiver
OAP需承担以下责任：
1. 接收代理跟踪或指标
1. 做L1聚合
1. 内部通信(发送)
## Aggregator
OAP需承担以下责任：
1. 内部沟通(接收)
1. 做L2聚合
1. 持久化数据
1. 告警

___
这些角色是基于安全和网络策略的复杂部署需求而设计的。

## Kubernetes
如果你正在使用我们原生的[Kubernetes coordinator](backend-cluster.md# Kubernetes)，`labelSelector`
的设置可用于`Aggregator`角色。根据你的需求选择正确的OAP部署。
