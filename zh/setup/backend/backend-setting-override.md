# 设置的覆盖
SkyWalking backend支持通过系统属性和环境变量来覆盖的设置。 
你可以覆盖`application.yml`文件中设置。

## 系统属性key规则
**ModuleName**.**ProviderName**.**SettingKey**.

- 事例

  覆盖设置分区的`restHost`
  
```yaml
core:
  default:
    restHost: ${SW_CORE_REST_HOST:0.0.0.0}
    restPort: ${SW_CORE_REST_PORT:12800}
    restContextPath: ${SW_CORE_REST_CONTEXT_PATH:/}
    gRPCHost: ${SW_CORE_GRPC_HOST:0.0.0.0}
    gRPCPort: ${SW_CORE_GRPC_PORT:11800}
```

使用命令行
```
-Dcore.default.restHost=172.0.4.12
```

## 系统环境变量
- 事例

  通过环境变量覆盖设置分区的`restHost`
  
```yaml
core:
  default:
    restHost: ${REST_HOST:0.0.0.0}
    restPort: ${SW_CORE_REST_PORT:12800}
    restContextPath: ${SW_CORE_REST_CONTEXT_PATH:/}
    gRPCHost: ${SW_CORE_GRPC_HOST:0.0.0.0}
    gRPCPort: ${SW_CORE_GRPC_PORT:11800}
```

如果`REST_HOST `存在于你操作系统的环境变量中，并且它的值为`172.0.4.12`，
`restHost`的值就会被重写为`172.0.4.12`,否则就会被设为`0.0.0.0`。

顺便说一下，占位符嵌套也支持，比如`${REST_HOST:${ANOTHER_REST_HOST:127.0.0.1}}`。
在这种情况下，如果`REST_HOST `环境变量不存在，但```REST_ANOTHER_REST_HOSTHOST```环境变量存在并且其值为`172.0.4.12`，此处的`restHost`也会被重写为`172.0.4.12`，否则就会被设为`127.0.0.1`。




