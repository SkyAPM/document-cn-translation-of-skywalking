# 插件开发指南

本文档描述了如何理解, 开发和贡献插件.

## 概念

### Span

Span 是分布式追踪系统中一个重要且常用的概念. 可从 [Google Dapper Paper](https://research.google.com/pubs/pub36356.html) 和 [OpenTracing](http://opentracing.io) 学习更多与 **Span** 相关的知识.

SkyWalking 从 2017 年开始支持 OpenTracing 和 OpenTracing-Java API, 我们的 Span 在概念上与谷歌论文和 OpenTracing 里描述的类似. 我们也扩展了 Span.

Span 有三种类型

1.1 EntrySpan

EntrySpan 代表服务提供者, 也是服务器端的端点. 作为一个 APM 系统, 我们的目标是应用服务器. 所以几乎所有的服务和 MQ-消费者 都是 EntrySpan.

1.2 LocalSpan

LocalSpan 表示普通的 Java 方法, 它与远程服务无关, 不是 MQ 生产者/消费者, 也不是服务（例如 HTTP 服务）提供者/消费者.

1.3 ExitSpan

ExitSpan 代表一个服务客户端或 MQ 的生产者, 在 SkyWalking 的早期命名为 `LeafSpan`. 例如 通过 JDBC 访问 DB, 读取 Redis/Memcached 被归类为 ExitSpan.

### 上下文载体 (ContextCarrier)

为了实现分布式追踪, 需要绑定跨进程的追踪, 并且上下文应该在整个过程中随之传播. 这就是 ContextCarrier 的职责.

以下是有关如何在 `A -> B` 分布式调用中使用 **ContextCarrier** 的步骤.

1. 在客户端, 创建一个新的空的 `ContextCarrier`.
1. 通过 `ContextManager#createExitSpan` 创建一个 ExitSpan 或者使用 `ContextManager#inject` 来初始化 `ContextCarrier`.
1. 将 `ContextCarrier` 所有信息放到请求头 (如 HTTP HEAD), 附件(如 Dubbo RPC 框架), 或者消息 (如 Kafka) 中
1. 通过服务调用, 将 `ContextCarrier` 传递到服务端.
1. 在服务端, 在对应组件的头部, 附件或消息中获取 `ContextCarrier` 所有内容.
1. 通过 `ContestManager#createEntrySpan` 创建 EntrySpan 或者使用 `ContextManager#extract` 来绑定服务端和客户端.

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

除了跨进程, 跨线程也是需要支持的, 例如异步线程（内存中的消息队列）和批处理在 Java 中很常见. 跨进程和跨线程十分相似, 因为都是需要传播上下文. 唯一的区别是, 跨线程不需要序列化.

以下是有关跨线程传播的三个步骤：
1. 使用 `ContextManager#capture` 方法获取 ContextSnapshot 对象.
2. 让子线程以任何方式, 通过方法参数或由现有参数携带来访问 ContextSnapshot
3. 在子线程中使用 `ContextManager#continued`.

## 核心 API

### 上下文管理器 (ContextManager)

ContextManager 提供所有主要 API.

1. 创建 EntrySpan

```java
public static AbstractSpan createEntrySpan(String endpointName, ContextCarrier carrier)
```

根据操作名称(例如服务名称, uri) 和 **上下文载体 (ContextCarrier)** 创建 EntrySpan.

2. 创建 LocalSpan

```java
public static AbstractSpan createLocalSpan(String endpointName)
```

根据操作名称(例如完整的方法签名)创建 LocalSpan.

3. 创建 ExitSpan

```java
public static AbstractSpan createExitSpan(String endpointName, ContextCarrier carrier, String remotePeer)
```

根据操作名称(例如服务名称, uri), **上下文载体 (ContextCarrier)** 以及对等端 (peer) 地址(例如 ip + port 或 hostname + port) 创建 ExitSpan.

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

组件 ID 由 SkyWalking 项目定义和保留, 对于组件的名称或 ID 的扩展, 请遵循[组件库的定义与扩展](Component-library-settings.md).

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

1. 在原始上下文中调用 `#prepareForAsync`.
1. 将该 Span  传播到其他线程.
1. 在全部操作就绪之后, 可在任意线程中调用 `#asyncFinish` 结束调用.
1. 当所有 Span 的 `#prepareForAsync` 完成后, 追踪上下文会结束, 并一起被回传到后端服务(根据 API 执行次数判断).

## 开发插件

### 摘要

追踪的基本方法是拦截 Java 方法, 使用字节码操作技术和 AOP 概念. SkyWalking 包装了字节码操作技术, 并追踪上下文的传播. 所以你只需要定义拦截点(换句话说就是 Spring 的切面).

### 拦截

SkyWalking 提供两类通用的定义去拦截构造方法, 实例方法和类方法.
* `ClassInstanceMethodsEnhancePluginDefine` 定义了构造方法 `Contructor` 拦截点和 `instance method` 实例方法拦截点.
* `ClassStaticMethodsEnhancePluginDefine` 定义了类方法 `class method` 拦截点.

当然, 您也可以继承 `ClassEnhancePluginDefine` 去设置所有的拦截点, 但这不常用.

### 实现插件

下文, 我将通过扩展 `ClassInstanceMethodsEnhancePluginDefine` 来演示如何实现一个插件

1. 定义目标类的名称

```java
protected abstract ClassMatch enhanceClass();
```

ClassMatch 表示如何去匹配目标类, 这里有四种方法:
* byName, 通过类的全限定名(Fully Qualified Class Name, 即 包名 + `.` + 类名).
* byClassAnnotationMatch, 根据目标类是否存在某些注解.
* byMethodAnnotationMatch, 根据目标类的方法是否存在某些注解.
* byHierarchyMatch, 根据目标类的父类或接口

**注意事项**:
* 禁止使用 `*.class.getName()` 去获取类名, 建议你使用文本字符串, 这是为了避免 ClassLoader 的问题.
* `by*AnnotationMatch` 不支持从父类继承来的注解.
* 除非确实必要, 否则不建议使用 `byHierarchyMatch`, 因为使用它可能会触发拦截许多预期之外的方法, 会导致性能问题和不稳定.

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

也可以使用 `Matcher` 来设置目标方法. 如果要在拦截器中更改引用参数, 请在 `isOverrideArgs` 中返回 **true**.

以下部分将告诉您如何实现拦截器.

3. 在文件 `skywalking-plugin.def` 中添加插件定义

```properties
tomcat-7.x/8.x=TomcatInstrumentation
```

### 实现一个拦截器

作为一个实例方法的拦截器, 需要实现 `org.apache.skywalking.apm.agent.core.plugin.interceptor.enhance.InstanceMethodsAroundInterceptor`

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

在方法调用前, 调用后以及异常处理阶段使用核心 API.

### 将插件贡献到 Apache SkyWalking 仓库中

我们欢迎大家贡献插件.

请按照以下步骤操作：

1. 提交有关您要贡献哪些插件的问题, 包括支持的版本.
1. 在 `apm-sniffer/apm-sdk-plugin` 或 `apm-sniffer/optional-plugins` 下创建子模块, 名称应包含支持的库名和版本
1. 按照本指南进行开发. 确保提供注释和测试用例.
1. 开发并测试.
1. 发送 Pull Request 并要求审核.
1. 提供自动测试用例.
1. 在提供自动测试用例并在 CI 通过测试后, 插件审批人员会批准您的插件.
1. SkyWalking 接受新的插件.
