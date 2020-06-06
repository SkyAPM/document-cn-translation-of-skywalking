* 通过maven或gradle引入toolkit依赖。
```xml
   <dependency>
      <groupId>org.apache.skywalking</groupId>
      <artifactId>apm-toolkit-log4j-1.x</artifactId>
      <version>{project.release.version}</version>
   </dependency>
```

* 配置layout
```properties
log4j.appender.CONSOLE.layout=TraceIdPatternLayout
```

* 在`layout.ConversionPattern`中设置`%T` (在2.0-2016中，你应该使用%x，[为什么做了修改?](https://github.com/wu-sheng/sky-walking/issues/77) )
```properties
log4j.appender.CONSOLE.layout.ConversionPattern=%d [%T] %-5p %c{1}:%L - %m%n
```

* 当你使用`-javaagent`激活skywalking tracer后，log4j将会输出**traceId**（如果存在的话）。如果tracer未激活，输出将是`TID: N/A`。
