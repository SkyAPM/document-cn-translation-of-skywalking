# Backend的telemetry
默认情况下, telemetry功能是关闭的，像这样：
```yaml
telemetry:
  none:
```

## Prometheus
Prometheus可做为telemetry功能的实现者。
使用这个，prometheus可以从Skywalking的backend收集度量。


将`prometheus`设置为provider。端点在开放在`http://0.0.0.0:1234/`和`http://0.0.0.0:1234/metrics`。
```yaml
telemetry:
  prometheus:
```

根据需要设置主机和端口。
```yaml
telemetry:
  prometheus:
    host: 127.0.0.1
    port: 1543
```

### Grafana可视化
提供了两种grafana仪表盘的设置。
1. 当SkyWalking 和追踪agent一起使用时，参考[SkyWalking trace-mode dashboard](telemetry/trace-mode-grafana.json)。
1. 当SkyWalking 和service mesh telemetry（包括istio和envoy）一起使用时，参考[SkyWalking mesh-mode dashboard](telemetry/mesh-mode-grafana.json)。