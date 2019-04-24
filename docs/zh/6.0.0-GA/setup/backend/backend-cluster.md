# 集群管理
在大多数生产环境中，后端应用需要支持高吞吐量并且支持高可用来保证服务的稳定，所以你始终需要在生产环境进行集群管理。
 
SkyWalking的backend提供了几种集群管理的方案。选择你需要或想要的那个。

- [Zookeeper协调方式](#Zookeeper协调方式)。使用ZooKeeper让后端实例彼此之间检测和通信。
- [Kubernetes](#kubernetes)。 当backend集群部署在kubernetes时，你可以使用k8s原生的APIs来管理集群。
- [Consul](#consul)。 使用Consul作为集群管理的实现者，来协调backend的实例。


## Zookeeper协调方式
Zookeeper是一个很常见并的广泛使用的集群协调者。在yml文件中设置 **cluster** 实现者为**zookeeper**来启用它。 

Zookeeper版本需要在3.4以上。

```yaml
cluster:
  zookeeper:
    nameSpace: ${SW_NAMESPACE:""}
    hostPort: ${SW_CLUSTER_ZK_HOST_PORT:localhost:2181}
    # Retry Policy
    baseSleepTimeMs: 1000 # initial amount of time to wait between retries
    maxRetries: 3 # max number of times to retry
```

- `hostPort`是zookeeper服务器的列表。格式`IP1:PORT1,IP2:PORT2,...,IPn:PORTn`
- `hostPort`, `baseSleepTimeMs` and `maxRetries` 是Zookeeper客户端的设置。.

在某些情况下，OAP默认的GRPC主机和核心端口不适合OAP节点之间的内部通信。
根据您自己的LAN环境，可通过下列参数设置你的host和port。
- InternalComHost。已注册的主机和其它OAP节点使用它与当前节点通信。
- InternalCompart。已注册的端口和其它OAP节点使用它与当前节点通信。

```yaml
zookeeper:
  nameSpace: ${SW_NAMESPACE:""}
  hostPort: ${SW_CLUSTER_ZK_HOST_PORT:localhost:2181}
  #Retry Policy
  baseSleepTimeMs: ${SW_CLUSTER_ZK_SLEEP_TIME:1000} # initial amount of time to wait between retries
  maxRetries: ${SW_CLUSTER_ZK_MAX_RETRIES:3} # max number of times to retry
  internalComHost: 172.10.4.10
  internalComPort: 11800
``` 


## Kubernetes
需要backend集群部署在kubernetes中, 可参考 [Deploy in kubernetes](backend-k8s.md).
设置实现者为`kubernetes`.

```yaml
cluster:
  kubernetes:
    watchTimeoutSeconds: 60
    namespace: default
    labelSelector: app=collector,release=skywalking
    uidEnvName: SKYWALKING_COLLECTOR_UID
```

## Consul
目前，consul正成为一个流行的系统，很多公司和开发者使用它作为服务发现的手段。在yml文件中设置 **cluster** 实现者为**consul**来启用它。

```yaml
cluster:
  consul:
    serviceName: ${SW_SERVICE_NAME:"SkyWalking_OAP_Cluster"}
    # Consul cluster nodes, example: 10.0.0.1:8500,10.0.0.2:8500,10.0.0.3:8500
    hostPort: ${SW_CLUSTER_CONSUL_HOST_PORT:localhost:8500}
```

和Zookeeper协调方式一样
在某些情况下，OAP默认的GRPC主机和核心端口不适合OAP节点之间的内部通信。
根据您自己的LAN环境，可通过下列参数设置你的host和port。
- InternalComHost。已注册的主机和其它OAP节点使用它与当前节点通信。
- InternalCompart。已注册的端口和其它OAP节点使用它与当前节点通信。