# 初始化模式
Skywalking后端支持多个存储实施器。它们中的大多数，比如Elastic Search, Database可以在启动时自动初始化存储。

但是根据存储情况，会有一些意外的情况发生，比如
`由于多个后端实例同时启动，会同时创建Elastic Search的索引。`
当出现一个Change，Elastic Search的API会被阻塞并且不报异常。
在类似K8S这样的容器管理平台上，发生的几率更大。

这就是您需要**初始化模式**启动的地方。

## 解决方法
在其它实例启动前，只有一个实例可以在**初始化模式**运行。
完成所有初始化步骤后，该实例将优雅地退出。

使用`oapServiceInit.sh`/`oapServiceInit.bat`来启动backend。你可以看到下面的log
> 2018-11-09 23:04:39,465 - org.apache.skywalking.oap.server.starter.OAPServerStartUp -2214 [main] INFO  [] - OAP starts up in init mode successfully, exit now...

## Kubernetes
这种模式下的初始化将包含在我们的kubernetes脚本和helm中。