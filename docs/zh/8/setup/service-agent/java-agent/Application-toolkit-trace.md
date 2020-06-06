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
* 如果标签处没有有效跨度，则用@Tag注释的方法将尝试使用给定键(`Tag＃key()`)和(`Tag#value()`)标记**当前有效跨度**。
若有，此注释不起作用。可以重复使用@Tag，并且可以与@Trace结合使用，请参见下面的示例。
 “标签”的“值”与[自定义增强跟踪](Customize-enhance-trace.md)中支持的值相同。
* 在跟踪方法的上下文中添加自定义标签, `ActiveSpan.tag("key", "val")`.

* `ActiveSpan.error()` 将当前跨度标记为错误状态。
* `ActiveSpan.error(String errorMsg)` 将当前跨度标记为错误状态,并设置错误消息提示。
* `ActiveSpan.error(Throwable throwable)` 将当前跨度标记为错误状态,并抛出Throwable异常。
* `ActiveSpan.debug(String debugMsg)` 在当前范围中添加调试级别日志消息。
* `ActiveSpan.info(String infoMsg)` 在当前范围中添加信息级别日志消息。
* `ActiveSpan.setOperationName(String operationName)` 自定义操作名称。

```java
ActiveSpan.tag("my_tag", "my_value");
ActiveSpan.error();
ActiveSpan.error("Test-Error-Reason");

ActiveSpan.error(new RuntimeException("Test-Error-Throwable"));
ActiveSpan.info("Test-Info-Msg");
ActiveSpan.debug("Test-debug-Msg");

/**
 * The codes below will generate a span,
 * and two types of tags, 
      one type tag: keys are `tag1` and `tag2`, values are the passed-in parameters, respectively, 
      the other type tag: keys are `username`  and `age`, values are the return value in User, respectively
 */
@Trace
@Tag(key = "tag1", value = "arg[0]")
@Tag(key = "tag2", value = "arg[1]")
@Tag(key = "username", value = "returnedObj.username")
@Tag(key = "age", value = "returnedObj.age")
public User methodYouWantToTrace(String param1, String param2) {
    // ActiveSpan.setOperationName("Customize your own operation name, if this is an entry span, this would be an endpoint name");
    // ...
}
```

* 使用 `TraceContext.putCorrelation()` API将自定义数据放入跟踪上下文。
```java
Optional<String> previous = TraceContext.putCorrelation("customKey", "customValue");
```
当值为`null`或为空时，CorrelationContext将删除该项目。

* 使用 `TraceContext.getCorrelation()` 获取自定义数据的API。
```java
Optional<String> value = TraceContext.getCorrelation("customKey");
```
可以在[代理配置](README.md#table-of-agent-configuration-properties)文件中找到CorrelationContext配置描述，前缀为`correlation.`。
