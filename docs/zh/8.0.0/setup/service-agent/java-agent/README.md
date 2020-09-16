# 安装Java agent
1. 7.x版本中代理支持 JDK 8 - 14， 6.x版本支持JDK 1.6 - JDK 12 [NOTICE¹](#notice)
2. 在SkyWalking发行包中找到`agent`文件夹
3. 配置`config/agent.config`中的`agent.service_name`。可以是任意的英文字符串。
4. 配置`config/agent.config`中的`collector.backend_service`。默认指向`127.0.0.1:11800`，表示仅作用于本地后端。
5. JVM参数中添加`-javaagent:/path/to/skywalking-package/agent/skywalking-agent.jar`，并且确保这个参数在`-jar`参数之前。

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
    +-- optional-plugins
         apm-gson-2.x-plugin.jar
         .....
    +-- bootstrap-plugins
         jdk-http-plugin.jar
         .....
    +-- logs
    skywalking-agent.jar
```

- 启动你的应用。

## 支持的中间件、框架和库
SkyWalking agent已经对多种中间件、框架和库进行了支持。
通过[支持列表](Supported-list.md)可以了解对哪些进行了支持以及支持了哪些版本。
如果插件被标注为**可选²**，请到[可选的插件](#可选的插件) 章节去学习如何激活插件。

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
- Jetty  
 修改 `jetty.sh`, 在启动应用程序的命令行中添加 `-javaagent` 参数. 比如:
 ```shell
 export JAVA_OPTIONS="${JAVA_OPTIONS} -javaagent:/path/to/skywalking-agent/skywalking-agent.jar"
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
`agent.cause_exception_depth`|在记录异常信息的时候, 探针需要记录的堆栈深度.|5|
`agent.force_reconnection_period `|grpc的强制重连周期，基于grpc_channel_check_interval.|`1`|
`agent.operation_name_threshold `|设置操作名不建议超过最大长度(190).|`150`|
`agent.keep_tracing`|如果该值为 `true`，即使后台不可用，也要保持跟踪.|`false`|
`osinfo.ipv4_list_size`| 限制ipv4列表的长度. |`10`|
`collector.grpc_channel_check_interval| 检查grpc的channel状态的时间间隔。|`30`|
`collector.backend_service`|接收skywalking trace数据的后端地址|`127.0.0.1:11800`|
`collector.heartbeat_period`|探针心跳报告时间. 单位秒.|`30`|
`collector.grpc_upstream_timeout`|grpc客户端向上游发送数据时超时多长时间. 单位秒.|`30` 秒|
`collector.get_profile_task_interval`|嗅探器获取配置文件任务列表间隔.|`20`|
`logging.level`|日志级别。默认为debug。|`DEBUG`|
`logging.file_name`|日志文件名|`skywalking-api.log`|
`logging.output`| 日志输出. 默认是文件. 使用控制台意味着输出到标准输出. |`FILE`|
`logging.dir`|日志目录。默认为空串，表示使用"system.out"输出日志。|`""`|
`logging.pattern `|日志格式. 所有的转换说明符: <br>&nbsp;&nbsp;* `%level` means log level. <br>&nbsp;&nbsp;*  `%timestamp` 表示现在的时间格式 `yyyy-MM-dd HH:mm:ss:SSS`.<br>&nbsp;&nbsp;*   `%thread` 表示当前线程的名称.<br>&nbsp;&nbsp;*   `%msg` 表示用户记录的某些消息. <br>&nbsp;&nbsp;*  `%class` 表示TargetClass的SimpleName. <br>&nbsp;&nbsp;*  `%throwable` 表示用户抛出的异常. <br>&nbsp;&nbsp;*  `%agent_name` 表示 `agent.service_name`  |`%level %timestamp %thread %class : %msg %throwable`|
`logging.max_file_size`|日志文件的最大大小。当日志文件大小超过这个数，归档当前的日志文件，将日志写入到新文件。|`300 * 1024 * 1024`|
`logging.max_history_files`|The max history log files. When rollover happened, if log files exceed this number,then the oldest file will be delete. Negative or zero means off, by default.|`-1`|
`jvm.buffer_size`|收集JVM信息的buffer的大小。|`60 * 10`|
`buffer.channel_size`|buffer的channel大小。|`5`|
`buffer.buffer_size`|buffer的大小|`300`|
`profile.active`|如果为`true`，SkyWalking代理将在用户创建新的配置文件任务时启用配置文件. 否则禁用概要.|`true`|
`profile.max_parallel`|并行监控段计数|`5`|
`profile.duration`|监控段最大时间(分钟)，如果当前监控段时间超出限制，则停止.|`10`|
`profile.dump_max_stack_depth`|最大线程转储的堆栈深度|`500`|
`profile.snapshot_transport_buffer_size`|快照传输到后端缓冲区的大小|`50`|
`plugin.mongodb.trace_param`|如果为true，记录所有访问MongoDB的参数信息。默认为false，表示仅记录操作名，不记录参数信息。|`false`|
`plugin.elasticsearch.trace_dsl`|如果为true，记录所有访问ElasticSearch的DSL信息。默认为false。|`false`|
`plugin.springmvc.use_qualified_name_as_endpoint_name`|如果为true，endpoint的name为方法的全限定名，而不是请求的URL。默认为false。|`false`|
`plugin.toolit.use_qualified_name_as_operation_name`|如果为true，operation的name为方法的全限定名，而不是给定的operation name。默认为false。|`false`|
`plugin.mysql.trace_sql_parameters`|如果设置为 true, SQL 查询 (典型的是 `java.sql.PreparedStatement`) 的参数也会被采集.|`false`|
`plugin.mysql.sql_parameters_max_length`|如果设置为正整数, 收集的 SQL 参数 `db.sql.parameters` 会被截断到这个长度, 否则会被完整保存, 这可能会导致性能问题.|`512`|
`plugin.solrj.trace_statement`|如果为 true, 追踪 Solr 查询请求中的所有查询参数(包括 deleteByIds 和 deleteByQuery) 默认为 false.|`false`|
`plugin.solrj.trace_ops_params`|如果为 true, 追踪 Solr 查询中所有操作参数, 默认为 false.|`false`|
`plugin.peer_max_length `|Peer 描述最大限制.|`200`|
`plugin.mongodb.filter_length_limit`|如果设置为正数, `WriteRequest.params` 将被截短到这个长度, 否则它将被完全保存，这可能会导致性能问题.|`256`|
`plugin.postgresql.trace_sql_parameters`|如果设置为true，将收集sql的参数(通常是 `java.sql.PreparedStatement`).|`false`|
`plugin.postgresql.sql_parameters_max_length`|如果设置为正数,  `db.sql.parameters` 将被截断到这个长度，否则它将被完全保存，这可能会导致性能问题.|`512`|
`plugin.mariadb.trace_sql_parameters`|如果设置为true，将收集sql的参数(通常是 `java.sql.PreparedStatement`).|`false`|
`plugin.mariadb.sql_parameters_max_length`|如果设置为正数，`db.sql` 将被截断到这个长度，否则它将被完全保存，这可能会导致性能问题.|`512`|
`plugin.light4j.trace_handler_chain`|如果为`true`，请跟踪属于请求的Light4J处理程序链的所有中间件/业务处理程序.|false|
`plugin.opgroup.*`|支持在不同插件中自定义组规则的操作名. 请阅读 [Group rule supported plugins](op_name_group_rule.md)|Not set|
`plugin.springtransaction.simplify_transaction_definition_name`|如果为`true`，事务定义名称将被简化.|false|
`plugin.jdkthreading.threading_class_prefixes` | 线程类 (`java.lang.Runnable` and `java.util.concurrent.Callable`) 及其子类(包括任何名称匹配 `THREADING_CLASS_PREFIXES` (以 `，` 分隔)的匿名内部类) 将被检测, 确保只指定短小的前缀，就像您预期要测试的一样, (`java.` 和 `javax.` 会因安全问题而被忽略) | Not set |
`plugin.tomcat.collect_http_params`| 这个配置项控制Tomcat插件是否应该收集请求的参数. 同样，在概要追踪中隐式激活. | `false` |
`plugin.springmvc.collect_http_params`| 这个配置项控制SpringMVC插件是否应该收集请求的参数, 当您的Spring应用程序基于Tomcat时, 只需要设置 `plugin.tomcat.collect_http_params` 或 `plugin.springmvc.collect_http_params` 之一. 同样，在概要追踪中隐式激活. | `false` |
`plugin.http.http_params_length_threshold`| 当启用 `COLLECT_HTTP_PARAMS`时，要保留多少字符并将其发送到OAP后端，请使用负值来保留和发送完整的参数. 添加这个配置项是为了提高性能.  | `1024` |
`correlation.element_max_number`|关联上下文的最大元素数.|`3`|
`correlation.value_max_length`|关联上下文元素的最大值长度.|`128`|

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
* Spring Cloud Gateway 2.1.x 插件. 只有当你在 Spring Gateway 端安装了插件时才开启此插件.

## Bootstrap 类插件
由于意外风险，所有 Bootstrap 插件都是可选的。Bootstrap 插件在 `bootstrap-plugins` 文件夹中提供.
若使用这些插件，您需要将目标插件 jar 文件放入 `/plugins` 中.

现在，我们有以下已知的 Bootstrap 插件.
* JDK HttpURLConnection 插件. Agent 兼容 JDK 1.6+
* JDK Callable and Runnable 插件. Agent 兼容 1.6+

## 高级特性
* 可通过设置系统属性覆盖配置文件中的配置。请见[配置覆盖](Setting-override.md).
* 可使用gRPC TLS将后端连接起来。请见[open TLS](TLS.md)
* 通过不同的SkyWalking服务实现对大集群的监控。使用[命名空间](Namespace.md)隔离上下文传播。
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

# 注意
¹ 由于自2018年以来gRPC不支持JDK 1.6, SkyWalking 在所有 7.x 版本都放弃支持 JDK 6/7. 
但是，gRPC向前向后兼容(至少目前如此), 所有 SkyWalking 6.x agents 均可以与 7.x(包括agent 和 后台)协作.
