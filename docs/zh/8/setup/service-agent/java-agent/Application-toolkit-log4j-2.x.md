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

* 支持log4j2 AsyncRoot，无需其他配置。请参阅下面的log4j2.xml演示。有关详细信息：[Log4j2异步记录器](https://logging.apache.org/log4j/2.x/manual/async.html)
```xml
    <Configuration>
        <Appenders>
            <Console name="Console" target="SYSTEM_OUT">
                <PatternLayout pattern="%d [%traceId] %-5p %c{1}:%L - %m%n"/>
            </Console>
        </Appenders>
        <Loggers>
            <AsyncRoot level="INFO">
                <AppenderRef ref="Console"/>
            </AsyncRoot>
        </Loggers>
    </Configuration>
```
* 支持log4j2 AsyncAppender，不需要其他配置。请参阅下面的log4j2.xml演示。 

   有关详细信息： [All Loggers Async](https://logging.apache.org/log4j/2.x/manual/async.html#AllAsync) 

    Log4j-2.9和更高版本要求在类路径上使用disruptor-3.3.4.jar或更高版本。在Log4j-2.9之前，需要使用interrupter-3.0.0.jar或更高版本。
    这是最简单的配置，并提供最佳性能。要使所有记录器异步，请将disruptor jar添加到类路径中并且 
    设置系统属性 `log4j2.contextSelector`为 `org.apache.logging.log4j.core.async.AsyncLoggerContextSelector`.
    ```xml
    <Configuration status="WARN">
      <Appenders>
        <!-- Async Loggers will auto-flush in batches, so switch off immediateFlush. -->
        <RandomAccessFile name="RandomAccessFile" fileName="async.log" immediateFlush="false" append="false">
          <PatternLayout>
            <Pattern>%d %p %c{1.} [%t] [%traceId] %m %ex%n</Pattern>
          </PatternLayout>
        </RandomAccessFile>
      </Appenders>
      <Loggers>
        <Root level="info" includeLocation="false">
          <AppenderRef ref="RandomAccessFile"/>
        </Root>
      </Loggers>
    </Configuration>
    ```
    For details: [Mixed Sync & Async](https://logging.apache.org/log4j/2.x/manual/async.html#MixedSync-Async)
    
    Log4j-2.9 and higher require disruptor-3.3.4.jar or higher on the classpath. Prior to Log4j-2.9, disruptor-3.0.0.jar or higher was required. 
    There is no need to set system property `Log4jContextSelector` to any value.
    
    ```xml
    <Configuration status="WARN">
      <Appenders>
        <!-- Async Loggers will auto-flush in batches, so switch off immediateFlush. -->
        <RandomAccessFile name="RandomAccessFile" fileName="asyncWithLocation.log"
                  immediateFlush="false" append="false">
          <PatternLayout>
            <Pattern>%d %p %class{1.} [%t] [%traceId] %location %m %ex%n</Pattern>
          </PatternLayout>
        </RandomAccessFile>
      </Appenders>
      <Loggers>
        <!-- pattern layout actually uses location, so we need to include it -->
        <AsyncLogger name="com.foo.Bar" level="trace" includeLocation="true">
          <AppenderRef ref="RandomAccessFile"/>
        </AsyncLogger>
        <Root level="info" includeLocation="true">
          <AppenderRef ref="RandomAccessFile"/>
        </Root>
      </Loggers>
    </Configuration>
    ```
* 支持log4j2 AsyncAppender，有关详细信息： [Log4j2 AsyncAppender](https://logging.apache.org/log4j/2.x/manual/appenders.html)
```xml
    <Configuration>
        <Appenders>
            <Console name="Console" target="SYSTEM_OUT">
                <PatternLayout pattern="%d [%traceId] %-5p %c{1}:%L - %m%n"/>
            </Console>
            <Async name="Async">
                <AppenderRef ref="Console"/>
            </Async>
        </Appenders>
        <Loggers>
            <Root level="INFO">
                <AppenderRef ref="Async"/>
            </Root>
        </Loggers>
    </Configuration>
```
* 当你使用`-javaagent`激活skywalking tracer后，log4j2将会输出**traceId**（如果存在的话）。如果tracer未激活，输出将是`TID: N/A`。
