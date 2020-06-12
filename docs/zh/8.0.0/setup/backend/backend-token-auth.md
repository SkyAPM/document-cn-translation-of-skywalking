# 令牌认证

## 支持版本

7.0.0 以上

## 为什么在我们拥有TLS之后还需要令牌身份验证

TLS 是关于传输安全的, 用于确保网络是可信的。令牌身份验证是关于监控应用数据是**可信**的。

## 令牌

当前版本，令牌考虑使用一个简单的字符串。

### 设置令牌

#### 1. 在 agent.config 中设置令牌

```properties
# 认证由后台设定激活，更多详情查看 application.yml 。
agent.authentication = ${SW_AGENT_AUTHENTICATION:xxxx}
```

#### 2. 在 `application.yml` 中设置令牌

```yaml
······
receiver-sharing-server:
  default:
    authentication: ${SW_AUTHENTICATION:""}
······
```

## 认证失败

Skywalking OAP 验证每个来自 agent 的请求，只允许令牌与 `application.yml` 中配置一致请求通过。

如果令牌不正确，将会在 agent 端看到如下日志

```java
org.apache.skywalking.apm.dependencies.io.grpc.StatusRuntimeException: PERMISSION_DENIED
```

## FAQ

### 可以使用令牌认证替代 TSL 吗

不能，在技术方面，你当然可以，但是令牌和 TLS 用于不受信任的网络环境。在这种情况下， TLS 具有更高的优先级。令牌只能在 TLS 保护下受信任。如果你在 非 TLS 的网络中发送，令牌很容易被窃取。

### 是否支持其他身份验证机制? 例如 ak/sk

现在没有，但是我们希望有人可以贡献这个特性。
