# Open Fetcher

Fetcher 是 SkyWalking 后台的一个概念。当从目标系统读数据时，使用拉模式相比 [receiver](backend-receivers.md) 更合适。这个模式典型的存在一些度量 SDK 中，例如 Prometheus。

## Prometheus Fetcher

```yaml
prometheus-fetcher:
  selector: ${SW_PROMETHEUS_FETCHER:default}
  default:
    active: ${SW_PROMETHEUS_FETCHER_ACTIVE:false}
```

TODO: 当 fetcher 提供时，将会添加更多信息。
