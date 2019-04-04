# 插件开发指南

本文档描述了如何理解, 开发和贡献插件.

## 概念

### Span

Span 是分布式跟踪系统中一个重要且常用的概念. 可从 [Google Dapper Paper](https://research.google.com/pubs/pub36356.html) 和 [OpenTracing](http://opentracing.io) 学习更多与 **Span** 相关的知识.

SkyWalking 从 2017 年开始支持 OpenTracing 和 OpenTracing-Java API, 我们的 Span 概念与论文和 OpenTracing 类似. 我们也扩展了 Span.

Span 有三种类型

1.1 EntrySpan

EntrySpan 代表服务提供者, 也是服务器端的端点. 作为一个 APM 系统, 我们的目标是应用服务器. 所以几乎所有的服务和 MQ-消费者 都是 EntrySpan.

1.2 LocalSpan

LocalSpan 表示普通的 Java 方法, 它与远程服务无关, 也不是 MQ 生产者/消费者, 也不是服务（例如 HTTP 服务）提供者/消费者.

1.3 ExitSpan

ExitSpan 代表一个服务客户端或MQ的生产者, 在 SkyWalking 的早期命名为 `LeafSpan`. 例如 通过 JDBC 访问DB, 读取 Redis/Memcached 被归类为 ExitSpan.

### 上下文载体 (ContextCarrier)

为了实现分布式跟踪, 需要绑定跨进程的追踪, 并且上下文应该在整个过程中随之传播. 这就是 ContextCarrier 的职责.

以下是有关如何在 `A -> B` 分布式调用中使用 **ContextCarrier** 的步骤.

1. 在客户端, 创建一个新的空的 `ContextCarrier`.
1. 通过 `ContextManager#createExitSpan` 创建一个 ExitSpan 或者使用 `ContextManager#inject` 来初始化 `ContextCarrier`.
1. 将 `ContextCarrier` 所有信息放到请求头 (如 HTTP HEAD), 附件(如 Dubbo RPC 框架), 或者消息 (如 Kafka) 中
1. 通过服务调用, 将 `ContextCarrier` 传递到服务端.
1. 在服务端, 在对应组件的头部, 附件或消息中获取 `ContextCarrier` 所有内容.
1. 通过 `ContestManager#createEntrySpan` 创建 EntrySpan 或者使用 `ContextManager#extract` 将服务端和客户端的绑定.

让我们通过 Apache HttpComponent client 插件和 Tomcat 7 服务器插件演示, 步骤如下:

1. 客户端 Apache HttpComponent client 插件

```java
    span = ContextManager.createExitSpan("/span/operation/name", contextCarrier, "ip:port");
    CarrierItem next = contextCarrier.items();
    while (next.hasNext()) {
        next = next.next();
        httpRequest.setHeader(next.getHeadKey(), next.getHeadValue());
    }
```

2. 服务端 Tomcat 7 服务器插件

```java
    ContextCarrier contextCarrier = new ContextCarrier();
    CarrierItem next = contextCarrier.items();
    while (next.hasNext()) {
        next = next.next();
        next.setHeadValue(request.getHeader(next.getHeadKey()));
    }

    span = ContextManager.createEntrySpan("/span/operation/name", contextCarrier);
```

### 上下文快照 (ContextSnapshot)

除了跨进程, 跨线程也是需要支持的, 例如异步线程（内存中的消息队列）和批处理在 Java 中很常见, 跨进程和跨线程十分相似, 因为都是需要传播上下文. 唯一的区别是, 不需要跨线程序列化.

以下是有关跨线程传播的三个步骤：
1. 使用 `ContextManager#capture` 方法获取 ContextSnapshot 对象.
2. 让子线程以任何方式, 通过方法参数或由现有参数携带来访问 ContextSnapshot
3. 在子线程中使用 `ContextManager#continued`.

## 核心 API

### 上下文管理器 (ContextManager)

ContextManager 提供所有主要API.

1. 创建 EntrySpan

```java
public static AbstractSpan createEntrySpan(String endpointName, ContextCarrier carrier)
```

根据操作名称创建 EntrySpan (例如服务名称, uri) 和 **上下文载体 (ContextCarrier)**.

2. 创建 LocalSpan

```java
public static AbstractSpan createLocalSpan(String endpointName)
```

根据操作名称创建 LocalSpan (例如完整的方法签名)

3. 创建 ExitSpan

```java
public static AbstractSpan createExitSpan(String endpointName, ContextCarrier carrier, String remotePeer)
```

根据操作名称创建 ExitSpan (例如服务名称, uri) 和 **上下文载体 (ContextCarrier)** 和对等端 (peer) 地址 (例如 ip + port 或 hostname + port)

### AbstractSpan

```java
    /**
     * Set the component id, which defines in {@link ComponentsDefine}
     *
     * @param component
     * @return the span for chaining.
     */
    AbstractSpan setComponent(Component component);

    /**
     * Only use this method in explicit instrumentation, like opentracing-skywalking-bridge.
     * It it higher recommend don't use this for performance consideration.
     *
     * @param componentName
     * @return the span for chaining.
     */
    AbstractSpan setComponent(String componentName);

    AbstractSpan setLayer(SpanLayer layer);

    /**
     * Set a key:value tag on the Span.
     *
     * @return this Span instance, for chaining
     */
    AbstractSpan tag(String key, String value);

    /**
     * Record an exception event of the current walltime timestamp.
     *
     * @param t any subclass of {@link Throwable}, which occurs in this span.
     * @return the Span, for chaining
     */
    AbstractSpan log(Throwable t);

    AbstractSpan errorOccurred();

    /**
     * Record an event at a specific timestamp.
     *
     * @param timestamp The explicit timestamp for the log record.
     * @param event the events
     * @return the Span, for chaining
     */
    AbstractSpan log(long timestamp, Map<String, ?> event);

    /**
     * Sets the string name for the logical operation this span represents.
     *
     * @return this Span instance, for chaining
     */
    AbstractSpan setOperationName(String endpointName);
```

除了设置操作名称, 标签信息和日志外, 还要设置两个属性, 即 component（组件）和 layer（层）, 特别是对于 EntrySpan 和 ExitSpan.

SpanLayer 是 span 的类别. 有五个值:

1. UNKNOWN (默认值)
1. DB
1. RPC_FRAMEWORK,（针对 RPC 框架, 非普通的 HTTP 调用）
1. HTTP
1. MQ

组件 ID 由 SkyWalking 项目定义和保留, 对于组件的名称或 ID 的扩展, 请遵循[组件库的定义与扩展](Component-library-settings.md) 

// TODO

### 高级 API

#### 异步 Span API

关于 Span 有一系列高级 API, 他们都是在异步场景下使用的. 当 Span 的 tag, 日志和属性(包括结束时间)需要在另一个线程中设置时, 你就应该使用这些 API.

```java
/**
 * The span finish at current tracing context, but the current span is still alive, until {@link #asyncFinish}
 * called.
 *
 * This method must be called<br/>
 * 1. In original thread(tracing context).
 * 2. Current span is active span.
 *
 * During alive, tags, logs and attributes of the span could be changed, in any thread.
 *
 * The execution times of {@link #prepareForAsync} and {@link #asyncFinish()} must match.
 *
 * @return the current span
 */
AbstractSpan prepareForAsync();

/**
 * Notify the span, it could be finished.
 *
 * The execution times of {@link #prepareForAsync} and {@link #asyncFinish()} must match.
 *
 * @return the current span
 */
AbstractSpan asyncFinish();
```

1. 在 `#prepareForAsync` in original context.
1. Propagate the span to any other thread.
1. After all set, call `#asyncFinish` in any thread.
1. Tracing context will be finished and report to backend when all spans's `#prepareForAsync` finished(Judged by count of API execution).


## 开发插件

### 摘要

跟踪的基本方法是拦截Java方法, 使用字节码操作技术和 AOP 概念. SkyWalking 包装了字节码操作技术并跟踪上下文的传播, 所以你只需要定义拦截点（换句话说就是 Spring 的切面）

### 拦截

SkyWalking 提供两类通用的定义去拦截构造器, 实例方法和类方法.
* Extend `ClassInstanceMethodsEnhancePluginDefine` defines `Contructor` intercept points and `instance method` intercept points.
* 继承 `ClassInstanceMethodsEnhancePluginDefine` 定义 `Contructor`（构造器）拦截点和 `instance method`（实例化方法）拦截点.
* 继承 `ClassStaticMethodsEnhancePluginDefine` 定义 `class method`（类方法）拦截点.

当然, 您也可以集成`ClassEnhancePluginDefine`去设置所有的拦截点, 担着不常用.

### Implement plugin（实现插件）
下文, 我将通过扩展`ClassInstanceMethodsEnhancePluginDefine`来演示如何实现一个插件

1. 定义目标类的名称

```java
protected abstract ClassMatch enhanceClass();
```

ClassMatch 以下有四种方法表示如何去匹配目标类:
* byName, 通过完整的类名(package name + `.` + class name)（包名+类名）.
* byClassAnnotationMatch, 通过目标类存在某些注释.
* byMethodAnnotationMatch, 通过目标类的方法存在某些注释.
* byHierarchyMatch, 通过目标类的父类或接口

**注意事项**:
* 禁止使用 `*.class.getName()` 去获取类名,  建议你使用文字字符串, 这是为了
避免ClassLoader问题.
* `by*AnnotationMatch` 不支持继承的注释.
* 非必要的话, 不推荐使用 `byHierarchyMatch`, 因为使用它可能会触发拦截
许多未被发现的方法, 会导致性能问题和不稳定.

实例：

```java
@Override
protected ClassMatch enhanceClassName() {
    return byName("org.apache.catalina.core.StandardEngineValve");		
}

```

2. 定义实例方法拦截点

```java
protected InstanceMethodsInterceptPoint[] getInstanceMethodsInterceptPoints();

public interface InstanceMethodsInterceptPoint {
    /**
     * class instance methods matcher.
     *
     * @return methods matcher
     */
    ElementMatcher<MethodDescription> getMethodsMatcher();

    /**
     * @return represents a class name, the class instance must instanceof InstanceMethodsAroundInterceptor.
     */
    String getMethodsInterceptor();

    boolean isOverrideArgs();
}
```

也可以使用`Matcher`来设置目标方法. 如果要更改参数, 请在`isOverrideArgs`中返回** true ** 参考拦截器.

以下部分将告诉您如何实现拦截器.

3. Add plugin define into skywalking-plugin.def file
```properties
tomcat-7.x/8.x=TomcatInstrumentation
```


### 实现一个拦截器
作为一个实例方法的拦截器, 需要实现
`org.apache.skywalking.apm.agent.core.plugin.interceptor.enhance.InstanceMethodsAroundInterceptor`
```java
/**
 * A interceptor, which intercept method's invocation. The target methods will be defined in {@link
 * ClassEnhancePluginDefine}'s subclass, most likely in {@link ClassInstanceMethodsEnhancePluginDefine}
 *
 * @author wusheng
 */
public interface InstanceMethodsAroundInterceptor {
    /**
     * called before target method invocation.
     *
     * @param result change this result, if you want to truncate the method.
     * @throws Throwable
     */
    void beforeMethod(EnhancedInstance objInst, Method method, Object[] allArguments, Class<?>[] argumentsTypes,
        MethodInterceptResult result) throws Throwable;

    /**
     * called after target method invocation. Even method's invocation triggers an exception.
     *
     * @param ret the method's original return value.
     * @return the method's actual return value.
     * @throws Throwable
     */
    Object afterMethod(EnhancedInstance objInst, Method method, Object[] allArguments, Class<?>[] argumentsTypes,
        Object ret) throws Throwable;

    /**
     * called when occur exception.
     *
     * @param t the exception occur.
     */
    void handleMethodException(EnhancedInstance objInst, Method method, Object[] allArguments, Class<?>[] argumentsTypes,
        Throwable t);
}
```
在before, after和exception处理阶段使用核心API.


### 将插件贡献到Apache SkyWalking 仓库中
我们欢迎大家贡献插件.

请按照以下步骤操作：

1. 提交有关您要贡献哪些插件的问题, 包括支持的版本.
1. 在`apm-sniffer / apm-sdk-plugin`或`apm-sniffer / optional-plugins`下创建子模块, 名称应包含支持的库名和版本
1. Create sub modules under `apm-sniffer/apm-sdk-plugin` or `apm-sniffer/optional-plugins`, and the name should include supported library name and versions
1. 按照本指南进行开发. 确保提供评论和测试用例.
1. 开发并测试.
1. 发送拉取请求并要求审核.
1. 提供自动测试用例.
所有测试用例都托管在[SkyAPMTest/agent-auto-integration-testcases repository](https://github.com/SkyAPMTest/agent-auto-integration-testcases).
关于如何编写测试用例, 请按照[如何编写](https://github.com/SkyAPMTest/agent-auto-integration-testcases/blob/master/docs/how-to-write-a-plugin-testcase.md) 文档来实现.
1. 在提供自动测试用例并在CI中递交测试后, 插件提交者会批准您的插件.
1. SkyWalking接受的插件.
