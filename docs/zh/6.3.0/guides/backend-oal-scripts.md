# 官方 OAL 脚本

在阅读本文档前请先阅读 [OAL 简介](../concepts-and-designs/oal.md).

在 SkyWalking 发行版压缩包中, `generated-analysis-x.y.z.jar/official_analysis.oal` 文件是
官方的 OAL 脚本, 在源代码仓库中也有文件 [official_analysis.oal](../../../oap-server/generated-analysis/src/main/resources/official_analysis.oal).

**注意**: 尽管此文件包含在发行版压缩包中, 但是它并不会影响运行时的任何东西.
你需要使用 OAL 代码生成工具来从这个文件中构建出真实的分析代码.
所有生成的代码都位于 **oap-server/generated-analysis/target/generated-sources** 下的 `oal` 文件夹.

所有在这个脚本中使用到的度量指标名称都可以在报警和界面查询中使用. 当然, 你也可以修改此脚本并重新生成
分析流程和度量指标, 比如添加过滤条件.

如果你想尝试添加或删除度量指标, 用户界面可能会遭到破坏, 我们建议只有当你想自己构建一个用户界面的时候,
才做这些修改.
