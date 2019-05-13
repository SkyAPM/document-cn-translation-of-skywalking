# 欢迎
**这是 SkyWalking 6 官方文档的中文翻译版, 由社区提供, 欢迎加入我们**

在这里你可以了解到所有有关 **SkyWalking** 的架构, 如何部署并使用 SkyWalking, 以及如何基于 SkyWalking 进行开发.

- [概念与设计](concepts-and-designs/README.md). 概念与设计解释了 SkyWalking 最重要的核心思想.
如果你想理解这些特性和可视化底层是如何运作的, 你可以从那里开始学习.
- [设置](setup/README.md). 包含了在不同场景下安装 SkyWalking 的教程. 作为一个平台, SkyWalking 提供了几种方式来提供观测能力.
包括监控以及报警. 
- [贡献指南](guides/README.md). 教程是针对 PMC, Committer 和新贡献者. 在这里你可以了解到如何从零进行贡献.
- [协议](protocols/README.md). 协议展示了 Agents/探针和后端的通讯方式. 对于上行遥测数据感兴趣的人, 绝对应该一读.
- [FAQs](FAQ/README.md). 包括了一个清单, 含有已知的设置问题, 如果你遇到什么问题, 请先在那里查找看是否已有解决方案.

除此之外, 你还可能对以下链接感兴趣.

- 最新的发行版本以及较老的发行版本都可以在 [Apache SkyWalking 发行页面](http://skywalking.apache.org/downloads/) 找到. 变更日志可以在 [此处](../CHANGES.md) 查看.
- 你可以在[社区资源目录](https://github.com/OpenSkywalking/Community)找到关于 SkyWalking 在大会上的演讲日程, 线上视频和文章.
如果你有其他与此相关的资源, 欢迎提交 PR.
- 我们一直在寻找优化文档和代码的帮助, 所以如果你发现了有什么问题, 不要犹豫, [给我们提 Issue](https://github.com/apache/skywalking/issues/new), 
或者, 通过 PR 把你的贡献提交给我们, 帮助我们把 SkyWalking 做得更好.

---

### 来自 5.x 版本的用户
SkyWalking 5.x 仍然由社区支持. 对于想从 5.x 升级到 6.x 的用户, 你应该要知道有些定义和概念已经改变了.

其中改变的最重要的两个概念是
1. (5.x 中的)应用现在已经改变为 **(6.x 中的)服务** 了, 同时, 应用实例也变为了 **服务实例**.
1. (5.x 中的)服务改变为 (6.x 中的) **端点(Endpoint)**.


