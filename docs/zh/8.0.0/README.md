# 欢迎

**这是 SkyWalking 8 官方文档的中文翻译版, 由社区提供, 欢迎加入我们.**

在这里你可以了解到所有有关 **SkyWalking** 的架构, 如何部署并使用 SkyWalking, 以及如何基于 Skywalking 贡献指南进行开发。

**NOTICE, SkyWalking 8 uses brand new tracing APIs, it is incompatible with all previous releases.**

- [概念与设计](concepts-and-designs/README.md). 你将会发现很多关于 Skywalking 的重要核心想法。 如果你想理解这些炫酷的特性和可视化底层是如何运作的, 你可以从那里开始学习.

- [安装](setup/README.md). 指导不同场景下 Skywalking 的安装。 作为一个平台，提供了提供了几种观测方式，包括监控和告警。

- [UI 介绍](ui/README.md). 介绍 UI 如何使用及特性。

- [贡献指南](guides/README.md). 指南适用于 PMC, Committer 和新贡献者. 在这里你可以了解到如何开始贡献。

- [协议](protocols/README.md). 协议展示了代理（探针）与后端的通讯方式. 对于上行遥测数据感兴趣的人, 绝对应该一读.

- [问题集](FAQ/README.md). 整理了已知的安装问题和二次开发尝试。如果你遇到什么问题, 请先在那里查找看是否已有解决方案.

除此之外, 你还可能对以下链接感兴趣:

- 最新的发行版本以及较老的发行版本都可以在 [Apache SkyWalking 发行页面](http://skywalking.apache.org/downloads/) 找到. 变更日志可以在 [此处](../CHANGES.md) 查看。

- [SkyWalking WIKI](https://cwiki.apache.org/confluence/display/SKYWALKING/Home) WIKI 中放了一些变更和事件。

- SkyWalking 模块 [调用流](https://sourcespy.com/github/skywalking/xx-omcalls-.html) 和 [层次结构](https://sourcespy.com/github/skywalking/xx-omhierarchy-.html) 的最新概述，包括单独分析每个模块的能力。

- 你可以在[社区资源目录](https://github.com/OpenSkywalking/Community)找到关于 SkyWalking 在大会上的演讲日程, 线上视频和文章。

我们一直在寻找优化文档和代码的帮助, 所以如果你发现了有什么问题, 不要犹豫, [给我们提 Issue](https://github.com/apache/skywalking/issues/new),
或者, 通过 PR 把你的贡献提交给我们, 帮助我们把 SkyWalking 做得更好。

___

## 文档目录

如果你已经熟悉 SkyWalking, 你可以使用这个目录直接找到相关的章节。

- [概念与设计](concepts-and-designs/README.md)
  - SkyWalking 是什么?
    - [核心概念纵览](concepts-and-designs/overview.md). 提供高层次的描述和介绍，包括项目解决的问题。
    - [项目目标](concepts-and-designs/project-goals.md). 阐述了 SkyWalking 所重点关注的目标和为此提供的特性.
  - 探针
    - [介绍](concepts-and-designs/probe-introduction.md). 引导读者理解探针是什么, 存在多少种不同的探测以及为什么需要它们。
    - [服务自动打点代理工具](concepts-and-designs/service-agent.md). 介绍自动 instrument 代理程序的功能以及 SkyWalking 已经支持哪些语言。
    - [手动打点 SDK](concepts-and-designs/manual-sdk.md). 介绍了手动打点 SDK 在 SkyWalking 生态系统中的作用。
    - [服务网格探针](concepts-and-designs/service-mesh-probe.md). 介绍了如何从服务网格和代理探针接收遥测数据。
  - 后台
    - [总览](concepts-and-designs/backend-overview.md). 提供了有关 OAP 后台的高层次的介绍。
    - [可观测性分析语言](concepts-and-designs/oal.md). 介绍了为聚合行为定义而设计的核心语言。
    - [OAP 中使用查询](protocols/README.md#query-protocol). 提供了一套基于可观察性分析语言指标定义的查询协议。
  - UI
    - [总览](concepts-and-designs/ui-overview.md).  对 SkyWalking 的用户界面进行简短介绍。
  - CLI (命令行客户端)
    - SkyWalking CLI 提供了一套提供与 Skywalking 后台交互的命令行接口（通过 GraphQL），更多信息，[查看此处](https://github.com/apache/skywalking-cli).
- [安装](setup/README.md).
  - 后端、UI、Java agent 和 CLI 都是 Apache 官方发行，你可以在[Apache SkyWalking 下载页](http://skywalking.apache.org/downloads/)找到它们。
  - 不同语言服务代理
    - [Java 代理](setup/service-agent/java-agent/README.md). 介绍了如何将 java agent 安装到你的服务中，无需修改任何代码。
      - [支持中间件、框架、第三方依赖](setup/service-agent/java-agent/Supported-list.md).
      - [代理配置属性](setup/service-agent/java-agent/README.md#table-of-agent-configuration-properties).
      - [可选插件](setup/service-agent/java-agent/README.md#optional-plugins).
      - [Bootstrap/JVM class plugin](setup/service-agent/java-agent/README.md#bootstrap-class-plugins).
      - [高级特性](setup/service-agent/java-agent/README.md#advanced-features).
      - [插件开发指南](setup/service-agent/java-agent/README.md#plugin-development-guide).
      - [代理插件测试和性能测试](setup/service-agent/java-agent/README.md#test).
    - [其他语言代理](setup/README.md#language-agents-in-service) 包括 Nginx LUA, Python, .NetCore, PHP, NodeJS, Go, 这些都有志愿者维护.
  - 服务网格
    - [SkyWalking on Istio](setup/istio/README.md). 介绍如何使用 Istio Mixer bypass Adapter 与 SkyWalking 一起工作。
    - 使用 [ALS (access log service)](https://www.envoyproxy.io/docs/envoy/latest/api-v2/service/accesslog/v2/als.proto) 观察服务网格, 无需 Mixer. 跟踪查看 [文档](setup/envoy/als_setting.md)。
  - [后台, UI and CLI 安装文档](setup/backend/backend-ui-setup.md).
    - [后台安装文档](setup/backend/backend-setup.md).
      - [Overriding settings](setup/backend/backend-setting-override.md) 支持重写 application.yml 中的设置。
      - [IP and port setting](setup/backend/backend-ip-port.md)。介绍如何设置和使用 IP 和端口。
      - [Backend init mode startup](setup/backend/backend-init-mode.md). 如何初始化环境并优雅地退出. 在尝试初始化新群集之前，请阅读此内容。
      - [Cluster management](setup/backend/backend-cluster.md). 指导你将 backend 服务器设置为群集模式。
      - [Deploy in kubernetes](setup/backend/backend-k8s.md). 指导你构建和使用 SkyWalking 镜像以及如何部署到 K8S 中。
      - [Choose storage](setup/backend/backend-storage.md). 如我们所知，在默认的快速启动中，backend 使用 h2 数据库运行。但显然，它不符合产品环境。在这里，你可以找到其他的选择。选择一个你想要的，我们也环境所有人新的存储实现方式。
      - [Set receivers](setup/backend/backend-receivers.md). 你可以根据你的要求选择接收器，大多数接收器是无害的，至少我们的默认接收器是无害的。您可以设置并激活所有提供的接收器。
      - [Open fetchers](en/setup/backend/backend-fetcher.md). 你可以打开不同的 fetcher 从目标应用读取度量指标。这些像接收器一样工作，但是是拉模式，典型的例如 prometheus。
      - 在后台设置 [链路跟踪采样率](setup/backend/trace-sampling.md) . 链路采样允许你保持度量的准确性，同时仅根据速率在存储中保留一些跟踪。
      - 根据Follow [慢数据库状态临界值slow DB statement threshold](setup/backend/slow-db-statement.md) 配置文档，理解如何在你的系统中检测慢数据库状态（包括慢 SQL 状态）。
      - 官方 [OAL 脚本](guides/backend-oal-scripts.md). 正如您从我们的 [OAL 介绍](concepts-and-designs/oal.md)中所知, 大多数后台分析能力都基于脚本。这是对官方脚本的描述，这有助于你理解正在处理的度量数据，也可以用于报警。
      - [Alarm](setup/backend/backend-alarm.md). 告警提供基于时间序列的检查机制。你可以设置针对分析 OAL 度量对象的告警规则。
      - [Advanced deployment options](setup/backend/advanced-deployment.md). 如果你需要部署的后台有非常大的扩展性并支持高负载，你需要看看这个。
      - [Metrics exporter](setup/backend/metrics-exporter.md). 使用 Metrics exporter 将 Metrics 数据转发给第三方系统。
      - [Time To Live (TTL)](setup/backend/ttl.md). Metrics 和 trace 是时间序列数据, 因此将会被永久存储, 你可以针对各个维度设置过期时间.
      - [Dynamic Configuration](setup/backend/dynamic-config.md). 使得 OAP 中配置可以被动态修改，从远程服务或者第三方配置管理系统。
      - [Uninstrumented Gateways](setup/backend/uninstrumented-gateways.md). 配置 Skywalking agent 不支持的网关/代理，并在拓扑图上反应代理。
    - [UI 安装文档](setup/backend/ui-setup.md).
    - [CLI 安装文档](https://github.com/apache/skywalking-cli).
- [UI 介绍](ui/README.md). 介绍 UI 如何使用及特性。
- [贡献指南](guides/README.md). 指南适用于 PMC, Committer 和新贡献者. 在这里你可以了解到如何开始贡献。
  - [联系我们](guides/README.md#contact-us). 引导用户了解如何联系官方 committer 团队或者与项目社区沟通交流。
  - [如何成为官方 Apache SkyWalking Committer](guides/asf/committer.md). 如何成为官方 committer or PMC 成员.
  - [编译指南](guides/How-to-build.md). 根据这个文档使用源码编译整个项目。
  - [Agent 插件开发指南](guides/Java-Plugin-Development-Guide.md). 引导开发者编写他们的插件，当你向上有 push 插件的时候，根据 [插件测试需求](guides/Plugin-test.md)进行测试。
- [协议](protocols/README.md). 协议显示了代理/探测器和后端之间的通信方式。任何对上行遥测数据感兴趣的人都应该阅读这篇文章。
- [问题集](FAQ/README.md). 整理了已知的安装问题和二次开发尝试。如果你遇到什么问题, 请先在那里查找看是否已有解决方案。
