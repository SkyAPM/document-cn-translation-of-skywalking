# 追踪数据协议

追踪数据协议描述了 SkyWalking 代理/sniffer 和后端之间的数据交互格式. 

## 摘要

本协议包含了上行数据和下行数据的数据格式。其他语言的代理和 SDK 可以使用这个协议来将数据上传到 SkyWalking 后端。

- 其他服务都是通过 HTTP/JSON 和 gRPC 两者提供，包括注册服务，追踪服务等。

### 版本

v1 版本

#### gRPC proto 文件

[gRPC proto 文件](https://github.com/apache/incubator-skywalking-data-collect-protocol/tree/v2.0)

## 追踪段服务

[gRPC 服务定义](https://github.com/apache/incubator-skywalking-data-collect-protocol/blob/v2.0/TraceSegmentService.proto)

- UniqueId 代表段的 ID （segmentId）和全局追踪 ID（globalTraceId）。它包含三部分（long 类型）。
  1. 应用实例 ID（applicationInstanceId）
  1. 线程 ID（ThreadId）
  1. 时间戳（Timestamp）+ 10000 + seq(seq is in [0, 100000) )
- Span 数据请参考 [插件开发指南](../guides/Java-Plugin-Development-Guide.md)
- 当以下 ID 和名称（name）同时存在时，在可能的情况下优先使用 ID
  - operationNameId/endpointName 
  - networkAddress/networkAddressId
  - entryServiceName/entryServiceId
  - parentServiceName/parentServiceId
  - peerId/peer
- 组件 ID（componentIds）在后端中定义，参考[此处](https://github.com/apache/incubator-skywalking//blob/master/apm-protocol/apm-network/src/main/java/org/apache/skywalking/apm/network/trace/component/ComponentsDefine.java)

HTTP 格式:

输入：

```
[
  {
    "gt": [[230150, 185809, 24040000]], 
    "sg": { //TraceSegmentObject 
      "ts": [137150, 185809, 48780000], 
      "ai": 2, //serviceId
      "ii": 3, //applicationInstanceId
      "ss": [ //SpanObject
        {
          "si": 0, //spanId
          "tv": 0, //SpanType
          "lv": 2, //SpanLayer
          "ps": -1, //parentSpanId
          "st": 1501858094726, //startTime
          "et": 1501858096804, //endTime
          "ci": 3, //componentId
          "cn": "", //component
          "oi": 0, //operationNameId
          "on": "org.skywaking.apm.testcase.dubbo.services.GreetService.doBusiness()", //endpointName
          "pi": 0, //peerId
          "pn": "", //peer
          "ie": false, //isError
          "rs": [ //TraceSegmentReference
            {
              "pts": [230150, 185809, 24040000], //parentTraceSegmentId
              "pii": 2, //parentServiceInstanceId
              "psp": 1, //parentSpanId
              "psi": 0, //parentServiceId
              "psn": "/dubbox-case/case/dubbox-rest", //parentServiceName
              "ni": 0,  //networkAddressId
              "nn": "172.25.0.4:20880", //networkAddress
              "eii": 2, //entryServiceInstanceId
              "esi": 0, //entryServiceId
              "esn": "/dubbox-case/case/dubbox-rest", //entryServiceName
              "rv": 0 //RefTypeValue
            }
          ],
          "to": [ //KeyWithStringValue
            {
              "k": "url", //key
              "v": "rest://172.25.0.4:20880/org.skywaking.apm.testcase.dubbo.services.GreetService.doBusiness()" //value
            },
            {
              "k": "http.method",
              "v": "GET"
            }
          ],
          "lo": [{
                "ti": 1501858094726,
                "ld": [{ 
                        "k": "NullPointException",
                        "v": "Error Stack"
                    }]
           }]
        },
        {
          "si": 1,
          "tv": 1,
          "lv": 1,
          "ps": 0,
          "st": 1501858094726,
          "et": 1501858095804,
          "ci": 9,
          "cn": "",
          "oi": 0,
          "on": "mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]",
          "pi": 0,
          "pn": "localhost:27017",
          "ie": false,
          "to": [],
          "lo": []
        }
      ]
    }
  }
]
```

## 弃用的服务

**弃用的服务** 是 SkyWalking 以前使用的 gRPC 服务。在 SkyWalking 6 以后，为了适配云原生（CloudNative）的通用概念，这些服务被废弃了。
尽管这些服务此时仍然支持，但是预计在 2019 年后将不再支持。

## ~~应用注册服务~~

**弃用的服务** 

### 摘要

注册应用代码到后端中，且返回一个整数标识该应用。

[gRPC 服务定义](https://github.com/apache/incubator-skywalking-data-collect-protocol/blob/v2.0/ApplicationRegisterService.proto)

- 应用编码（applicationCode）是配置在你 `agent.config` 中的.
- 返回的 ID 是**应用 ID（ApplicationId）**，存储在 `KeyWithIntegerValue` 的 `value` 中，会在后续数据上传中使用到。

## ~~发现服务~~

**弃用的服务** 

### ~~注册实例服务~~

[gRPC 服务定义](https://github.com/apache/incubator-skywalking-data-collect-protocol/blob/v2.0/DiscoveryService.proto#L29)

- 代理唯一键（agentUUID）由代理生成，必须全局唯一，且至少在重启之前保持一致
- **应用实例 ID（ApplicationInstanceId）** 将会在后续数据上传过程中使用

HTTP 格式 http://ip:port/instance/register(默认: localhost:12800) 

输入:

```
{
    ai: x, #serviceId
    au: "", #agentUUID
    rt: x, #registerTime
    oi: "", #osinfo
}
```

输出:

```
{
    ai: x, #serviceId
    ii: x, #applicationInstanceId
}
```

### ~~心跳服务~~

[gRPC 服务定义](https://github.com/apache/incubator-skywalking-data-collect-protocol/blob/v2.0/DiscoveryService.proto#L32)

- 推荐每隔 20-60 秒进行一次心跳检测。
- Java 代理不会使用这个服务，因为 JVM 指标上游替换了此服务的能力。 

HTTP 格式 http://ip:port/instance/heartbeat(默认: localhost:12800) 

输入:

```
{
    "ii": x, #applicationInstanceId
    "ht": x #heartbeatTime, java timestamp format
}
```

## ~~服务名称发现服务~~

**弃用的服务**

### 摘要

使用整型 ID 替换字符串类型的服务（操作）名称

[gRPC 服务定义](.https://github.com/apache/incubator-skywalking-data-collect-protocol/blob/v2.0/DiscoveryService.proto#L70)

- 可选的服务，降低网络开销，但增大内存使用（因为使用了缓冲映射）

HTTP 格式 http://ip:port/servicename/discovery(默认: localhost:12800) 

输入：

```
{
    ai: x, #serviceId
    sn: "", #serviceName
    st: x, #srcSpanType
}
```

输出：

```
{
    si: x, #osinfo
    el: { #element
        ai: x, #serviceId
        sn: "", #serviceName
        st: x, #srcSpanType
    }
}
```

## ~~网络地址注册服务~~

### 摘要

网络地址包括所有已经被移除的服务的地址，包含 IP，端口，主机等，这些在 RPC 框架和消息队列、数据库等中使用。

[gRPC 服务定义](https://github.com/apache/incubator-skywalking-data-collect-protocol/blob/v2.0/NetworkAddressRegisterService.proto)

- 可选的服务，降低网络开销，但增大内存使用（因为使用了缓冲映射）

