# 令牌鉴权

## 令牌 
在当前版本中，令牌为简单的字符串。

### 设置令牌
在agent.config文件中设置令牌
```properties
# Authentication active is based on backend setting, see application.yml for more details.
agent.authentication = xxxx
```

同时，开启[后端令牌鉴权](../../backend/backend-token-auth.md).

## 鉴权失败 
Collector会对从agent来的每一个请求进行验证，仅允许令牌匹配上的请求通过。

如果令牌不对，你会在agent日志中看到如下错误。
```
org.apache.skywalking.apm.dependencies.io.grpc.StatusRuntimeException: PERMISSION_DENIED
```

## FAQ
### 我能使用令牌鉴权代替TLS吗?
不能，也不应该。从技术的角度，当然是可以的，令牌和TLS都用于不可信的网络环境。在不可信网络下，TLS与令牌相比应当优先考虑。只有在使用TLS时，令牌才是可信的。在非TLS网络中，在发送令牌时，很容易被窃取。

### 支持其他鉴权机制吗？比如ak/sk？
目前不支持。但是我们真诚感谢其他伙伴对此特性贡献力量。

