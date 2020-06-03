# 高级部署

OAP 服务器在集群环境中相互通信。在集群模式中，可通过不同的角色运行。

- Mixed(默认)
- Receiver
- Aggregator

有时，用户希望集群中的节点承担明确的角色。可以参考以下：

## 混合(Mixed)

默认角色，OAP 需承担以下责任：

1. 接收 agent 的跟踪或指标
2. 做一级(L1)聚合
3. 内部通信(发送/接收)
4. 做二级(L2)聚合
5. 持久化数据
6. 告警

## 接收(Receiver)

OAP 需承担以下责任：

1. 接收代理跟踪或指标
2. 做一级(L1)聚合
3. 内部通信(发送)

## 聚合(Aggregator)

OAP需承担以下责任：

1. 内部沟通(接收)
2. 做一级(L2)聚合
3. 持久化数据
4. 告警

___
这些角色是基于安全和网络策略的复杂部署需求而设计的。

## Kubernetes

如果你正在使用我们原生的[Kubernetes coordinator](backend-cluster.md#kubernetes)，`labelSelector` 的设置可用于 `Aggregator` 角色选择规则。根据你的需求选择合适的 OAP 部署方式。