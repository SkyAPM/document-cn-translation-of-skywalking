# Backend和UI

SkyWalking backend 发行包包括下面几部分
1. **bin/cmd scripts**，在`/bin`文件夹下。 包含linux和Windows环境下启动脚本， 来启动Backend的服务器和UI。
1. **Backend config**， 在 `/config` 文件夹下。 包含Backend的设置文件,`application.yml`、
`log4j.xml`和`alarm-settings.yml`。大多数开放的设置都在这些文件里面。
1. **Libraries of backend**， 在 `/oap-libs`文件夹下。所有Backend的jar包文件都在里面。
1. **Webapp env**， 在 `webapp` 文件夹下。 UI 前端的jar包文件和它的`webapp.yml` 设置文件在这。

## 快速入门
要求： **JDK8**

在看这个之前，希望您知道，快速入门只是是运行Skywalking的Backend和UI来进行预览或演示。在这儿，性能和长期运行不是我们的目标。

想部署到product或test环境？查看[部署Backend和UI](#部署Backend和UI)

你可以通过`bin/startup.sh`(或cmd) 在默认设置下启动Backend和UI,，同时希望你能了解：
- 默认使用H2存储，这样就不需要部署别的了。
- Backend的gRPC相关的API可访问`0.0.0.0/11800`，rest相关的API可访问`0.0.0.0/12800`。
在Java，.NetCore，Node.js， Istio agents/probe中，设置gRPC服务地址为`ip/host:11800`。
(ip/host填写Backend暴露的)
- UI 监听`8080` 端口,同时请求`127.0.0.1/12800`来做GraphQL查询。  

## 部署Backend和UI
快速入门后，您应该希望在分布式环境中部署后端和UI。
在那之前， 你需要知道agent/probe, Backend和UI之间是怎么交互的。

<img src="http://skywalking.apache.org/doc-graph/communication-net.png"/>

- 所有原生的agents和probes，不管基于何种语言或网格探针，都是使用gRPC服务(`core/default/gRPC*` in `application.yml`) 上报数据给Backend。另外，json格式支持Jetty服务。
- UI 使用GraphQL(HTTP)查询来访问Backend，在Jetty服务中也是(`core/default/rest*` in `application.yml`)。

现在，让我们继续来看进行Backend和UI的设置文档。
- [Backend setup document](backend-setup.md)
- [UI setup document](ui-setup.md)

此外，为了方便起见，我们提供了[k8s settings](https://github.com/apache/incubator-skywalking-kubernetes#deploy-skywalking-backend-to-kubernetes-cluster)。
