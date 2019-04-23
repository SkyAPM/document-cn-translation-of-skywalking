# 启动模式
在不同的部署工具（如K8S）中，可能需要不同的启动模式。
我们还提供另外两种可选的启动模式。

## Default mode
Default mode。 如果需要，进行初始化工作，启动listen并提供服务。

运行 `/bin/oapService.sh`(.bat) 来启动这个模式。也可以在使用 `startup.sh`(.bat)来启动。

## Init mode
在此模式下，OAP服务器启动以执行初始化工作，然后退出。
您可以使用此模式初始化存储，例如ElasticSearch索引、MySQL和TIDB表，和init数据。

运行 `/bin/oapServiceInit.sh`(.bat) 来启动这个模式。

## No-init mode
在此模式下，OAP服务器不进行初始化。
但它等待存在弹性搜索索引、mysql和tidb表，开始倾听并提供服务。意味着此OAP服务希望别的OAP服务器进行初始化。

运行 `/bin/oapServiceNoInit.sh`(.bat) 来启动这个模式。