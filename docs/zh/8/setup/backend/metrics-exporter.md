# Metrics Exporter

Skywalking 提供基本的，并且最为重要的度量聚合、告警和分析。

在实际使用时，人们可能希望将数据转发到第三方系统，以便进行更深入的分析或其它任何操作。

**Metrics Exporter** 使之成为可能。

Metrics exporter 是一个独立的模块，您需要手动激活它。

目前，我们提供下面这些 exporter

1. gRPC exporter

## gRPC exporter

gRPC exporter 使用 SkyWalking 原生的 exporter 服务定义。下面是 proto 的定义。

```proto
service MetricExportService {
    rpc export (stream ExportMetricValue) returns (ExportResponse) {
    }

    rpc subscription (SubscriptionReq) returns (SubscriptionsResp) {
    }
}

message ExportMetricValue {
    string metricName = 1;
    string entityName = 2;
    string entityId = 3;
    ValueType type = 4;
    int64 timeBucket = 5;
    int64 longValue = 6;
    double doubleValue = 7;
    repeated int64 longValues = 8;
}

message SubscriptionsResp {
    repeated string metricNames = 1;
}

enum ValueType {
    LONG = 0;
    DOUBLE = 1;
    MULTI_LONG = 2;
}

message SubscriptionReq {

}

message ExportResponse {
}
```

要激活这个 exporter，你需要把下面的配置加到 `application.yml` 文件中。

```yaml
exporter:
  grpc:
    targetHost: 127.0.0.1
    targetPort: 9870
```

- `targetHost`:`targetPort`是期望的目标服务器地址。你可以设置任意 gRPC 服务来接收数据。
- 目标 gRPC 服务准备好，否则 OAP 会启动失败。

## 针对目标 exporter 的服务

### 订阅实现

返回预期的度量名称列表，所有名称都必须与 OAL 脚本定义匹配。如果要导出所有度量值，则返回空列表。

### 导出实现

流服务，所有订阅的度量将根据 OAP 核心调度发送到这里。另外，如果 OAP 部署为集群，此方法将被并发地调用。对于度量值，您需要根据 `#type` 选择 `#longValue` 或 `#doubleValue`。
