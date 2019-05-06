# 配置 Envoy 来向 SkyWalking 发送度量指标

为了让 Envoy 发送度量指标到 SkyWalking，我们需要给予 Envoy 一些包含 `stats_sinks` 的配置，其中包括 `envoy.metrics_service`。
`envoy.metrics_service` 此配置应该作为 [`config.grpc_service`](https://www.envoyproxy.io/docs/envoy/latest/api-v2/api/v2/core/grpc_service.proto#envoy-api-msg-core-grpcservice) 的一个配置项进行配置。

以下配置展示了其中比较有用的某些配置项：

```yaml
stats_sinks:
  - name: envoy.metrics_service
    config:
      grpc_service:
        # 注意: 我们也可以使用 google_grpc 实现
        envoy_grpc:
          cluster_name: service_skywalking

static_resources:
  ...
  clusters:
  - name: service_skywalking
    connect_timeout: 5s
    type: LOGICAL_DNS
    http2_protocol_options: {}
    dns_lookup_family: V4_ONLY
    lb_policy: ROUND_ROBIN
    load_assignment:
      cluster_name: service_skywalking
      endpoints:
      - lb_endpoints:
        - endpoint:
            address:
              socket_address:
                address: skywalking
                # 这是 SkyWalking 为 Envoy 度量指标的 gRPC 服务流提供服务的端口
                port_value: 11800
```

更完整的静态配置可以在[此处](config.yaml)找到。

需要注意的是 Envoy 还可以进行通过 [xDS Protocol](https://github.com/envoyproxy/data-plane-api/blob/master/XDS_PROTOCOL.md) 进行动态配置。

# 度量指标数据

Envoy 的一些统计数据列在[这个列表](https://www.envoyproxy.io/docs/envoy/latest/configuration/statistics)中，
有一个示例数据包含了 identifier ，可以在[此处](identify.json)找到，只包含指标的示例数据可以在[此处](metrics.json)找到.
