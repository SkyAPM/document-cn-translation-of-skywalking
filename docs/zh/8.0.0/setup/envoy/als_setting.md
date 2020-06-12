# 通过 ALS 观测服务网格

Envoy [ALS(access log service)](https://www.envoyproxy.io/docs/envoy/latest/api-v2/service/accesslog/v2/als.proto) 提供了 RPC 路由的完整日志, 包括 HTTP 和 TCP.

方案由 [Sheng Wu](https://github.com/wu-sheng), [Hongtao Gao](https://github.com/hanahmily), [Lizan Zhou](https://github.com/lizan) 和 [Dhi Aurrahman](https://github.com/dio) 在 2019/5/17 启动实现，并在 [KubeCon China 2019](https://kccncosschn19eng.sched.com/event/NroB/observability-in-service-mesh-powered-by-envoy-and-apache-skywalking-sheng-wu-lizan-zhou-tetrate) 提出。这里有 [视频](https://www.youtube.com/watch?v=tERm39ju9ew) 记录。

SkyWalking 是全球范围内第一个介绍这种基于 ALS 的解决方案的开源项目。这为网格服务提供了一种低负载但有相同可观测性的新方法。

你需要以下三步来开启 ALS:

1. 在 Istio 中打开 Envoy 访问日志服务，详见 [ProxyConfig 中启用 **envoyAccessLogService**](https://istio.io/docs/reference/config/istio.mesh.v1alpha1/#ProxyConfig)
2. 打开 SkyWalking [Envoy 接收器](../backend/backend-receivers.md).
3. 激活 ALS k8s-mesh 分析，配置如下

```yaml
envoy-metric:
  default:
    alsHTTPAnalysis: "k8s-mesh"
```

多个值使用 `,` 分隔

提示，只有当 Envoy 受 Istio 控制时才可以使用，并且在 K8S 环境下。 OAP 需要所有 pods 都有读 K8S API 服务的权限。
