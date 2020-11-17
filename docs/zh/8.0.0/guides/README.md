# 贡献指南
你可以通过以下方式, 为 SkyWalking 社区做出贡献.

- 通览 SkyWalking 的文档, 指出或修复文档不准确的地方, 也可以将 SkyWalking 文档翻译成其他语言.
- 下载我们的[发行版本](http://skywalking.apache.org/downloads/), 尝试用其监控你的应用程序, 并向我们反馈您的想法, 疑问或使用案例.
- 阅读我们的源码, 如果对于细节有疑问, 向我们提问.
- 寻找代码中的 bug, 可在[这里](https://github.com/apache/skywalking/issues)提交问题, 并且您也可以尝试修复它.
- 从标记为[需要帮助的问题列表](https://github.com/apache/skywalking/issues?q=is%3Aopen+is%3Aissue+label%3A%22help+wanted%22)中入手开始你的贡献.
- 在 [GitHub 的问题列表](https://github.com/apache/skywalking/issues/new)提交问题或开启一个讨论.
- 通过[网页邮件列表](https://lists.apache.org/list.html?dev@skywalking.apache.org)查看所有邮件的讨论, 如果您是 SkyWalking 项目的 committer, 可在浏览器模式下登录并使用邮件列表. 否则, 按照下面指南进行订阅.
- 问题的报告和讨论也可在 `dev@skywalking.apache.org` 邮件列表中进行.
发送任意内容邮件到 `dev-subscribe@skywalking.apache.org`, 按照回复订阅邮件列表.

## 与我们联系
下所有渠道均向社区开放, 您可以选择自己喜欢的方式.

* 提交[问题](https://github.com/apache/skywalking/issues)
* 邮件列表: **dev@skywalking.apache.org**. 发送邮件到 `dev-subscribe@skywalking.apache.org`, 按照回复订阅邮件列表.
* [Gitter](https://gitter.im/openskywalking/Lobby)
* QQ 群: 392443393


## 成为正式的Apache SkyWalking提交者
PMC将评估每个贡献者的贡献，包括但不限于：代码贡献，并遵循Apache指南来提名、投票和邀请新的提交者和PMC成员。参考 [成为正式的Apache SkyWalking提交者](asf/committer.md) 获得更多详细内容。

## 对于代码开发者
对于代码开发者，首先阅读[编译指南](How-to-build.md)，它会告诉你如何在本地构建 SkyWalking。

### 集成测试
在搭建好本地开发环境并且写完代码之后, 为了能够让你的代码更快更容易被 SkyWalking 合并, 你需要在本地先运行单元测试 (UT), 确保你的代码没有破坏任何已有的功能, 你还需要写一些单元测试代码来验证新代码是否能正常运行, 同时也可以避免你的代码被以后的贡献者破坏.如果新增的代码涉及到第三方组件或库, 你还需要编写集成测试 (IT).

SkyWalking 利用插件 `maven-surefire-plugin` 来运行单元测试, 使用插件 `maven-failsafe-plugin` 来运行集成测试。插件 `maven-surefire-plugin` 运行时将会把集成测试 (类名以 `IT` 开头的类) 排除, 留给插件 `maven-failsafe-plugin` 去运行，`maven-failsafe-plugin` 被绑定到 maven 的 `CI-with-IT` profile, `verify` 执行目标上. 因此, 如果你要运行单元测试，使用 `./mvnw clean test`, 这只会运行单元测试, 不会运行集成测试.

如果你想运行集成测试, 请确保激活 `CI-with-IT` profile 以及你要运行集成测试的模块的 profile。举个例子, 如果你想运行 `oap-server` 模块的集成测试，使用 `./mvnw -Pbackend,CI-with-IT clean verify`，如果你想运行所有集成测试，只需要运行`./mvnw -Pall,CI-with-IT clean verify`.

请注意，如果你要编写集成测试，命名时请使用 `IT*` 开头，使得只在 `CI-with-IT` profile 下才会运行。

### 端到端测试
从6.3.0版本开始，我们已经引入了更多的自动测试来执行软件质量保证，E2E是最重要的部分之一。

> 端到端测试是一种用于测试应用程序流程从头到尾是否按设计执行的方法。执行端到端测试的目的是识别系统依赖关系，并确保在各种系统组件和系统之间传递正确的信息。

e2e测试涉及部分或全部的OAP服务器、存储、协调器、webapp和仪表化服务，所有这些都是由`docker-compose`编排的，此外，有一个测试控制器(JUnit测试)运行在容器的外部，它将流量发送到服务，然后通过 SkyWalking Web 应用程序的 GraphQL API 验证请求完成后的相应结果。

#### 编写端到端测试用例

- IntelliJ IDEA 设置环境

e2e测试是 SkyWalking 根目录下的一个独立项目，默认情况下，IDEA无法识别它。因此你需要右键点击
文件`test/e2e/pom.xml`，然后点击`Add as Maven Project`。但我们建议在一个单独的IDE窗口打开`skywalking/test/e2e`目录以获得更好的体验，因为可能会有阴影类问题。

- 编排组件

我们E2E测试的目标是以单节点模式和集群模式测试整个 SkyWalking 项目，包括OAP服务器、存储、协调器、webapp，甚至前端UI(不是现在)。因此，第一步是确定我们要验证和编排什么组件。

为了使它更容易编排，我们使用了[docker-compose](https://docs.docker.com/compose/)，它提供了一个方便的文件格式(`docker-compose.yml`)
编排所需的容器，并为我们提供了定义组件依赖关系的可能性。

测试前需要考虑内容:
1. 决定需要什么(和多少)容器，例如对于集群测试，你至少需要2个OAP节点，协调器像zookeeper，存储ElasticSearch和工具化的服务;
2. 在`docker-compose`中定义容器，并仔细指定依赖关系、启动顺序，最重要的是将它们链接在一起，例如在代理端设置正确的OAP地址，在OAP中设置正确的协调器地址等。
3. 编写(或最好重用)测试代码，以验证结果是正确的。

至于最后一步，我们有一个友好的框架来帮助您更快地开始，它提供了注释`@DockerCompose("docker-compose.yml")`来加载或解析并以适当的顺序启动所有容器，`@ContainerHost / @ContainerPort`获取容器的实际主机或端口，`@ContainerHostAndPort`获取两者，`@DockerContainer`获取运行的容器。

- 编写collector测试

简单地说，测试collector基本上是可以绑定到Maven“集成-测试/验证”阶段的测试。
它们向仪表化的服务发送**设计的**请求，并期望从SkyWalking webapp GraphQL API获得相应的跟踪/度量/元数据。

在测试框架中，我们提供了一个`TrafficController`定期发送流量数据到工具服务，您可以简单地通过提供一个url和流量数据启用它，参考 `../../../test/e2e/e2e-test/src/test/java/org/apache/skywalking/e2e/base/TrafficController.java`。

- 故障排除

我们以非CI(本地)模式将所有容器中的所有日志暴露给标准输出，但是将它们全部保存上传到GitHub服务器，您可以在“Artifacts/ download Artifacts/ logs”按钮中下载它们(仅在测试失败时)以便调试。

**注意:**请先在本地验证新添加的E2E测试用例，如果您发现它在本地通过但PR检查状态失败，请确保所有更新或新添加的文件(特别是子模块中的文件)都已经提交并包含在PR中，或将git HEAD重置到远程并在本地再次验证。


### 项目扩展
SkyWalking 提供了许多方法来对现有功能进行扩展。如果你对这些方式感兴趣，阅读以下指南。

- [Java agent 插件开发指南](Java-Plugin-Development-Guide.md)。本文将帮助您开发 SkyWalking Java agent 插件以支持更多框架或组件。无论是进行开源插件或私有插件的开发都需要阅读这个指南。
- 如果您想开发其他语言的探针或组件，请阅读[组件库定义和扩展](Component-library-settings.md)文档。
- [数据存储扩展开发指南](storage-extention.md)。除了官方已经支持的存储实现外，可帮助潜在的贡献者实现新的数据存储的实现。
- 定制分析的oal脚本。OAL脚本位于`config/oal/*.oal`。您可以更改它并重新启动OAP服务器。如果你想了解更多关于OAL脚本的内容，请查阅[观测分析语言介绍](../concepts-and-designs/oal.md)。
- [为新指标提供源和域扩展](source-extension.md)。如果你想分析一个目前 SkyWalking 尚未提供的新的指标，你就无法使用[既有的接收器](../setup/backend/backend-receivers.md)，而要添加一个新的接收器。此时你很可能需要添加新的源和域。本文将教你如何做。
- [后端存储实体扩展](inventory-extension.md)。如果你想扩展 SkyWalking 存储实体，并且想将上游修改贡献回 Apache 开源软件仓库(本仓库)，请阅读此文中的原则。


### UI 开发者
SkyWalking UI 由静态页面和 Web 容器构成。

- 自 6.1 版本开始, [RocketBot UI](https://github.com/apache/skywalking-rocketbot-ui) 已经成为 SkyWalking 的原生 UI，其采用 Vue + TypeScript 构建。你可以在 RocketBot 代码仓库了解更多。
- **Web容器** 源代码在 `apm-webapp` 模块中，这是一个简单的 zuul 代理，用于托管静态资源并使用 GraphQL 向后端发送查询请求。
- [旧的 UI 代码仓库](https://github.com/apache/skywalking-ui)依旧保留，但从 6.0.0-GA 以后不会再被包含在 SkyWalking 发行版中。

### OAP后端依赖管理
> 本节仅适用于后端模块的依赖关系

作为Apache软件基金会(ASF)的顶级项目之一, SkyWalking 应该遵循[ASF第三方许可策略](https://apache.org/legal/resolved.html)。因此，如果要向项目中添加新的依赖项，则需要检查新添加的依赖项是否会破坏策略，并将他们的许可证和 NOTICE 添加到项目中。

我们有一个[简单的脚本](../../../tools/dependencies/check-LICENSE.sh)来帮助你确保没有错过任何新添加的依赖:

- 构建一个分发包，将它解压到文件夹`dist`。
- 在根目录下运行脚本，它将打印出所有新添加的依赖项。
- 检查这些依赖关系的许可证和通知，如果它们可以包含在ASF项目中，将它们添加到`apm-dist/release-docs/{LICENSE,NOTICE}`文件中。
- 将这些依赖项的名称添加到`tools/dependencies/known-oap-backend-dependencies.txt`文件中(**按照字母顺序**)，接下来运行`check-LICENSE.sh`应该会通过。

## 剖析
性能剖析是 APM 系统中的一个增强特性。我们使用线程转储来估计方法执行时间, 而不是添加许多本地span。这种方法的资源消耗比采用分布式跟踪的慢速定位方法要少得多。该特性适用于生产环境。以下文档对于开发人员理解此特性的关键部分非常重要：

- [Profile data report procotol](https://github.com/apache/skywalking-data-collect-protocol/tree/master/profile) 像其他跟踪、JVM数据一样通过gRPC提供。
- [Thread dump merging mechanism](backend-profile.md) 引入了合并机制，帮助最终用户理解剖析报告。
- [Exporter tool of profile raw data](backend-profile-export.md) 介绍了当界面有异常问题时，如何打包原始的剖析数据发送给社区排查。

## 发布
[Apache发布指南](How-to-release.md)向提交者团队介绍了如何进行正式的Apache版本发布，以避免违反任何Apache规则。如果你保留我们的许可证和NOTICE，则Apache许可证允许所有人重新分发。