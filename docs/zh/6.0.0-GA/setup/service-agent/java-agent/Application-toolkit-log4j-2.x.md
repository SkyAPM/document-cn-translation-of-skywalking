* 使用maven或gradle引入toolkit依赖。
```xml
   <dependency>
      <groupId>org.apache.skywalking</groupId>
      <artifactId>apm-toolkit-log4j-2.x</artifactId>
      <version>{project.release.version}</version>
   </dependency>
```

* 在log4j2.xml的pattern中配置`[%traceId]`
```xml
   <Appenders>
      <Console name="Console" target="SYSTEM_OUT">
         <PatternLayout pattern="%d [%traceId] %-5p %c{1}:%L - %m%n"/>
      </Console>
   </Appenders>
```
* 当你使用`-javaagent`激活skywalking tracer后，log4j2将会输出**traceId**（如果存在的话）。如果tracer未激活，输出将是`TID: N/A`。
