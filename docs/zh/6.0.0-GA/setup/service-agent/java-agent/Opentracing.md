* 使用maven或gradle引入toolkit依赖。
```xml
   <dependency>
      <groupId>org.apache.skywalking</groupId>
      <artifactId>apm-toolkit-opentracing</artifactId>
      <version>{project.release.version}</version>
   </dependency>
```


* 使用我们的OpenTracing tracer实现
```java
Tracer tracer = new SkywalkingTracer();
Tracer.SpanBuilder spanBuilder = tracer.buildSpan("/yourApplication/yourService");

```
