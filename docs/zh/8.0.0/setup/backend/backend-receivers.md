# 选择接收器

接收器是 SkyWalking 后台中的一个概念。 全模块所有负责从其它监控系统接收遥测或追踪数据的模块都被称为 **Receiver** 。如果你正在寻找拉模式，可以参考 [fetcher 文档](backend-fetcher.md)

我们有以下 receiver，Apache distribution 中提供了 `default` 实现者。

1. **receiver-trace**. gRPC 和 HTTPRestful 服务，接收 SkyWalking 格式的追踪数据。
2. **receiver-register**. gRPC 和 HTTPRestful 服务，提供服务、服务实例、端点的注册。
3. **service-mesh**. gRPC 服务，接收来自网格探针的数据。
4. **receiver-jvm**. gRPC 服务，接收 JVM 度量数据。
5. **istio-telemetry**. Istio 遥测数据来自 Istio 官方旁路适配器, 这个接收器匹配自身的 gRPC 服务.
6. **envoy-metric**. Envoy `metrics_service` 和 `ALS(access log service)` 由它提供支持。 OAL 脚本支持所有仪表类型度量。
7. **receiver-profile**. gRPC 服务，接收分析任务状态和快照报告。
8. **receiver_zipkin**. 参考 [详细](#zipkin-receiver).
9. **receiver_jaeger**. 参考 [详细](#jaeger-receiver).

这些 receiver 的示例已默认设置在 `application.yml` 文件中，在此也列一下：

```yaml
receiver-register:
  default:
receiver-trace:
  default:
    sampleRate: ${SW_TRACE_SAMPLE_RATE:1000} # 采样率精度是 1/10000. 10000 代表 100% 采样.
    slowDBAccessThreshold: ${SW_SLOW_DB_THRESHOLD:default:200,mongodb:100} # 慢数据库访问临界值，单位毫秒.
receiver-jvm:
  default:
service-mesh:
  default:
istio-telemetry:
  default:
envoy-metric:
  default:
receiver_zipkin:
  default:
    host: 0.0.0.0
    port: 9411
    contextPath: /
receiver-profile:
  default:
```

## 提供给 receiver 的 gRPC/HTTP 服务

默认情况下，所有 GRPC/HTTP 服务都应在 `core/gRPC` 和 `core/rest` 处提供。
但是 `receiver-sharing-server` 模块提供了一种方式可使所有的 receiver 提供不同的 ip:port ，当然这需要你明确地设置。

```yaml
receiver-sharing-server:
  default:
    restHost: ${SW_SHARING_SERVER_REST_HOST:0.0.0.0}
    restPort: ${SW_SHARING_SERVER_REST_PORT:12800}
    restContextPath: ${SW_SHARING_SERVER_REST_CONTEXT_PATH:/}
    gRPCHost: ${SW_SHARING_SERVER_GRPC_HOST:0.0.0.0}
    gRPCPort: ${SW_SHARING_SERVER_GRPC_PORT:11800}
```

注意，如果添加这些设置，请确保它们与核心模块不同，因为核心的 GRPC/HTTP 服务器仍用于 UI 和 OAP 内部通信。

## Zipkin receiver

Zipkin receiver 可以在两种不同的模式下工作。

### 1. Tracing mode(default)

追踪模式就是，skywalking 的 OAP 表现得像个 zipkin 收集器，通过 HTTP 服务完全支持 Zipkin 的 v1/v2 格式， 还在 Skywalking 用户界面提供了持久化和查询。但它不会从中分析度量标准。在大多数情况下，当度量来自 service mesh 时，我建议你可以使用这个特性。

注意，在这种模式下，Zipkinr receiver 需要激活 `zipkin-elasticsearch` 的存储实现。阅读[这里](backend-storage.md#elasticsearch-6-with-zipkin-trace-extension)了解如何激活。

使用以下配置来激活

```yaml
receiver_zipkin:
  default:
    host: ${SW_RECEIVER_ZIPKIN_HOST:0.0.0.0}
    port: ${SW_RECEIVER_ZIPKIN_PORT:9411}
    contextPath: ${SW_RECEIVER_ZIPKIN_CONTEXT_PATH:/}
```

### 2. Analysis mode(非生产环境准备)

通过 HTTP 服务接收 Zipkin 的 v1/v2 格式。将 trace 转换为 skywalking 原生的格式，并像 skywalking trace 那样进行分析。这个特性尚不能在生产环境使用，这是因为 zipkin tag/endpoint 不可预测，我们无法确保它符合生产环境要求。

激活 `analysis mode`, 你需要设置 `needAnalysis` 配置。

```yaml
receiver_zipkin:
  default:
    host: ${SW_RECEIVER_ZIPKIN_HOST:0.0.0.0}
    port: ${SW_RECEIVER_ZIPKIN_PORT:9411}
    contextPath: ${SW_RECEIVER_ZIPKIN_CONTEXT_PATH:/}
    needAnalysis: true
```

注意，Zipkin receiver 只提供在 `apache-skywalking-apm-x.y.z.tar.gz` tar 中。

## Jaeger receiver

Jaeger receiver 目前只在 `Tracing Mode(跟踪模式)` 下工作，不支持分析。Jaeger receiver 提供额外的 gRPC host/port，如果没有就使用 sharing-server host/port， 还没有就使用 core gRPC host/port。

Receiver 需要激活 `jaeger-elasticsearch` 的存储实。阅读 [此处](backend-storage.md#elasticsearch-6-with-jaeger-trace-extension) 了解如何激活.

同时，你需要 [jaeger agent](https://www.jaegertracing.io/docs/1.11/architecture/#agent) 批量发送 span 至 SkyWalking oap server。 详细参考 [Jaeger Architecture](https://www.jaegertracing.io/docs/1.11/architecture/)

激活 Jaeger receiver

```yaml
receiver_jaeger:
  default:
    gRPCHost: ${SW_RECEIVER_JAEGER_HOST:0.0.0.0}
    gRPCPort: ${SW_RECEIVER_JAEGER_PORT:14250}
```

注意，Jaeger receiver 只提供在 `apache-skywalking-apm-x.y.z.tar.gz` tar 中。
