# 为新指标提供源和域

从 [OAL 域介绍](../concepts-and-designs/oal.md#scope)一文中你应该已经了解到什么是域了.
现在, 如果你想做更多扩展, 你需要深入了解什么是 **源**.

**源** 和 **域** 是相互绑定的概念. **域** 声明了唯一标志(id)和名称(name), **源** 声明了属性.
请按照以下步骤来新建源和域.

1. 在 OAP 核心模块中, 提供了 **SourceReceiver** 内部服务.

```java
public interface SourceReceiver extends Service {
    void receive(Source source);
}
```

2. 所有分析数据必须是 **org.apache.skywalking.oap.server.core.source.Source** 的子类,
并带有 `@SourceType` 注解, 且在 `org.apache.skywalking` 包下,
此时它就可以被 OAL 脚本和 OAP 内核支持了.

比如对于已经存在的源, **Service**.

```java
@ScopeDeclaration(id = SERVICE, name = "Service")
public class Service extends Source {
    @Override public int scope() {
        return DefaultScopeDefine.SERVICE;
    }

    @Override public String getEntityId() {
        return String.valueOf(id);
    }

    @Getter @Setter private int id;
    @Getter @Setter private String name;
    @Getter @Setter private String serviceInstanceName;
    @Getter @Setter private String endpointName;
    @Getter @Setter private int latency;
    @Getter @Setter private boolean status;
    @Getter @Setter private int responseCode;
    @Getter @Setter private RequestType type;
}
```

3. `Source#scope()` 方法返回了一个 ID, 这并非一个随意的值, 这个 ID 也必须在 `@ScopeDeclaration` 注解中声明.
对于同一个源, `@ScopeDeclaration` 和 `scope()` 方法中的 ID 必须一致.

4. 方法 `Source#getEntityId()` 返回代表和域相关的唯一实体的值.
比如说, 在这个 `Service` 域中, id 就是 Service id, 代表某个特定的服务, 如 **订单服务**.
该值在 [OAL 分组机制](../concepts-and-designs/oal.md#group) 中使用.

5. 将域名称作为关键字, 添加到 `OALLexer.g4` 语法定义文件中. `OALLexer.g4` 文件位于 `generate-tool-grammar` 模块下的 `antlr4` 文件夹下.

6. 将域名称作为关键字, 添加到 `OALParser.g4` 解析器定义文件中, 该文件与 `OALLexer.g4` 位于同一级文件夹.

7. 在文件夹 `generated-analysis/src/main/resources` 下的 `generator-scope-meta.yml` 文件中为新的域设置默认字段.
如果你想知道为什么需要这些字段, 你必须先了解所有既有的查询. 还有一种简单的方法就是, 仿照已经存在的域.
举个例子, 如果你想添加一个新的指标: 服务实例的连接数, 可以仿照现有的 `ServiceInstance`.

___
做完以上步骤之后, 你就可以构建一个接收器了, 通常你需要:
1. 获取指标的原始数据
1. 构建源, 发送到 `SourceReceiver`
1. 编写完整的 OAL 脚本
1. 重新打包项目
