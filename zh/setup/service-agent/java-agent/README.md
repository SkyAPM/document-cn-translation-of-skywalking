# 安装Java agent
1. 在SkyWalking发行包中找到`agent`文件夹
1. 配置`config/agent.config`中的`agent.service_name`。可以是任意的英文字符串。
1. 配置`config/agent.config`中的`collector.backend_service`。默认指向`127.0.0.1:11800`，表示仅作用于本地后端。
1. JVM参数中添加`-javaagent:/path/to/skywalking-package/agent/skywalking-agent.jar`，并且确保这个参数在`-jar`参数之前。

agent发行包包含在Apache[官方发行版](http://skywalking.apache.org/downloads/)中。最新的agent包结构如下。
```
+-- agent
    +-- activations
         apm-toolkit-log4j-1.x-activation.jar
         apm-toolkit-log4j-2.x-activation.jar
         apm-toolkit-logback-1.x-activation.jar
         ...
    +-- config
         agent.config  
    +-- plugins
         apm-dubbo-plugin.jar
         apm-feign-default-http-9.x.jar
         apm-httpClient-4.x-plugin.jar
         .....
    skywalking-agent.jar
```

- 启动你的应用。

## 支持的中间件、框架和库
SkyWalking agent已经对多种中间件、框架和库进行了支持。
通过[支持列表](Supported-list.md)可以了解对哪些进行了支持以及支持了哪些版本。
如果插件被标注为**可选²**，请到[可选的插件](#optional-plugins) 章节去学习如何激活插件。

## 高级特性
- 所有的插件都在`/plugins`文件夹下。当某个插件的jar包在此文件夹下，表示此插件已被激活；从这个文件夹下移除后，表示插件不可用. 
- 默认的日志输出文件夹为`/logs`。

## 关于安装java agent的FAQ
- Linux Tomcat 7, Tomcat 8  
修改`tomcat/bin/catalina.sh`的第一行。
```shell
CATALINA_OPTS="$CATALINA_OPTS -javaagent:/path/to/skywalking-agent/skywalking-agent.jar"; export CATALINA_OPTS
```

- Windows Tomcat 7, Tomcat 8  
修改`tomcat/bin/catalina.bat`的第一行。
```shell
set "CATALINA_OPTS=-javaagent:/path/to/skywalking-agent/skywalking-agent.jar"
```
- JAR包
使用命令行启动应用时，添加`-javaagent`参数。比如： 
 ```shell
 java -javaagent:/path/to/skywalking-agent/skywalking-agent.jar -jar yourApp.jar
 ```
 
## Agent的可配置属性列表
这里是`agent/config/agent.config`中支持的属性列表。

属性名 | 描述 | 默认值 |
----------- | ---------- | --------- | 
`agent.namespace` | 命名空间，用于隔离跨进程传播的header。如果进行了配置，header将为`HeaderName:Namespace`. | 未设置 | 
`agent.service_name` | 在SkyWalking UI中展示的服务名。5.x版本对应Application，6.x版本对应Service。 建议：为每个服务设置个唯一的名字，服务的多个服务实例为同样的服务名 | `Your_ApplicationName` |
`agent.sample_n_per_3_secs`| 负数或0表示不采样，默认不采样。SAMPLE_N_PER_3_SECS表示每3秒采样N条。|未设置|
`agent.authentication`|鉴权是否开启取决于后端的配置，可查看application.yml的详细描述。对于大多数的场景，需要后端对鉴权进行扩展。目前仅实现了基本的鉴权功能。|未设置|
`agent.span_limit_per_segment`|单个segment中的span的最大个数。通过这个配置项，Skywalking可评估应用程序内存使用量。| 未设置|
`agent.ignore_suffix`|如果这个集合中包含了第一个span的操作名，这个segment将会被忽略掉。|未设置|
`agent.is_open_debugging_class`|如果为true，skywalking会将所有经Instrument转换过的类文件保存到`/debugging`文件夹下。Skywalking团队会要求你提供这些类文件以解决兼容性问题。|未设置|
`agent.active_v2_header`|是否默认使用v2版本的header。|`true`|
`agent.instance_uuid` |实例id。skywalking会将实例id相同的看做一个实例。如果为空，skywalking agent会生成一个32位的uuid。|`""`|
`agent.active_v1_header `|是否默认使用v1版本的header。|`false`|
`collector.grpc_channel_check_interval`|检查grpc的channel状态的时间间隔。|`30`|
`collector.app_and_service_register_check_interval`|检查应用和服务的注册状态的时间间隔。|`3`|
`collector.backend_service`|接收skywalking trace数据的后端地址|`127.0.0.1:11800`|
`logging.level`|日志级别。默认为debug。|`DEBUG`|
`logging.file_name`|日志文件名|`skywalking-api.log`|
`logging.dir`|日志目录。默认为空串，表示使用"system.out"输出日志。|`""`|
`logging.max_file_size`|日志文件的最大大小。当日志文件大小超过这个数，归档当前的日志文件，将日志写入到新文件。|`300 * 1024 * 1024`|
`jvm.buffer_size`|收集JVM信息的buffer的大小。|`60 * 10`|
`buffer.channel_size`|buffer的channel大小。|`5`|
`buffer.buffer_size`|buffer的大小|`300`|
`dictionary.service_code_buffer_size`|The buffer size of application codes and peer|`10 * 10000`|
`dictionary.endpoint_name_buffer_size`|The buffer size of endpoint names and peer|`1000 * 10000`|
`plugin.mongodb.trace_param`|如果为true，记录所有访问MongoDB的参数信息。默认为false，表示仅记录操作名，不记录参数信息。|`false`|
`plugin.elasticsearch.trace_dsl`|如果为true，记录所有访问ElasticSearch的DSL信息。默认为false。|`false`|
`plugin.springmvc.use_qualified_name_as_endpoint_name`|如果为true，endpoint的name为方法的全限定名，而不是请求的URL。默认为false。|`false`|
`plugin.toolit.use_qualified_name_as_operation_name`|如果为true，operation的name为方法的全限定名，而不是给定的operation name。默认为false。|`false`|

## 可选的插件 
Java agent的所有插件都是可插拔的。在agent或第三方仓库的`optional-plugins` 文件夹下提供了可选的插件。
想要使用可选插件，你需要将对应插件移动到`/plugins`文件夹下。

目前，我们提供了如下的可选插件。
* [追踪Spring注解的bean](agent-optional-plugins/Spring-annotation-plugin.md)
* [追踪Oracle和Resin](agent-optional-plugins/Oracle-Resin-plugins.md)
* [通过指定endpoint模式来过滤trace](agent-optional-plugins/trace-ignore-plugin.md)
* Gson序列化库
* Lettuce 5.x(JRE1.8+)
* Zookeeper 3.4.x。 将此插件定为可选插件的原因在于：会生成大量的业务不相关的trace，对agent和后端产生了额外的负载。并且，这些trace数据可能仅仅是心跳数据。
* [自定义增强](Customize-enhance-trace.md) 基于描述文件对方法进行追踪，而不是通过写插件或修改源代码。

## 高级特性
* 可通过设置系统属性覆盖配置文件中的配置。请见[配置覆盖](Setting-override.md).
* 可使用gRPC TLS将后端连接起来。请见[open TLS](TLS.md)
* 通过不同的SkyWalking服务实现对大集群的监控。使用[命令空间](Namespace.md)隔离上下文传播。
* 如果后端开启了[token鉴权](../../backend/backend-token-auth.md)，客户端可设置[token](Token-auth.md)。
* 应用工具包。应用工具包，是Skywalking提供的一些库的集合。通过这些库，可以将你的应用同Skywalking agent联系起来。
    * 如果你想使用OpenTracing的Java API，可以试试[兼容OpenTracing的Skywalking tracer](Opentracing.md). 更多细节可以查看http://opentracing.io
    * 如果你想在你的日志中打印trace上下文（比如traceId），选择你使用的日志框架[log4j](Application-toolkit-log4j-1.x.md), 
[log4j2](Application-toolkit-log4j-2.x.md), [logback](Application-toolkit-logback-1.x.md)
    * 如果你想使用注解或者SkyWalking本地API读取trace上下文，试一下[SkyWalking manual APIs](Application-toolkit-trace.md)吧。
    * 如果你想通过手动的方式实现trace跨线程传递，可以使用[跨线程传递API](Application-toolkit-trace-cross-thread.md)。
* 如果你想指定你的agent.config文件的路径，请见[通过系统属性设置配置文件路径](Specified-agent-config.md)

## 插件开发指南
SkyWalking java agent支持插件的开发以扩展[支持列表](Supported-list.md)。如果你想开发java agent插件，请阅读[插件开发指南](../../../guides/Java-Plugin-Development-Guide.md)。

# 测试
如果你对插件的兼容性测试或agent的性能感兴趣，可以阅读下面的测试报告。
* [Plugin Test](https://github.com/SkyAPMTest/agent-integration-test-report)
* [Java Agent Performance Test](https://skyapmtest.github.io/Agent-Benchmarks/)
