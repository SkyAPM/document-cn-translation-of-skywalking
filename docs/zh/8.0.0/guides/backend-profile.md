# 线程转储归并机制
性能概要是APM系统中的一个增强特性. 我们使用线程转储来估算方法执行时间，而不是添加许多局部span。
这样，与使用分布式跟踪来定位慢方法相比，资源成本要低得多。该特性适用于生产环境。本文档介绍如何将线程转储作为堆栈树归并到最终报告中.
## 线程分析
### 读取数据并进行转换
从数据库中读取数据并将其转换为gRPC中的数据结构.
```
st=>start: Start
e=>end: End
op1=>operation: Load data using paging
op2=>operation: Transform data using parallel

st(right)->op1(right)->op2
op2(right)->e
```
复制代码并将其粘贴到此 [链接](http://flowchart.js.org/)以生成流程图.
1. 使用流按页读取数据(每页50条记录).
2. 以并行流的形式将数据转换为gRPC数据结构.
3. 合并到数据列表中.
### 数据分析
使用Java并行流中的group by和collector模式根据数据库记录中的第一个堆栈元素进行分组，
并使用collector执行数据聚合. 生成一个多根树.
```
st=>start: Start
e=>end: End
op1=>operation: Group by first stack element
sup=>operation: Generate empty stack tree
acc=>operation: Accumulator data to stack tree
com=>operation: Combine stack trees
fin=>operation: Calculate durations and build result

st(right)->op1->sup(right)->acc
acc(right)->com(right)->fin->e
```
复制代码并将其粘贴到此 [链接](http://flowchart.js.org/)以生成流程图.
- **按第一个堆栈元素分组**:使用每个堆栈中的第一级元素分组，确保堆栈具有相同的根节点e.
- **生成空栈**:生成多个顶级空树，为下一步做准备，
  生成多个顶级树的原因是可以并行地添加原始数据，而不需要生成锁.
- **累加数据至栈中**: 将每个线程转储添加到生成的树中.
    1. 遍历线程转储中的每个元素，以查找父元素中是否有具有相同代码签名和相同堆栈深度的子元素. 
    如果不是，则添加此元素.
    2. 保留原节点中的转储序列和时间戳.
- **合并所有堆栈**: 使用与 `累加器` 相同的规则将所有树结构组合成一个树结构.
    1.使用LDR遍历树节点。使用 `堆栈` 数据结构来避免递归调用，每个堆栈元素表示需要合并的节点.
    2. 合并两个节点的方法是合并其子节点的列表. 如果它们具有相同的代码签名和相同的父节点，则在此节点中保存转储序列和时间戳. 否则，需要将节点作为新的子节点添加到目标节点中.
- **计算时间并生成结果**: 计算相关统计数据并生成响应.
    1. 使用与 `合并所有堆栈` 步骤相同的遍历节点逻辑. 转换为GraphQL数据结构，并将所有节点放到一个列表中，以用于后续的持续时间计算.
    2. 并行计算每个节点的持续时间. 对于每个节点，对序列进行排序，如果有两个连续序列，持续时间应该添加这两个seq的时间戳的持续时间.
    3. 并行计算每个节点的执行. 对于每个节点，当前节点的持续时间应该减去所有子节点消耗的时间.

## 配置文件数据调试
请按照 [导出工具](backend-profile-export.md#export-command-line-usage) 打包配置文件数据. 解压配置文件数据并使用 [分析主要功能](../../../oap-server/server-tools/profile-exporter/tool-profile-snapshot-bootstrap/src/test/java/org/apache/skywalking/oap/server/tool/profile/exporter/ProfileExportedAnalyze.java) 运行.