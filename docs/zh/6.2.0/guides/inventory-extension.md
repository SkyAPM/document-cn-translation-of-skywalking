# 后端存储实体扩展

SkyWalking 包含四种存储实体.

- 服务存储(Service Inventory)
- 服务实例存储(Service Instance Inventory)
- 端点存储(Endpoint Inventory)
- 网络地址存储(Network Address Inventory)

所有指标, 拓扑结构, 追踪和报警都跟这些实体 ID 相关联.

为了了解 **服务(Service)**, **服务实例(Service Instance)** 和 **端点(Endpoint)** 相关概念,
请阅读[项目介绍](../concepts-and-designs/overview.md#why-use-skywalking).

对于 **网络地址存储** 来说, 它代表了所有网络地址, 格式诸如: ip:端口, 主机名, 域名, 这些都可以通过特定编程语言 agent 和探针自动探测.

## 扩展

目前后端内核只支持了 **服务存储** 扩展.
服务以 JSON 的格式提供了字段 `properties`, 通常被用于特定的服务(如数据库, 缓存, 消息队列等), 而很少用于业务服务.

为了保持代码一致以及对查询和可视化友好, 该 JSON 属性需要遵循指定规则.

### 数据库

1. 节点类型 == **数据库**. NodeType == **Database(1)**
1. JSON 属性包含以下 Key.
  - `database`. 数据库名称, 如 MySQL, PostgreSQL.
  - `db.type`. 数据库类型, 如 sql db, redis db.
  - `db.instance`. 数据库实例名称.


