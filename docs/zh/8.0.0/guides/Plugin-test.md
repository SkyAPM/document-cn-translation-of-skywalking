# 插件自动测试框架

插件测试框架旨在验证插件的功能和兼容状态。由于有数十个插件和数百个版本需要验证，因此无法手动进行。
测试框架使用基于容器的技术堆栈，需要一组安装了代理程序的真实服务，然后运行测试模拟OAP后端以检查从代理程序发送的分段数据。
主仓库中维护的每个插件都需要相应的测试用例，并且还与受支持的列表文档中的版本匹配。

## 环境要求

1. MacOS/Linux
2. jdk 8+
3. Docker
4. Docker Compose

## 用例库镜像简介

该测试框架提供了 `JVM-container` 和 `Tomcat-container` 基础镜像。您可以为您的测试用例选择合适的一个，如果其中一个合适，建议选择`JVM-container` 。

### JVM容器镜像简介

[JVM-container](../../../test/plugin/containers/jvm-container) 使用 `openjdk:8` 作为基础镜像。
必须使用“ mvn clean package”将测试用例项目打包为“ project-name.zip”，包括“ startup.sh”和uber jar。

以以下测试项目为例

* [sofarpc-scenario](../../../test/plugin/scenarios/sofarpc-scenario)作为单个项目用例。
* [webflux-scenario](../../../test/plugin/scenarios/webflux-scenario) 作为包括多个项目的用例。

### Tomcat容器镜像简介

[Tomcat-container](../../../test/plugin/containers/tomcat-container) 使用 `tomcat:8.5.42-jdk8-openjdk` 作为基础镜像。
必须使用 `mvn package` 将测试用例项目打包为 `project-name.war`。

Take the following test project as a good example

* [spring-4.3.x-scenario](https://github.com/apache/skywalking/tree/master/test/plugin/scenarios/spring-4.3.x-scenario)

## 测试项目层次结构

该测试用例是一个独立的Maven项目，需要将其打包为war tar 文件或zip文件，具体取决于所选的基本镜像。此外，还需要两个外部可访问端点，主要是两个URL。

所有的测试用例代码都应该在`org.apache.skywalking.apm.testcase`包中，除非期望使用某些代码，否则这些类可以在`test.org.apache.skywalking.apm.testcase`包中。

**JVM容器测试项目层次结构**

```
[plugin-scenario]
    |- [bin]
        |- startup.sh
    |- [config]
        |- expectedData.yaml
    |- [src]
        |- [main]
            |- ...
        |- [resource]
            |- log4j2.xml
    |- pom.xml
    |- configuration.yaml
    |- support-version.list

[] = directory
```

**Tomcat容器测试项目层次结构**

```
[plugin-scenario]
    |- [config]
        |- expectedData.yaml
    |- [src]
        |- [main]
            |- ...
        |- [resource]
            |- log4j2.xml
        |- [webapp]
            |- [WEB-INF]
                |- web.xml
    |- pom.xml
    |- configuration.yaml
    |- support-version.list

[] = directory
```

## 测试用例配置文件

每个测试用例都需要以下文件。

文件名| 内容描述
---|---
`configuration.yml` | 声明基本的测试用例介绍, 包括, 用例名称, 入口端点，模式，依赖项。
`expectedData.yaml` | 描述预期的追踪段数据。
`support-version.list` | 列出此用例的目标版本。
`startup.sh` | 仅仅在`JVM-container` 中需要, 当使用`Tomcat-container`时不需要。

`*` `support-version.list` 格式要求每一行是个单独版本。可以使用`＃`来注释该版本。

### configuration.yml

| 参数 | 描述
| --- | ---
| type | 镜像类型, 可选 `jvm` 或者 `tomcat`。必填。
| entryService | 用于测试用例访问的入口端点(URL)。必填。 (HTTP Method: GET)
| healthCheck | 用于测试用例健康检查的端点(URL)。必填。 (HTTP Method: HEAD)
| startScript | 启动脚本的路径。仅在镜像类型为jvm(`type: jvm`)时必填。
| framework | 用例名称.
| runningMode | 运行模式是否带有可选插件,可选, 默认`default`, `with_optional`, `with_bootstrap`
| withPlugins | 插件选择规则。例:`apm-spring-annotation-plugin-*.jar`。当 `runningMode=with_optional` 或 `runningMode=with_bootstrap`必填。
| environment | 例如`docker-compose#environment`.
| depends_on | 例如`docker-compose#depends_on`.
| dependencies | 例如`docker-compose#services`, 支持`image、links、hostname、environment、depends_on`。

**注意：仅当`dependencies`为空时，`docker-compose`才有效。**

**runningMode**选项说明。

| 选项 | 描述
| --- | ---
| default | 激活`plugin`文件夹中的所有插件，例如官方发行的插件。 
| with_optional | 激活`default`模式和`optional-plugin`文件夹里面的通过指定选择器指定的插件。
| with_bootstrap |激活`default`模式和`bootstrap-plugin`文件夹里面的通过指定选择器指定的插件。

with_optional/with_bootstrap 支持多个选择器,以`;`分割.

**文件格式**

```
type:
entryService:
healthCheck:
startScript:
framework:
runningMode:
withPlugins:
environment:
  ...
depends_on:
  ...
dependencies:
  service1:
    image:
    hostname:
    expose:
      ...
    environment:
      ...
    depends_on:
      ...
    links:
      ...
    entrypoint:
      ...
    healthcheck:
      ...
```

* 依赖项支持docker撰写`healthcheck`。但是格式略有不同。我们需要将-作为每个配置项的开头，并将其描述为字符串行。

例如在官方文档中，健康检查是

```yaml
healthcheck:
  test: ["CMD", "curl", "-f", "http://localhost"]
  interval: 1m30s
  timeout: 10s
  retries: 3
  start_period: 40s
```

在这里，应该写成

```yaml
healthcheck:
  - 'test: ["CMD", "curl", "-f", "http://localhost"]'
  - "interval: 1m30s"
  - "timeout: 10s"
  - "retries: 3"
  - "start_period: 40s"
```

在某些情况下，需要依赖服务（主要是第三方服务器，例如SolrJ服务器）来保持与客户端库版本相同的版本，该库在pom中定义为`$ {test.framework.version}`。
可以将`${CASE_SERVER_IMAGE_VERSION}`用作版本号，它将在测试中针对每个版本进行更改。

> 不支持与资源相关的配置，例如卷，端口和ulimit。因为在测试场景中 
> 不需要将任何端口映射到主机VM，也无需安装任何文件夹。

**以以下测试用例可以作为参考**

* [dubbo-2.7.x with JVM-container](../../../test/plugin/scenarios/dubbo-2.7.x-scenario/configuration.yml)
* [jetty with JVM-container](../../../test/plugin/scenarios/jetty-scenario/configuration.yml)
* [gateway with runningMode](../../../test/plugin/scenarios/gateway-2.1.x-scenario/configuration.yml)
* [canal with docker-compose](../../../test/plugin/scenarios/canal-scenario/configuration.yml)

### expectedData.yaml

**数字运算符**

| 运算符 | 描述 |
| :--- | :--- |
| `nq` | != |
| `eq` | ==(默认情况下) |
| `ge` | >= |
| `gt` | > |

**字符串运算符**

| 运算符 | 描述 |
| :--- | :--- |
| `not null` | 不为空 |
| `null` | 空或者空字符串 |
| `eq` | 等于(默认情况下) |

**细分目标描述格式**

```yml
segmentItems:
-
  serviceName: SERVICE_NAME(string)
  segmentSize: SEGMENT_SIZE(int)
  segments:
  - segmentId: SEGMENT_ID(string)
    spans:
        ...
```

| 属性 |  描述
| --- | ---  
| serviceName | 服务名称。
| segmentSize | 预期的目标数量。
| segmentId | 链路ID。
| spans | 追踪段跨度列表。遵循下一部分，以了解如何描述每个跨度。

**跨度验证描述格式**

**注意**：跨度列表的顺序应遵循跨度完成时间的顺序。

```yml
    operationName: OPERATION_NAME(string)
    parentSpanId: PARENT_SPAN_ID(int)
    spanId: SPAN_ID(int)
    startTime: START_TIME(int)
    endTime: END_TIME(int)
    isError: IS_ERROR(string: true, false)
    spanLayer: SPAN_LAYER(string: DB, RPC_FRAMEWORK, HTTP, MQ, CACHE)
    spanType: SPAN_TYPE(string: Exit, Entry, Local)
    componentId: COMPONENT_ID(int)
    tags:
    - {key: TAG_KEY(string), value: TAG_VALUE(string)}
    ...
    logs:
    - {key: LOG_KEY(string), value: LOG_VALUE(string)}
    ...
    peer: PEER(string)
    refs:
    - {
       traceId: TRACE_ID(string),
       parentTraceSegmentId: PARENT_TRACE_SEGMENT_ID(string),
       parentSpanId: PARENT_SPAN_ID(int),
       parentService: PARENT_SERVICE(string),
       parentServiceInstance: PARENT_SERVICE_INSTANCE(string),
       parentEndpoint: PARENT_ENDPOINT_NAME(string),
       networkAddress: NETWORK_ADDRESS(string),
       refType:  REF_TYPE(string: CrossProcess, CrossThread)
     }
   ...
```

| 属性 | 描述 
|--- |--- 
| operationName | 跨度操作名称。
| parentSpanId | 父跨度ID。 **注意**: 第一个跨度的父跨度ID应该为-1。
| spanId | 跨度ID. **注意**, 从0开始。 
| startTime | 跨度开始时间。 无法获得准确的时间，非0。
| endTime | 跨度结束时间。无法获得准确的时间，非0。
| isError | 跨度状态, true 或者 false. 
| componentId | 插件的组件ID。
| tags | 跨度的标签列表。 **注意**,保持与插件编码相同的顺序。
| logs | 跨度的日志列表。 **注意**,保持与插件编码相同的顺序。
| SpanLayer | 可选项, DB, RPC_FRAMEWORK, HTTP, MQ, CACHE.
| SpanType | 跨度类型, 可选项, Exit、Entry 或 Local.
| peer | 远端网络地址, 大部分情况下是IP +端口。对于Exit跨度，这是必需的。 

SegmentRef的验证描述

| 属性 | 描述 
|---- |---- 
| traceId | 链路ID
| parentTraceSegmentId | 父追踪段ID，指向父追踪段中的追踪段ID。
| parentSpanId | 父跨度ID, 指向父追踪段中跨度的跨度ID。
| parentService | 父/下游服务的名称。
| parentServiceInstance | 父/下游服务实例的名称。
| parentEndpoint |  父/下游服务的端点。
| networkAddress |父出口跨度的远端网络地址。
| refType |引用类型, 可选项, 跨进程 或 跨线程.

### startup.sh

该脚本提供了基于JVM的服务的起点，其中大多数以带有一些变量的`java -jar`开始。 Shell中提供了以下系统环境变量。

| 变量   | 描述    |
|:----     |:----        |
| agent_opts               |     选择代理插件，请检查插件文档中的详细信息或此PR中添加的相同选项。       |
| SCENARIO_NAME       | 服务名称。默认与用例文件夹名称相同   |
| SCENARIO_VERSION           | 版本 |
| SCENARIO_ENTRY_SERVICE             | 进入此服务的入口URL|
| SCENARIO_HEALTH_CHECK_URL          | 健康检查的URL  |


> `${agent_opts}` 必须添加到你的 `java -jar` 命令行, 其中包括测试框架注入的参数, 和
> 保证探针能被加载。 所有其他参数应在之后 `${agent_opts}`添加.

测试框架默认情况下会将服务名称设置为测试用例文件夹名称，但是在某些情况下，需要多个测试项目才能以不同的服务代码运行，可以像以下示例一样显式设置它。

示例
```bash
home="$(cd "$(dirname $0)"; pwd)"

java -jar ${agent_opts} "-Dskywalking.agent.service_name=jettyserver-scenario" ${home}/../libs/jettyserver-scenario.jar &
sleep 1

java -jar ${agent_opts} "-Dskywalking.agent.service_name=jettyclient-scenario"  ${home}/../libs/jettyclient-scenario.jar &

```

> 仅在确实需要时设置此选项或使用其他SkyWalking选项。

**以以下测试用例为例**
* [undertow](../../../test/plugin/scenarios/undertow-scenario/bin/startup.sh)
* [webflux](../../../test/plugin/scenarios/webflux-scenario/webflux-dist/bin/startup.sh)

## 最佳实践

### 如何使用原型创建测试用例项目

我们提供了原型和脚本，使创建项目更加容易。它创建了一个完整的测试用例项目。
这样我们只需要关注用例。首先，我们可以关注以下命令来获取有关脚本的用法。

`bash ${SKYWALKING_HOME}/test/plugin/generator.sh`

然后运行并在`./scenarios`中生成一个以`scenario_name`的项目。

### 推荐Pom

```xml
    <properties>
        <!-- 在pom中提供和使用此属性. -->
        <!-- 此版本应与库版本匹配, -->
        <!-- 在这种情况下，http组件库版本为4.3. -->
        <test.framework.version>4.3</test.framework.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.apache.httpcomponents</groupId>
            <artifactId>httpclient</artifactId>
            <version>${test.framework.version}</version>
        </dependency>
        ...
    </dependencies>

    <build>
        <!-- 将程序包的最终名称设置为与测试用例文件夹的大小写相同. -->
        <finalName>httpclient-4.3.x-scenario</finalName>
        ....
    </build>
```

### 如何实现心跳服务

心跳服务旨在检查服务可用状态。该服务是一个简单的HTTP服务，返回200表示目标服务已准备就绪。
然后，流量生成器将访问进入服务并验证预期数据。用户应考虑使用此服务来检测诸如相关服务是否已准备就绪，尤其是在相关服务是数据库或集群时。

请注意，由于可以完整或部分跟踪心跳服务，因此, `expectedData.yaml`中的`segmentSize`(追踪段数量)应当使用 `ge`作为操作符,
并且不要在预期的追踪段数据中包含心跳服务的追踪段。

### 编写预期数据的示例过程

预期数据文件, `expectedData.yaml`, 包含 `SegmentItems`.

我们正在使用HttpClient插件来展示如何写入期望的数据.

测试有两个要点

1. 是否创建HttpClient span。
2. ContextCarrier是否正确创建，并在进程之间传播。

```
+-------------+         +------------------+            +-------------------------+
|   Browser   |         |  Case Servlet    |            | ContextPropagateServlet |
|             |         |                  |            |                         |
+-----|-------+         +---------|--------+            +------------|------------+
      |                           |                                  |
      |                           |                                  |
      |       WebHttp            +-+                                 |
      +------------------------> |-|         HttpClient             +-+
      |                          |--------------------------------> |-|
      |                          |-|                                |-|
      |                          |-|                                |-|
      |                          |-| <--------------------------------|
      |                          |-|                                +-+
      | <--------------------------|                                 |
      |                          +-+                                 |
      |                           |                                  |
      |                           |                                  |
      |                           |                                  |
      |                           |                                  |
      +                           +                                  +
```

#### segmentItems

通过遵循HttpClient用例的流程，应该创建两个追踪段。

1. 一段代表 用例Servlet 访问。我们将其命名为`SegmentA`。
2. 一段代表ContextPropagateServlet访问。我们将其命名为`SegmentB`。

```yml
segmentItems:
  - serviceName: httpclient-case
    segmentSize: ge 2 # Could have more than one health check segments, because, the dependency is not standby.
```

由于Tomcat插件是SkyWalking的默认插件，因此在SegmentA中有两个跨度

1. Tomcat 入口跨度(entry span)
2. HttpClient 出口跨度(exit span)

SegmentA跨度列表应如下

```yml
    - segmentId: not null
      spans:
        - operationName: /httpclient-case/case/context-propagate
          parentSpanId: 0
          spanId: 1
          startTime: nq 0
          endTime: nq 0
          isError: false
          spanLayer: Http
          spanType: Exit
          componentId: eq 2
          tags:
            - {key: url, value: 'http://127.0.0.1:8080/httpclient-case/case/context-propagate'}
            - {key: http.method, value: GET}
          logs: []
          peer: 127.0.0.1:8080
        - operationName: /httpclient-case/case/httpclient
          parentSpanId: -1
          spanId: 0
          startTime: nq 0
          endTime: nq 0
          spanLayer: Http
          isError: false
          spanType: Entry
          componentId: 1
          tags:
            - {key: url, value: 'http://localhost:{SERVER_OUTPUT_PORT}/httpclient-case/case/httpclient'}
            - {key: http.method, value: GET}
          logs: []
          peer: null
```

SegmentB应该只有一个Tomcat跨度，但应包含指向SegmentA的Ref。

SegmentB跨度列表应如下

```yml
- segmentId: not null
  spans:
  -
   operationName: /httpclient-case/case/context-propagate
   parentSpanId: -1
   spanId: 0
   tags:
   - {key: url, value: 'http://127.0.0.1:8080/httpclient-case/case/context-propagate'}
   - {key: http.method, value: GET}
   logs: []
   startTime: nq 0
   endTime: nq 0
   spanLayer: Http
   isError: false
   spanType: Entry
   componentId: 1
   peer: null
   refs:
    - {parentEndpoint: /httpclient-case/case/httpclient, networkAddress: 'localhost:8080', refType: CrossProcess, parentSpanId: 1, parentTraceSegmentId: not null, parentServiceInstance: not null, parentService: not null, traceId: not null}
```

## 本地测试和提交

首先，测试用例项目可以成功编译，具有正确的项目结构并且可以部署。
开发人员应该测试启动脚本是否可以在Linux / MacOS中运行，并且 入口服务/健康检查服务 能够提供响应。

您可以使用以下命令运行测试

```bash
cd ${SKYWALKING_HOME}
bash ./test/plugin/run.sh -f ${scenario_name}
```

**注意**，如果 `./apm-sniffer` 中的代码被改变, 不管是因为您的更改还是git更新，
请重新编译 `skywalking-agent`模块。因为测试框架将使用现有的 `skywalking-agent` 文件夹,
而不是每次都重新编译它。

使用 `${SKYWALKING_HOME}/test/plugin/run.sh -h` 了解更多命令选项。

如果本地测试通过，则可以将其添加到 `.github/workflows/plugins-test.<n>.yaml` 文件, 它将驱动在官方SkyWalking存储库的Github Actions上运行的测试。
根据您的插件名称, 请按字母顺序将测试用例添加到`.github/workflows/plugins-test.<n>.yaml` 文件中。

每个测试用例都是一个Github Actions作业(job)。请使用 `<scenario name> + <version range> + (<supported version count>)`作为作业(job)的 `title`, 并且方案目录作为作业的`name`,
通常，您只需要确定要在文件（`plugins-test。<n> .yaml`）中添加测试用例，并简单地在其中添加一行（如下所示），以现有用例为示例。

```yaml
jobs:
  PluginsTest:
    name: Plugin
    runs-on: ubuntu-18.04
    timeout-minutes: 90
    strategy:
      fail-fast: true
      matrix:
        case:
          # ...
          - { name: '<your case name>', title: '<PluginName, i.e. Spring> (<Supported Version Count, i.e 12>)' } # <<== 按字母顺序插入一行
          # ...
```
