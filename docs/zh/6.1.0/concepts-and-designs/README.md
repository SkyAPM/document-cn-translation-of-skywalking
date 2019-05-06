# 概念与设计

了解 SkyWalking 的概念与设计能够帮助大家从总体上了解 SkyWalking 以及进一步学习 SkyWalking. 

- 什么是 SkyWalking?
  - [核心概念与概览](overview.md) 一文对 SkyWalking 进行高层次的概述和介绍, 包括 SkyWalking 所要解决的问题.
  - [项目目标](project-goals.md) 一文阐述了 SkyWalking 所重点关注的目标和为此提供的特性.

阅读完以上文档后, 你应该对 SkyWalking 的基本目标有所了解. 接下来你可以选择以下文章中你感兴趣的, 深入阅读.

- 探针
  - [简介](probe-introduction.md) 一文将引导读者理解什么是探针, 目前有多少已支持的探针以及为什么我们需要它们.
  - [服务自动打点代理工具](service-agent.md) 介绍了自动打点代理工具具体做了什么, 以及目前 SkyWalking 已经支持自动打点的编程语言.
  - [手动打点 SDK](manual-sdk.md) 介绍了手动打点 SDK 在 SkyWalking 生态中所扮演的角色.
  - [服务网格(Service Mesh)探针](service-mesh-probe.md) 介绍了 SkyWalking 为何需要以及如何从服务网格和代理探针接收遥测数据的.
- 后端
  - [总览](backend-overview.md) 一文从高层次对观测分析协议(Observability Analysis Protocol, OAP)后端进行介绍.
  - [观测分析语言(Observability Analysis Language)](oal.md) 介绍了一些用于聚合行为定义的核心语言.
  - [OAP 中的查询](../protocols/README.md#query-protocol) 基于观测分析语言的指标定义, 涵盖了一系列查询协议.
- 用户界面(UI)
  - [总览](ui-overview.md) 对 SkyWalking 的用户界面进行简短介绍.
