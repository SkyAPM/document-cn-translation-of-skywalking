# 通过 ALS 观测服务网格

Envoy [ALS(access log service)](https://www.envoyproxy.io/docs/envoy/latest/api-v2/service/accesslog/v2/als.proto)
提供了 RPC 路由的完整日志, 包括 HTTP 和 TCP.

你需要以下三步来开启 ALS.

1. 到目前为止, Istio pilot 还不支持开启 ALS, 所以你需要修改 pilot 代码.
1. 开启 SkyWalking [envoy 接收器](../backend/backend-receivers.md).
1. 激活 ALS k8s-mesh 分析

```yaml
envoy-metric:
  default:
    alsHTTPAnalysis:
      - k8s-mesh
```

需要注意的是, 仅在 Envoy 受 Istio 控制的情况下才使用此功能.
否则, 你需要自己实现 `ALSHTTPAnalysis` 并且将其注册到接收器.
