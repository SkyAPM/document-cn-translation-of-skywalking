### 现象
Kafka消费端trace断裂

### 原因
Kafka探针只是追踪了对Kafka的拉取动作，而整个后续处理过程不是由kafka consumer发起。所以需要在消费处理的发起点，进行手动埋点。

### 解决方法
 可以通过Application Toolkit中的 `@Trace` 标注，或者OpenTracing API进行手动埋点。
