# 手动打点 SDK

我们尚未正式提供手动打点 SDK, 欢迎贡献以下语言的 SDK:
- Go
- Python
- C++

## SkyWalking 的数据格式和传播协议是什么

从[协议文档](../protocols/README.md)中了解这些协议的详情.

## SkyWalking 能够提供以上语言的 OpenCensus 导出器吗

写作本文档的时候, **不能**. 因为 OpenCensus 没有提供上下文可扩展的机制, 在操作 span 的时候也没有钩子(hook)机制.
而 SkyWalking 要依赖这些来传播除 trace id 和 span id 之外的更多内容.

我们已经进行了讨论, 参考[Issue](https://github.com/census-instrumentation/opencensus-specs/issues/70),
在 OpenCensus 官方提供了以上所缺的功能之后, 我们就可以提供相应的导出器了.

## Zipkin 打点 SDK 怎么样

参考[Zipkin 接收器](../setup/backend/backend-receivers.md)的**选择接收器**一节. 
