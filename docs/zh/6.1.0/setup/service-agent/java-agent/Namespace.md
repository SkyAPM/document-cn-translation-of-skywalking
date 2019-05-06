# 命名空间

## 背景 
SkyWalking是一个监控工具，会从分布式系统中收集度量值。在真实世界中，大型分布式系统包含了成百上千的服务和服务实例。在这种情况下，很有可能不止一个小组、甚至不止一个公司在维护和监控分布式系统。每个小组或公司掌管着不同的部分，它们不希望也不应该共享它们的度量值。 

命名空间正是基于以上背景提出的，用于追踪监控系统的隔离。

## 设置命名空间
### 在agent配置文件中设置agent.namespace
```properties
# The agent namespace
# agent.namespace=default-namespace
``` 

`agent.namespace`的默认值为空。 

**影响**
SkyWalking的默认header为`sw3`。更多细节可查看[文档](../../../protocols/Skywalking-Cross-Process-Propagation-Headers-Protocol-v1.md).
设置`agent.namespace`后，header变为`namespace-sw3`。

当两边使用了不同的命名空间时，跨进程传播链将断掉。
