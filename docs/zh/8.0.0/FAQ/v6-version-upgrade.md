# V6 升级

SkyWalking v6 广泛使用在生产环境。用户可能想从老版本升级为新版本。这个指南将告诉用户如何做。

**注意**，一下方法不是唯一的升级方式。

## 使用金丝雀发布

和所有的应用一样，SkyWalking 可以一下步骤使用 `金丝雀发布` 方式升级

1. 使用最新版本 SkyWalking OAP 集群和新的数据库集群部署一个新的集群
2. 一旦目标（被监控）服务有机会升级 agent.jar (或仅仅需要重启)，将 `collector.backend_service` 改为指向新的 OAP 后台，并且使用/添加一个新的命名空间（`agent.namespace`，在 [agent 配置属性表](../setup/service-agent/java-agent/README.md#table-of-agent-configuration-properties) 中配置）。命名空间将避免不同版本之间的冲突。
3. 当所有目标服务都重新启动后，可以废弃旧的 OAP 集群。

`金丝雀发布` 方式适用任何版本升级。

## 在线热重启升级

我们需要使用 `金丝雀发布` 的原因是：SkyWalking 代理具有缓存机制，切换到新的集群会使缓存对新的OAP集群不可用。

在 6.5.0+(特别是 agent 版本)，我们有 [**代理热重启触发机制**](../setup/backend/backend-setup.md#agent-hot-reboot-trigger-mechanism-in-oap-server-upgrade).

通过使用它，我们可以用一种更简单的方式进行升级，**使用最新版本 SkyWalking OAP 集群和新的数据库集群部署一个新的集群**，一次性的将流量迁移到新的集群。基于这个机制，所有的代理将会进入 `cool_down` 模式，然后重新上线。更多细节，请阅读后端安装文档。

**注意**, 6.4.0 有一个已知的 bug, 他的 agent 有重连问题, 所以, 即使在 6.4.0 中包含了这种重新启动机制，它也可能无法在某些网络场景中工作，特别是在 k8s 中。

## Agent 兼容性

所有的 SkyWalking 6.x(甚至 7.x)是相互兼容的，所以用户可以优先只升级 OAP 服务。这个代理还会再版本之间进行增强，所以根据 SkyWalking 团队的建议，一旦有机会，就升级 agent。
