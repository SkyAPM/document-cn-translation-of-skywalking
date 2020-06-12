# 配置覆盖
默认情况下，SkyWalking为agent提供了`agent.config`配置文件。

配置覆盖意味着用户可以通过系统属性或agent选项覆盖配置文件中的配置。

## 系统属性

使用 `skywalking.` + 配置文件中的配置名 作为系统属性的配置名来覆盖配置文件中的值。

- 为什么需要这个前缀？

  因为agent系统属性是和目标应用共享的，加前缀是为了避免冲突。

- 例子 

  通过下面的系统属性覆盖`agent.application_code`。
```
-Dskywalking.agent.application_code=31200
```

## agent选项

在JVM参数的agent路径后面添加选项。

```
-javaagent:/path/to/skywalking-agent.jar=[option1]=[value1],[option2]=[value2]
```

- 例子 

  通过下面的配置覆盖`agent.application_code``logging.level`。

```
-javaagent:/path/to/skywalking-agent.jar=agent.application_code=31200,logging.level=debug
```

- 特殊字符

  如果在选项或选项值中有分隔符(`,`或者`=`)，应该用引号包起来。

```
-javaagent:/path/to/skywalking-agent.jar=agent.ignore_suffix='.jpg,.jpeg'
```

## 系统环境变量
- 例子 

  通过下面的配置覆盖`agent.application_code`和`logging.level`。

```
# The service name in UI
agent.service_name=${SW_AGENT_NAME:Your_ApplicationName}

# Logging level
logging.level=${SW_LOGGING_LEVEL:INFO}
```

如果 `SW_AGENT_NAME ` 环境变量在你的操作系统中已存在，并且，它的值为 `skywalking-agent-demo`，那么这里的`agent.service_name`的值将会被覆写为
`skywalking-agent-demo`, 否则, 它将会被设置成 `Your_ApplicationName`。

另外，占位符嵌套也是支持的，比如 `${SW_AGENT_NAME:${ANOTHER_AGENT_NAME:Your_ApplicationName}}`。
在这种情况下，如果 `SW_AGENT_NAME ` 环境变量不存在，但是 ```ANOTHER_AGENT_NAME``` 环境变量存在，并且它的值为 `skywalking-agent-demo`, 那么这里的`agent.service_name`的值将会被覆写为`skywalking-agent-demo`, 否则, 它将会被设置成 `Your_ApplicationName`。


## 覆盖优先级

agent选项 > 系统属性(-D) > 系统环境变量 > 配置文件 
