# 新度量指标的源和范围扩展

从 [OAL 域介绍](../concepts-and-designs/oal.md#scope), 中你应该已经了解到什么是域了.现在, 如果你想做更多扩展, 你需要深入了解什么是 **源**.

**源** 和 **域** 是相互绑定的概念. **域** 声明了唯一标志(id)和名称(name), **源** 声明了属性.请按照以下步骤来新建源和域.

1. 在 OAP 核心模块中, 提供了 **SourceReceiver** 内部服务.

```java
public interface SourceReceiver extends Service {
    void receive(Source source);
}
```

2. 所有分析数据必须是 **org.apache.skywalking.oap.server.core.source.Source** 的子类, 带有 `@SourceType` 注解且在 `org.apache.skywalking` 包下, 此时它就可以被 OAL 脚本和 OAP 内核支持了

比如对于已经存在的源, **Service**.

```java
@ScopeDeclaration(id = SERVICE_INSTANCE, name = "ServiceInstance", catalog = SERVICE_INSTANCE_CATALOG_NAME)
@ScopeDefaultColumn.VirtualColumnDefinition(fieldName = "entityId", columnName = "entity_id", isID = true, type = String.class)
public class ServiceInstance extends Source {
    @Override public int scope() {
        return DefaultScopeDefine.SERVICE_INSTANCE;
    }

    @Override public String getEntityId() {
        return String.valueOf(id);
    }

    @Getter @Setter private int id;
    @Getter @Setter @ScopeDefaultColumn.DefinedByField(columnName = "service_id") private int serviceId;
    @Getter @Setter private String name;
    @Getter @Setter private String serviceName;
    @Getter @Setter private String endpointName;
    @Getter @Setter private int latency;
    @Getter @Setter private boolean status;
    @Getter @Setter private int responseCode;
    @Getter @Setter private RequestType type;
}
```

3. 源中的 `scope()` 方法返回了一个 ID, 这并非一个随意的值, 这个 ID 也必须在 `@ScopeDeclaration` 注解中声明. 对于同一个源, `@ScopeDeclaration` 和 `scope()` 方法中的 ID 必须一致

4. 源中的 `String getEntityId()` 方法，请求返回代表和域相关的唯一实体的值. 比如说, 在这个 `Service` 域中, id 就是 Service id, 代表某个特定的服务, 如 `Order` 服务. 该值在 [OAL 分组机制](../concepts-and-designs/oal.md#group) 中使用。

5. `@ScopeDefaultColumn.VirtualColumnDefinition` 和 `@ScopeDefaultColumn.DefinedByField` 是必须的, 所有声明的字段(virtual/byField)将被推入持久实体, 匹配例如 ElasticSearch 索引和数据库表的列。例如，主要包括实体 id，以及端点和服务实例级范围的服务 id。引用所有现有的作用域。
所有这些字段都是由 OAL 运行时检测到的，以及在查询阶段是必需的。

6. 将域名称作为关键字, 添加到 `OALLexer.g4` 语法定义文件中. `OALLexer.g4` 文件位于 `generate-tool-grammar` 模块下的 `antlr4` 文件夹下。

7. 将域名称作为关键字, 添加到 `OALParser.g4` 解析器定义文件中, 该文件与 `OALLexer.g4` 位于同一级文件夹

___
做完以上步骤之后, 你就可以构建一个接收器了, 通常你需要

1. 获取指标的原始数据
2. 构建源, 发送到 `SourceReceiver`
3. 编写完整的 OAL 脚本
4. 重新打包项目
