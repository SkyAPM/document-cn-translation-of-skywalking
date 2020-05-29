# Plugin Development Guide
This document describe how to understand, develop and contribute plugin.

## Concepts
### Span
Span is an important and common concept in distributed tracing system. Learn **Span** from 
[Google Dapper Paper](https://research.google.com/pubs/pub36356.html)  and
[OpenTracing](http://opentracing.io)

SkyWalking supports OpenTracing and OpenTracing-Java API from 2017. Our Span concepts are similar with the paper and OpenTracing.
Also we extend the Span.

There are three types of Span

1.1 EntrySpan
EntrySpan represents a service provider, also the endpoint of server side. As an APM system, we are targeting the 
application servers. So almost all the services and MQ-consumer are EntrySpan(s).

1.2 LocalSpan
LocalSpan represents a normal Java method, which does not relate to remote service, neither a MQ producer/consumer
nor a service(e.g. HTTP service) provider/consumer.

1.3 ExitSpan
ExitSpan represents a client of service or MQ-producer, as named as `LeafSpan` at early age of SkyWalking.
e.g. accessing DB by JDBC, reading Redis/Memcached are cataloged an ExitSpan. 

### ContextCarrier
In order to implement distributed tracing, the trace across process need to be bind, and the context should propagate 
across the process. That is ContextCarrier's duty.

Here are the steps about how to use **ContextCarrier** in a `A->B` distributed call.
1. Create a new and empty `ContextCarrier` at client side.
1. Create an ExitSpan by `ContextManager#createExitSpan` or use `ContextManager#inject` to init the `ContextCarrier`.
1. Put all items of `ContextCarrier` into heads(e.g. HTTP HEAD), attachments(e.g. Dubbo RPC framework) or messages(e.g. Kafka)
1. The `ContextCarrier` propagates to server side by the service call.
1. At server side, get all items from heads, attachments or messages.
1. Create an EntrySpan by `ContextManager#createEntrySpan` or use `ContextManager#extract` to bind the client and server.


Let's demonstrate the steps by Apache HTTPComponent client plugin and Tomcat 7 server plugin
1. Client side steps by Apache HTTPComponent client plugin
```java
            span = ContextManager.createExitSpan("/span/operation/name", contextCarrier, "ip:port");
            CarrierItem next = contextCarrier.items();
            while (next.hasNext()) {
                next = next.next();
                httpRequest.setHeader(next.getHeadKey(), next.getHeadValue());
            }
```

2. Server side steps by Tomcat 7 server plugin
```java
            ContextCarrier contextCarrier = new ContextCarrier();
            CarrierItem next = contextCarrier.items();
            while (next.hasNext()) {
                next = next.next();
                next.setHeadValue(request.getHeader(next.getHeadKey()));
            }

            span = ContextManager.createEntrySpan(“/span/operation/name”, contextCarrier);
```

### ContextSnapshot
Besides across process, across thread but in a process need to be supported, because async process(In-memory MQ) 
and batch process are common in Java. Across process and across thread are similar, because they are both about propagating
context. The only difference is that, don't need to serialize for across thread.

Here are the three steps about across thread propagation:
1. Use `ContextManager#capture` to get the ContextSnapshot object.
1. Let the sub-thread access the ContextSnapshot by any way, through method arguments or carried by an existed arguments
1. Use `ContextManager#continued` in sub-thread.

## Core APIs
### ContextManager
ContextManager provides all major and primary APIs.

1. Create EntrySpan
```java
public static AbstractSpan createEntrySpan(String endpointName, ContextCarrier carrier)
```
Create EntrySpan by operation name(e.g. service name, uri) and **ContextCarrier**.

2. Create LocalSpan
```java
public static AbstractSpan createLocalSpan(String endpointName)
```
Create LocalSpan by operation name(e.g. full method signature)

3. Create ExitSpan
```java
public static AbstractSpan createExitSpan(String endpointName, ContextCarrier carrier, String remotePeer)
```
Create ExitSpan by operation name(e.g. service name, uri) and new **ContextCarrier** and peer address
(e.g. ip+port, hostname+port)

### AbstractSpan
```java
    /**
     * Set the component id, which defines in {@link ComponentsDefine}
     *
     * @param component
     * @return the span for chaining.
     */
    AbstractSpan setComponent(Component component);

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
Besides set operation name, tags and logs, two attributes shoule be set, which are component and layer, 
especially for EntrySpan and ExitSpan

SpanLayer is the catalog of span. Here are 5 values:
1. UNKNOWN (default)
1. DB
1. RPC_FRAMEWORK, for a RPC framework, not an ordinary HTTP
1. HTTP
1. MQ

Component IDs are defined and reserved by SkyWalking project.
For component name/ID extension, please follow [Component library definition and extension](Component-library-settings.md) document.

### Advanced APIs
#### Async Span APIs
There is a set of advanced APIs in Span, which work specific for async scenario. When tags, logs, attributes(including end time) of the span
needs to set in another thread, you should use these APIs.

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
1. Call `#prepareForAsync` in original context.
1. Do `ContextManager#stopSpan` in original context when your job in current thread is done.
1. Propagate the span to any other thread.
1. After all set, call `#asyncFinish` in any thread.
1. Tracing context will be finished and report to backend when all spans's `#prepareForAsync` finished(Judged by count of API execution).

## Develop a plugin
### Abstract
The basic method to trace is intercepting a Java method, by using byte code manipulation tech and AOP concept.
SkyWalking boxed the byte code manipulation tech and tracing context propagation,
so you just need to define the intercept point(a.k.a. aspect pointcut in Spring)

### Intercept
SkyWalking provide two common defines to intercept Contructor, instance method and class method.
* Extend `ClassInstanceMethodsEnhancePluginDefine` defines `Contructor` intercept points and `instance method` intercept points.
* Extend `ClassStaticMethodsEnhancePluginDefine` definec `class method` intercept points.

Of course, you can extend `ClassEnhancePluginDefine` to set all intercept points. But it is unusual. 

### Implement plugin
I will demonstrate about how to implement a plugin by extending `ClassInstanceMethodsEnhancePluginDefine`

1. Define the target class name
```java
protected abstract ClassMatch enhanceClass();
```

ClassMatch represents how to match the target classes, there are 4 ways:
* byName, through the full class name(package name + `.` + class name)
* byClassAnnotationMatch, through the class existed certain annotations.
* byMethodAnnotationMatch, through the class's method existed certain annotations.
* byHierarchyMatch, through the class's parent classes or interfaces

**Attentions**:
* Never use `ThirdPartyClass.class` in the instrumentation definitions, such as `takesArguments(ThirdPartyClass.class)`, or `byName(ThirdPartyClass.class.getName())`, because of the fact that `ThirdPartyClass` dose not necessarily exist in the target application and this will break the agent; we have `import` checks to help on checking this in CI, but it doesn't cover all scenarios of this limitation, so never try to work around this limitation by something like using full-qualified-class-name (FQCN), i.e. `takesArguments(full.qualified.ThirdPartyClass.class)` and `byName(full.qualified.ThirdPartyClass.class.getName())` will pass the CI check, but are still invalid in the agent codes, **Use Full Qualified Class Name String Literature Instead**.
* Even you are perfectly sure that the class to be intercepted exists in the target application (such as JDK classes), still, don't use `*.class.getName()` to get the class String name. Recommend you to use literal String. This is for 
avoiding ClassLoader issues.
* `by*AnnotationMatch` doesn't support the inherited annotations.
* Don't recommend to use `byHierarchyMatch`, unless it is really necessary. Because using it may trigger intercepting 
many unexcepted methods, which causes performance issues and concerns.

Example：
```java
@Override
protected ClassMatch enhanceClassName() {
    return byName("org.apache.catalina.core.StandardEngineValve");		
}		      

```

2. Define an instance method intercept point
```java
public InstanceMethodsInterceptPoint[] getInstanceMethodsInterceptPoints();

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
Also use `Matcher` to set the target methods. Return **true** in `isOverrideArgs`, if you want to change the argument
ref in interceptor.

The following sections will tell you how to implement the interceptor.

3. Add plugin define into skywalking-plugin.def file
```properties
tomcat-7.x/8.x=TomcatInstrumentation
```


### Implement an interceptor
As an interceptor for an instance method, the interceptor implements 
`org.apache.skywalking.apm.agent.core.plugin.interceptor.enhance.InstanceMethodsAroundInterceptor`
```java
/**
 * A interceptor, which intercept method's invocation. The target methods will be defined in {@link
 * ClassEnhancePluginDefine}'s subclass, most likely in {@link ClassInstanceMethodsEnhancePluginDefine}
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
Use the core APIs in before, after and exception handle stages.

### Do bootstrap class instrumentation.
SkyWalking has packaged the bootstrap instrumentation in the agent core. It is easy to open by declaring it in the Instrumentation definition.

Override the `public boolean isBootstrapInstrumentation()` and return **true**. Such as
```java
public class URLInstrumentation extends ClassEnhancePluginDefine {
    private static String CLASS_NAME = "java.net.URL";

    @Override protected ClassMatch enhanceClass() {
        return byName(CLASS_NAME);
    }

    @Override public ConstructorInterceptPoint[] getConstructorsInterceptPoints() {
        return new ConstructorInterceptPoint[] {
            new ConstructorInterceptPoint() {
                @Override public ElementMatcher<MethodDescription> getConstructorMatcher() {
                    return any();
                }

                @Override public String getConstructorInterceptor() {
                    return "org.apache.skywalking.apm.plugin.jre.httpurlconnection.Interceptor2";
                }
            }
        };
    }

    @Override public InstanceMethodsInterceptPoint[] getInstanceMethodsInterceptPoints() {
        return new InstanceMethodsInterceptPoint[0];
    }

    @Override public StaticMethodsInterceptPoint[] getStaticMethodsInterceptPoints() {
        return new StaticMethodsInterceptPoint[0];
    }

    @Override public boolean isBootstrapInstrumentation() {
        return true;
    }
}
```

**NOTICE**, doing bootstrap instrumentation should only happen in necessary, but mostly it effect the JRE core(rt.jar),
and could make very unexpected result or side effect.


### Contribute plugins into Apache SkyWalking repository
We are welcome everyone to contribute plugins.

Please follow there steps:
1. Submit an issue about which plugins you are going to contribute, including supported version.
1. Create sub modules under `apm-sniffer/apm-sdk-plugin` or `apm-sniffer/optional-plugins`, and the name should include supported library name and versions
1. Follow this guide to develop. Make sure comments and test cases are provided.
1. Develop and test.
1. Provide the automatic test cases. Learn `how to write the plugin test case` from this [doc](Plugin-test.md)
1. Send the pull request and ask for review. 
1. The plugin committers approve your plugins, plugin CI-with-IT, e2e and plugin tests passed.
1. The plugin accepted by SkyWalking. 
