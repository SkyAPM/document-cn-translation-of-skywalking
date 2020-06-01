# 后台遥测数据

默认情况下, telemetry 功能是关闭的(`selector` to `none`)，像这样：

```yaml
telemetry:
  selector: ${SW_TELEMETRY:none}
  none:
  prometheus:
    host: ${SW_TELEMETRY_PROMETHEUS_HOST:0.0.0.0}
    port: ${SW_TELEMETRY_PROMETHEUS_PORT:1234}
  so11y:
    prometheusExporterEnabled: ${SW_TELEMETRY_SO11Y_PROMETHEUS_ENABLED:true}
    prometheusExporterHost: ${SW_TELEMETRY_PROMETHEUS_HOST:0.0.0.0}
    prometheusExporterPort: ${SW_TELEMETRY_PROMETHEUS_PORT:1234}
```

但你可以设置 `prometheus` 或者 `so11y` 其中之一启用，更多参考如下。

## Prometheus

Prometheus 可做为 telemetry 功能的实现者。使用这个功能，prometheus 可以从 Skywalking 的 backend 收集度量数据。

将 `prometheus` 提供出来。端点在开放在 `http://0.0.0.0:1234/` 和 `http://0.0.0.0:1234/metrics`。

```yaml
telemetry:
  selector: ${SW_TELEMETRY:prometheus}
  prometheus:
```

根据需要设置主机和端口。

```yaml
telemetry:
  selector: ${SW_TELEMETRY:prometheus}
  prometheus:
    host: 127.0.0.1
    port: 1543
```

### Grafana 可视化

提供了 Grafana 面板的设置。参考 [SkyWalking 遥测数据面板](grafana.json)。

## 自身可观测性

SkyWalking 支持将遥测数据直接收集到 OAP 后台。用户可以通过 UI 或 GraphQL API 查看它们。

添加如下配置可以启用 `so11y`(自观测)关联模块。

```yaml
receiver-so11y:
  selector: ${SW_RECEIVER_SO11Y:default}
  default:
telemetry:
  selector: ${SW_TELEMETRY:so11y}
  # ... other configurations
```

另一个示例展示如何结合 `promethues` 和 `so11y`。添加一些项到 `so11y` 使其生效。

```yaml
telemetry:
  selector: ${SW_TELEMETRY:so11y}
  so11y:
    prometheusExporterEnabled: true
    prometheusExporterHost: 0.0.0.0
    prometheusExporterPort: 1234
```

prometheus exporter 监听在 `0.0.0.0:1234`。
