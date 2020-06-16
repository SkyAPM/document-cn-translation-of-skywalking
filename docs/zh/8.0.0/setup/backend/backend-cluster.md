# 集群管理

在大多数生产环境中，后端应用需要支持高吞吐量并且支持高可用来保证服务的稳定，所以你始终需要在生产环境进行集群管理。

SkyWalking 的后台提供了几种集群管理的方案。选择你需要或想要的那个。

- [Zookeeper 协调方式](#zookeeper-coordinator)。使用 ZooKeeper 让后端实例彼此之间检测和通信。
- [Kubernetes](#kubernetes). 当后台集群部署在 kubernetes 中，你可以选择使用 k8s 原生的 APIs 来管理集群。
- [Consul](#consul). 使用 Consul 作为集群管理的实现者，来协调的实例。
- [Etcd](#etcd). 使用 Etcd 作为集群管理的实现者, 来协调后端的实例。

在 `application.yml` 中, 在 `cluster` 部分有上述协调器的默认配置，你可以通过 `selector` 属性启用他们中的任意一个。

## Zookeeper coordinator

Zookeeper 是一个很常见并的广泛使用的集群协调者。在 yml 文件中设置 **cluster/selector** 为 **zookeeper** 来启用它。

Zookeeper 需要在 3.4 以上版本。

```yaml
cluster:
  selector: ${SW_CLUSTER:zookeeper}
  # other configurations
```

- `hostPort` 是 zookeeper 一组服务列表。 格式： `IP1:PORT1,IP2:PORT2,...,IPn:PORTn`
- `enableACL` 启用 [Zookeeper ACL](https://zookeeper.apache.org/doc/r3.4.1/zookeeperProgrammers.html#sc_ZooKeeperAccessControl) 来控制访问 zk 节点.
- `schema` Zookeeper ACL schemas.
- `expression` ACL 的表达式. 表达式的格式是针对于 [schema](https://zookeeper.apache.org/doc/r3.4.1/zookeeperProgrammers.html#sc_BuiltinACLSchemes).
- `hostPort`, `baseSleepTimeMs` 和 `maxRetries` 是Zookeeper客户端的设置。

提醒:

- 如果 `Zookeeper ACL` 是启用的并且 `/skywalking` 节点存在，必须确保 `SkyWalking` 拥有 `CREATE`, `READ` 和 `WRITE` 的权限。如果 `/skywalking` 节点不存在，将会自动创建并针对指定用户授权所有权限。同时节点也会授权任意用户读权限。
- 如果设置 `schema` 为 `digest` ，密码的表达式设置在 **明文** 中。

在某些情况下，OAP 默认的 gRPC 主机和端口在核心中不适合 OAP 节点之间的内部通信。根据您自己的 LAN 环境，可通过下列参数设置你的 host 和 port。

- internalComHost, 已注册的主机和其它 OAP 节点使用它与当前节点通信。
- internalComPort, 已注册的端口和其它 OAP 节点使用它与当前节点通信。

```yaml
zookeeper:
  nameSpace: ${SW_NAMESPACE:""}
  hostPort: ${SW_CLUSTER_ZK_HOST_PORT:localhost:2181}
  #Retry Policy
  baseSleepTimeMs: ${SW_CLUSTER_ZK_SLEEP_TIME:1000} # initial amount of time to wait between retries
  maxRetries: ${SW_CLUSTER_ZK_MAX_RETRIES:3} # max number of times to retry
  internalComHost: 172.10.4.10
  internalComPort: 11800
  # Enable ACL
  enableACL: ${SW_ZK_ENABLE_ACL:false} # disable ACL in default
  schema: ${SW_ZK_SCHEMA:digest} # only support digest schema
  expression: ${SW_ZK_EXPRESSION:skywalking:skywalking}
```

## Kubernetes

需要后端集群部署在 kubernetes 中，参考 [kubernetes 中部署](backend-k8s.md)。设置 selector 为 `kubernetes`。

```yaml
cluster:
  selector: ${SW_CLUSTER:kubernetes}
  # other configurations
```

## Consul

目前，consul 正成为一个著名的系统，很多公司和开发者使用它作为服务发现的方案。在 yml 文件中设置 **cluster/selector** 为 **consul** 启用它。

```yaml
cluster:
  selector: ${SW_CLUSTER:consul}
  # other configurations
```

与 Zookeeper 协调器一样，在某些情况下，OAP 默认的 gRPC 主机和端口在核心中不适合 OAP 节点之间的内部通信。根据您自己的 LAN 环境，可通过下列参数设置你的 host 和 port。

- internalComHost, 已注册的主机和其它 OAP 节点使用它与当前节点通信。
- internalComPort, 已注册的端口和其它 OAP 节点使用它与当前节点通信。

## Etcd

在 yml 文件中设置 **cluster/selector** 为 **etcd** 启用它。

```yaml
cluster:
  selector: ${SW_CLUSTER:etcd}
  # other configurations
```
