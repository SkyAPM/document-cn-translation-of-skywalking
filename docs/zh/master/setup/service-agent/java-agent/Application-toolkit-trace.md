* 使用maven或gradle引入toolkit依赖。
```xml
   <dependency>
      <groupId>org.apache.skywalking</groupId>
      <artifactId>apm-toolkit-trace</artifactId>
      <version>${skywalking.version}</version>
   </dependency>
```

* 使用 `TraceContext.traceId()` API得到traceId
```java
import TraceContext;
...

modelAndView.addObject("traceId", TraceContext.traceId());
```
_仅为示例代码_

* 在你想追踪的方法上添加`@Trace`注解。添加后，你就可以在方法调用栈中查看到span的信息。
* 在被追踪的方法的上下文周期内添加自定义tag。
```java
ActiveSpan.tag("my_tag", "my_value");
```
