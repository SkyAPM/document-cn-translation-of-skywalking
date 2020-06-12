# V8 版本升级

SkyWalking v8 版本开始使用 [v3 协议](../protocols/README.md)，所以，不兼容上个发行版本。打算在 v8 系列版本中升级的用户可以遵循本指南。

SkyWalking v8 begins to use [v3 protocol](../protocols/README.md), so, it is incompatible with previous releases.
Users who intend to upgrade in v8 series releases could follow this guidance.

为了更好的可扩展性，v8 版本中移除了 v6 和 v7 的注册，请根据以下方式升级。

1. 使用不同的存储或者一个新的 namespace 。此外，还可以考虑删除所有与 SkyWalking 相关的存储索引/表。
2. 部署整个集群，并公开一个新的网络地址。
3. 如果您正在使用语言代理，也要升级新的代理，与此同时，确保代理支持不同的语言。

并为新的 SkyWalking OAP 集群设置后端地址。