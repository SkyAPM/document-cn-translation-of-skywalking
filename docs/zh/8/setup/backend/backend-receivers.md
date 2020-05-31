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

Notice, if you add these settings, make sure they are not as same as core module,
because gRPC/HTTP servers of core are still used for UI and OAP internal communications.

## Zipkin receiver

Zipkin receiver could work in two different mode.

### 1. Tracing mode(default). Tracing mode is that, skywalking OAP acts like zipkin collector,

fully supports Zipkin v1/v2 formats through HTTP service,
also provide persistence and query in skywalking UI.
But it wouldn't analysis metrics from them. In most case, I suggest you could use this feature, when metrics come from service mesh.
Notice, in this mode, Zipkin receiver requires `zipkin-elasticsearch` storage implementation active. 
Read [this](backend-storage.md#elasticsearch-6-with-zipkin-trace-extension) to know 
how to active.

Use following config to active.

```yaml
receiver_zipkin:
  default:
    host: ${SW_RECEIVER_ZIPKIN_HOST:0.0.0.0}
    port: ${SW_RECEIVER_ZIPKIN_PORT:9411}
    contextPath: ${SW_RECEIVER_ZIPKIN_CONTEXT_PATH:/}
```

### 2. Analysis mode(Not production ready), receive Zipkin v1/v2 formats through HTTP service. Transform the trace to skywalking

native format, and analysis like skywalking trace. This feature can't work in production env right now,
because of Zipkin tag/endpoint value unpredictable, we can't make sure it fits production env requirements.

Active `analysis mode`, you should set `needAnalysis` config.

```yaml
receiver_zipkin:
  default:
    host: ${SW_RECEIVER_ZIPKIN_HOST:0.0.0.0}
    port: ${SW_RECEIVER_ZIPKIN_PORT:9411}
    contextPath: ${SW_RECEIVER_ZIPKIN_CONTEXT_PATH:/}
    needAnalysis: true
```

NOTICE, Zipkin receiver is only provided in `apache-skywalking-apm-x.y.z.tar.gz` tar.

## Jaeger receiver

Jaeger receiver right now only works in `Tracing Mode`, and no analysis.
Jaeger receiver provides extra gRPC host/port, if absent, sharing-server host/port will be used, then core gRPC host/port.
Receiver requires `jaeger-elasticsearch` storage implementation active. 
Read [this](backend-storage.md#elasticsearch-6-with-jaeger-trace-extension) to know how to active.

Right now, you need [jaeger agent](https://www.jaegertracing.io/docs/1.11/architecture/#agent) to batch
send spans to SkyWalking oap server. Read [Jaeger Architecture](https://www.jaegertracing.io/docs/1.11/architecture/)
to get more details.

Active the receiver.

```yaml
receiver_jaeger:
  default:
    gRPCHost: ${SW_RECEIVER_JAEGER_HOST:0.0.0.0}
    gRPCPort: ${SW_RECEIVER_JAEGER_PORT:14250}
```

NOTICE, Jaeger receiver is only provided in `apache-skywalking-apm-x.y.z.tar.gz` tar.