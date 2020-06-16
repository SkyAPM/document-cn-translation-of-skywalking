# 后台遥测数据

默认情况下, telemetry 功能是关闭的(`selector` to `none`)，像这样：

```yaml
telemetry:
  selector: ${SW_TELEMETRY:none}
  none:
  prometheus:
    host: ${SW_TELEMETRY_PROMETHEUS_HOST:0.0.0.0}
    port: ${SW_TELEMETRY_PROMETHEUS_PORT:1234}
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

添加如下配置可以启用自观测关联模块。

### 1. 设置 prometheus 遥测数据

```yaml
receiver-so11y:
  selector: ${SW_TELEMETRY:prometheus}
  prometheus:
    host: 127.0.0.1
    port: 1543
```

### 2. 设置 prometheus fetcher

```yaml
prometheus-fetcher:
  selector: ${SW_PROMETHEUS_FETCHER:default}
  default:
    active: ${SW_PROMETHEUS_FETCHER_ACTIVE:true}
```

### 3. 确保 `config/fetcher-prom-rules/self.yaml` 存在

一旦你部署了 oap-server 集群，目标主机应该替换为专用 IP 或者主机名。对于实例，在你的集群中有三个 oap server，它们的主机分别是 `service1`, `service2` 和 `service3` 。你可以更新每个 `self.yaml` 配置目标主机

service1:

```yaml
fetcherInterval: PT15S
fetcherTimeout: PT10S
metricsPath: /metrics
staticConfig:
  # targets will be labeled as "instance"
  targets:
    - service1:1234
  labels:
    service: oap-server
...
```

service2:

```yaml
fetcherInterval: PT15S
fetcherTimeout: PT10S
metricsPath: /metrics
staticConfig:
  # targets will be labeled as "instance"
  targets:
    - service2:1234
  labels:
    service: oap-server
...
```

service3:

```yaml
fetcherInterval: PT15S
fetcherTimeout: PT10S
metricsPath: /metrics
staticConfig:
  # targets will be labeled as "instance"
  targets:
    - service3:1234
  labels:
    service: oap-server
...
```
