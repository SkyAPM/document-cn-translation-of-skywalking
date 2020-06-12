#服务端、UI和CLI设置

SkyWalking 后端发布包主要分以下几块内容;

1. **启动脚本**,  `/bin` 目录. 包含linux启动和Windows启动服务和UI的脚本.

2. **配置文件**,  `/config` 目录. 包含以下配置文件.
   * `application.yml`
   * `log4j.xml`
   * `alarm-settings.yml`

3. **引用Jar**, `/oap-libs` 目录. 包含服务端所有依赖的jar包.

4. **Webapp相关**,  `webapp` 目录.  包含UI界面的jar和 `webapp.yml`的配置文件. 

## 快速入门

### 默认环境

需要: **JDK8到JDK12已测试**，其他版本未测试.

在看这个之前，希望你知道，快速入门只是是运行Skywalking的Backend和UI来进行预览或演示。在这儿，性能和长期运行不是本章节内容。

想部署到product或test等环境查看[部署服务端和界面UI](#部署Backend和UI)

你可以通过`bin/startup.sh`(或cmd) 在默认设置下启动Backend和UI,，同时希望你能了解：

- 默认使用H2存储，这样就不需要部署别的了。
- Backend的gRPC相关的API可访问`0.0.0.0/11800`，rest相关的API可访问`0.0.0.0/12800`。

在Java，.NetCore，Node.js， Istio agents/probe中，设置gRPC服务地址为`ip/host:11800`。
(ip/host填写Backend暴露的)
- UI 监听`8080` 端口,同时请求`127.0.0.1/12800`来做GraphQL查询。  

## 部署服务端和界面UI

快速入门后，您应该希望在分布式环境中部署后端和UI。
在那之前， 你需要知道agent/probe, Backend和UI之间是怎么交互的。

<img src="http://skywalking.apache.org/doc-graph/communication-net.png"/>

- 所有原生的agents和probes，不管基于何种语言或网格探针，都是使用gRPC服务(`core/default/gRPC*` in `application.yml`) 上报数据给服务端。另外，json格式支持Jetty服务。
- UI 使用GraphQL(HTTP)查询来访问Backend，在Jetty服务中也是(`core/default/rest*` in `application.yml`)。

现在，让我们继续来看进行Backend和UI的设置文档。
- [服务端启动](backend-setup.md)
- [界面UI启动](ui-setup.md)
- [CLI设置](https://github.com/apache/skywalking-cli)
