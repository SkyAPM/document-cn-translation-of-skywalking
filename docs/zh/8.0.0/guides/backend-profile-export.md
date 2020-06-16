#概要文件原始数据的导出工具
当可视化不能通过官方UI很好地工作时，用户可以提交问题报告. 该工具帮助用户打包原始概要数据，以帮助社区在用户案例中定位问题. 
注意，这个报告包括类名、方法名、行号等。在提交之前，请确保这不会导致您的系统泄露.
## 使用命令行导出
1. 通过遵循您的用例来创建 `tools/profile-exporter/application.yml` 文件，并设置存储.
2. 准备资料
    - 概要文件任务 ID: 概要文件任务 ID
    - 追踪 ID: 概要文件错误追踪 ID
    - 导出目录: 数据导出的目录
3. 进入 Skywalking 根路径
4. 执行 shell 命令
    ```bash
   bash tools/profile-exporter/profile_exporter.sh --taskid={profileTaskId} --traceid={traceId} {exportDir}
   ```
5. 执行 shell 后将生成 `{traceId}.tar.gz` 文件.

## 导出的数据内容
1. `basic.yml`: 包含追踪中已分析段的完整信息.
2. `snapshot.data`: 当前段中所有监控线程的快照数据. 

## 上报概要文件的问题
1. 提供从该工具生成的导出数据.
2. 提供span操作名，分析模式(包含或不包含子节点).
3. 问题描述. (如果有UI截屏，效果会更好)
