### 现象
当你通过agent启动你的应用时，如果发现如下异常，意味着“EnhanceRequireObjectCache”不能强制转换为“EnhanceRequireObjectCache”。例：
```java
ERROR 2018-05-07 21:31:24 InstMethodsInter :  class[class org.springframework.web.method.HandlerMethod] after method[getBean] intercept failure
java.lang.ClassCastException: org.apache.skywalking.apm.plugin.spring.mvc.commons.EnhanceRequireObjectCache cannot be cast to org.apache.skywalking.apm.plugin.spring.mvc.commons.EnhanceRequireObjectCache
	at org.apache.skywalking.apm.plugin.spring.mvc.commons.interceptor.GetBeanInterceptor.afterMethod(GetBeanInterceptor.java:45)
	at org.apache.skywalking.apm.agent.core.plugin.interceptor.enhance.InstMethodsInter.intercept(InstMethodsInter.java:105)
	at org.springframework.web.method.HandlerMethod.getBean(HandlerMethod.java)
	at org.springframework.web.servlet.handler.AbstractHandlerMethodExceptionResolver.shouldApplyTo(AbstractHandlerMethodExceptionResolver.java:47)
	at org.springframework.web.servlet.handler.AbstractHandlerExceptionResolver.resolveException(AbstractHandlerExceptionResolver.java:131)
	at org.springframework.web.servlet.handler.HandlerExceptionResolverComposite.resolveException(HandlerExceptionResolverComposite.java:76)
	...
```

### 原因
此异常可能由某些热部署工具(`spring-boot-devtool`)或者其它可以在运行时改变`classloader` 的工具导致。
### 解决方法
1. 生产环境不会出现此错误是因为开发者工具自动关闭了，详细见[spring-boot-devtools说明](https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-devtools.html)
2. 开发环境如果想正常调试,可以暂时把lib路径下的热部署相关的jar包移除。
