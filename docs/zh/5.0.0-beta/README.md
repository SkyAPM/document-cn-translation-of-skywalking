# 欢迎
**这是 SkyWalking 5 官方文档的中文翻译版, 由社区提供, 欢迎加入我们**

在这里你可以了解到所有有关 **SkyWalking** 的架构, 如何部署并使用 SkyWalking, 以及如何基于 SkyWalking 进行开发.

  * 快速入门
    * [快速入门](Quick-start-CN.md)
    * [中间件，框架与类库支持列表](Supported-list.md)
        * [如何关闭特定插件](How-to-disable-plugin-CN.md)
        * [可选插件](Optional-plugins-CN.md)
  * 高级特性
    * [通过系统启动参数进行覆盖配置](Setting-override-CN.md)
    * [服务直连(Direct uplink)及禁用名称服务(naming service)](Direct-uplink-CN.md)
    * [开启TLS](TLS-CN.md)
    * [命名空间隔离](Namespace-CN.md)
    * [基于Token认证](Token-auth-CN.md)
  * APM相关介绍资料
    * [OpenTracing中文版](https://github.com/opentracing-contrib/opentracing-specification-zh)
  * Application Toolkit，应用程序工具包
    * [概述](Application-toolkit-CN.md)
    * [OpenTracing Tracer](Opentracing-CN.md)
    * 日志组件
      * [log4j组件](Application-toolkit-log4j-1.x-CN.md)
      * [log4j2组件](Application-toolkit-log4j-2.x-CN.md)
      * [logback组件](Application-toolkit-logback-1.x-CN.md)
    * [Trace](Application-toolkit-trace-CN.md)
    * [调用链跨线程传递](Application-toolkit-trace-cross-thread-CN.md) 
  * 测试用例
    * [插件测试](https://github.com/SkywalkingTest/agent-integration-test-report)
    * [Java 探针性能测试](https://skywalkingtest.github.io/Agent-Benchmarks/README_zh.html)
  * 开发指南
    * [工程编译指南](How-to-build-CN.md)
    * [插件开发指南](Plugin-Development-Guide-CN.md)
    * 交互协议
        * [Cross Process Propagation Headers Protocol, v1.0  跨进程追踪上下文传递协议](Skywalking-Cross-Process-Propagation-Headers-Protocol-CN-v1.md)
        * [SkyWalking Trace Data Protocol 探针与Collector间网络协议](Trace-Data-Protocol-CN.md)
  * [Roadmap](ROADMAP.md)
  * 社区提供的共享资源
    * [公开演讲](https://github.com/OpenSkywalking/Community#public-speakings)
    * [视频](https://github.com/OpenSkywalking/Community#videos)
    * [文章](https://github.com/OpenSkywalking/Community#articles)
  * FAQ
    * [Trace查询有数据，但是没有拓扑图和JVM数据?](FAQ/Why-have-traces-no-others-CN.md)
    * [加载探针，Console被GRPC日志刷屏](FAQ/Too-many-gRPC-logs-CN.md)
    * [Kafka消息消费端链路断裂](FAQ/Kafka-plugin-CN.md)
    * [Protoc-Plugin Maven编译时异常](FAQ/Protoc-Plugin-Fails-When-Build-CN.md)
    * [EnhanceRequireObjectCache 类转换异常](FAQ/EnhanceRequireObjectCache-Cast-Exception-CN.md)
    
