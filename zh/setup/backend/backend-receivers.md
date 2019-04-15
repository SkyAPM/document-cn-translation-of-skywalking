# 选择Receiver
Receiver是SkyWalking backend中的一个概念。 所有负责从其它监控系统接收测量或追踪数据的模块都被称为**Receiver** 。虽然目前大多数Receiver都使用gRPC或HTTPRestful来提供服务，事实上，不管是监听模式或者拉取模式都可以是receiver。Receiver可以基于从远程拉取数据，如Kakfa MQ。

我们有以下receiver，Apache distribution.zzz中提供了`default`实现者。
1. **receiver-trace**。 gRPC和HTTPRestful服务，接收skywalk格式的跟踪。
1. **receiver-register**.。gRPC和HTTPRestful服务，提供服务,服务实例和终端的注册。.
1. **service-mesh**。 gRPC服务，接收来自入站网格探针的数据。
1. **receiver-jvm**。 gRPC services，接收JVM度量数据。.
1. **istio-telemetry**。 ISTIO遥测来自ISTIO官方的旁路适配器，这个Receiver匹配它自己的gRPC服务。
1. **envoy-metric**. Envoyd的`metrics_service` 由它提供支持。 OAL脚本支持所有仪表类型度量。
1. **receiver_zipkin**. 详见 [details](#zipkin-receiver).
1. **receiver_jaeger**. 详见 [details](#jaeger-receiver).

这些receiver的示例设置已在默认的`application.yml`文件中，在此也列一下：
```yaml
receiver-register:
  default:
receiver-trace:
  default:
    bufferPath: ../trace-buffer/  # Path to trace buffer files, suggest to use absolute path
    bufferOffsetMaxFileSize: 100 # Unit is MB
    bufferDataMaxFileSize: 500 # Unit is MB
    bufferFileCleanWhenRestart: false
    sampleRate: ${SW_TRACE_SAMPLE_RATE:1000} # The sample rate precision is 1/10000. 10000 means 100% sample in default.
receiver-jvm:
  default:
service-mesh:
  default:
    bufferPath: ../mesh-buffer/  # Path to trace buffer files, suggest to use absolute path
    bufferOffsetMaxFileSize: 100 # Unit is MB
    bufferDataMaxFileSize: 500 # Unit is MB
    bufferFileCleanWhenRestart: false
istio-telemetry:
  default:
envoy-metric:
  default:
receiver_zipkin:
  default:
    host: 0.0.0.0
    port: 9411
    contextPath: /
```

## 提供给receiver的gRPC/HTTP服务
默认情况下，所有GRPC/HTTP服务都应在`core/gRPC`和`core/rest`处提供。
但是`receiver-sharing-server`模块提供了一种方式可使所有的receiver提供不同的ip:port，当然这需要你明确地设置。
```yaml
receiver-sharing-server:
  default:
    restHost: ${SW_SHARING_SERVER_REST_HOST:0.0.0.0}
    restPort: ${SW_SHARING_SERVER_REST_PORT:12800}
    restContextPath: ${SW_SHARING_SERVER_REST_CONTEXT_PATH:/}
    gRPCHost: ${SW_SHARING_SERVER_GRPC_HOST:0.0.0.0}
    gRPCPort: ${SW_SHARING_SERVER_GRPC_PORT:11800}
```

注意，如果添加这些设置，请确保它们与核心模块不同，因为核心的GRPC/HTTP服务器仍用于UI和OAP内部通信。

## Zipkin receiver
Zipkin receiver可以在两种不同的模式下工作。
1. Tracing mode(默认)。追踪模式就是，skywalking的OAP表现得像个Zipkin收集器，通过HTTP服务完全支持Zipkin的v1/v2格式， 还在Skywalking用户界面提供了持久化和查询。但它不会从中分析度量标准。在大多数情况下，当度量来自service mesh时，我建议你可以使用这个特性。                    
注意，在这种模式下，Zipkinr eceiver需要激活`zipkin-elasticsearch`的存储实现。阅读[this]（backend storage.md elasticsearch-6-with-zipkin-trace-extension）了解如何激活。

使用以下配置来激活.
```yaml
receiver_zipkin:
  default:
    host: ${SW_RECEIVER_ZIPKIN_HOST:0.0.0.0}
    port: ${SW_RECEIVER_ZIPKIN_PORT:9411}
    contextPath: ${SW_RECEIVER_ZIPKIN_CONTEXT_PATH:/}
```

2. Analysis mode(Not production ready),。通过HTTP服务接收Zipkin的v1/v2格式。将trace转换为skywalking原生的格式，并像skywalking trace那样进行分析。这个特性尚不能在生产环境使用，这是因为zipkin标签/端点值不可预测，我们无法确保它符合生产环境要求。

要激活 `analysis mode`, 你需要设置`needAnalysis`配置
```yaml
receiver_zipkin:
  default:
    host: ${SW_RECEIVER_ZIPKIN_HOST:0.0.0.0}
    port: ${SW_RECEIVER_ZIPKIN_PORT:9411}
    contextPath: ${SW_RECEIVER_ZIPKIN_CONTEXT_PATH:/}
    needAnalysis: true
```

## Jaeger receiver
Jaeger receiver 目前只在跟踪模式下工作，不支持分析模式。Jaeger receiver提供额外的GRPC主机/端口，如果没有，将使用共享服务器主机/端口，还没有就使用核心GRPC主机/端口。Receiver需要激活Jaeger ElasticSearch存储实现。阅读此内容了解如何激活。right now only works in `Tracing Mode`, and no analysis.
Jaeger receiver provides extra gRPC host/port, if absent, sharing-server host/port will be used, then core gRPC host/port.
Receiver requires `jaeger-elasticsearch` storage implementation active. 
Read [this](backend-storage.md#elasticsearch-6-with-jaeger-trace-extension) to know how to active.

现在，你需要[jaeger agent](https://www.jaegertracing.io/docs/1.11/architecture/#agent) 来批量发送spans到 SkyWalking的oap服务器。
阅读[Jaeger Architecture](https://www.jaegertracing.io/docs/1.11/architecture/)获取更多详情。

激活这个receiver。
```yaml
receiver_jaeger:
  default:
    gRPCHost: ${SW_RECEIVER_JAEGER_HOST:0.0.0.0}
    gRPCPort: ${SW_RECEIVER_JAEGER_PORT:14250}
``` 