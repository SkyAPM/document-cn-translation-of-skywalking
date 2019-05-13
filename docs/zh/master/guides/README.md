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

以下所有渠道均向社区开放, 您可以选择自己喜欢的方式.

* 提交[问题](https://github.com/apache/skywalking/issues)
* 邮件列表: **dev@skywalking.apache.org**. 发送邮件到 `dev-subscribe@skywalking.apache.org`, 按照回复订阅邮件列表.
* [Gitter](https://gitter.im/openskywalking/Lobby)
* QQ 群: 392443393

## 对于代码开发者

作为开发者, 首先阅读[编译指南](How-to-build.md). 它会告诉你如何在本地构建 SkyWalking.

### 项目扩展

SkyWalking 提供了许多方法来对现有功能进行扩展. 如果你对这些方式感兴趣, 阅读以下指南.

- [Java agent 插件开发指南](Java-Plugin-Development-Guide.md). 本文将帮助您开发 SkyWalking Java agent 插件以支持更多框架或组件. 无论是进行开源插件或私有插件的开发都需要阅读这个指南.
- 如果您想开发其他语言的探针或组件, 请阅读[组件库定义和扩展](Component-library-settings.md)文档.
- [数据存储扩展开发指南](storage-extention.md). 除了官方已经支持的存储实现外, 可帮助潜在的贡献者实现新的数据存储的实现.
- [通过 oal 脚本自定义分析](write-oal.md). 指导您使用 oal 脚本来提供自己所需的指标.
- [为新指标提供源和域扩展](source-extension.md). 如果你想分析一个目前 SkyWalking 尚未提供的新的指标, 你就无法使用[既有的接收器](../setup/backend/backend-receivers.md), 
而要添加一个新的接收器. 此时你很可能需要添加新的源和域. 本文将引导你如何做到.
- [后端存储实体扩展](inventory-extension.md). 如果你想扩展 SkyWalking 存储实体, 并且想将上游修改贡献回 Apache 开源软件仓库(本仓库), 请阅读此文中的原则.

### UI 开发者

SkyWalking UI 由静态页面和 Web 容器构成.

- 自 6.1 版本开始, [RocketBot UI](https://github.com/apache/skywalking-rocketbot-ui) 已经成为 SkyWalking 的原生 UI. 其采用 VueJS + TypeScript 构建. 你可以在 RocketBot 代码仓库了解更多.
- **Web容器** 源代码在 `apm-webapp` 模块中. 这是一个简单的 zuul 代理, 用于托管静态资源并使用 GraphQL 向后端发送查询请求.
- [旧的 UI 代码仓库](https://github.com/apache/skywalking-ui) 依旧保留, 但从 6.0.0-GA 以后不会再被包含在 SkyWalking 发行版中.

## 发布

[Apache 发布指南](How-to-release.md)向提交者团队介绍了正式的 Apache 版本发布流程, 以避免破坏任何 Apache 规则. 如果您在重新分发中保留我们的许可和须知, 则 Apache 许可允许每个人重新分发.
