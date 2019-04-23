## 支持忽略自定义的traceSupport custom trace ignore
`apm-trace-ignore-plugin`是可选插件。

## 介绍 
- 这个插件的目的是过滤掉你希望被忽略的endpoint。
- 你可以设置多个URL路径模式。匹配到的endpoint将会不会被追踪。
- 当前的匹配规则遵循`Ant Path`匹配样式，比如`/path/*`，`/path/**`，`/path/?`。
- 将`apm-trace-ignore-plugin-x.jar`复制到`agent/plugins`，重启`agent`，插件将会生效。                                                                                                         

## 如何配置 
有两种方式配置要忽略的endpoint的模式。通过系统环境变量配置有更高的优先级。
 1. 通过设置系统环境变量配置。增加 `skywalking.trace.ignore_path` 到系统环境变量中，值是要忽略的路径。多个路径之间用`,`分隔。
 2. 将`/agent/optional-plugins/apm-trace-ignore-plugin/apm-trace-ignore-plugin.config` 复制到 `/agent/config/` 目录下, 增加过滤规则。
```
trace.ignore_path=/your/path/1/**,/your/path/2/**
```

