# 无法打点的网关/代理

SkyWalking agent 插件启动时无法对网关进行打点，但它们是可以在 `gateways.yml` 或者通过[动态配置](dynamic-config.md)设置。他们不能自动在后端注册的原因是没有合适的代理插件，例如，没有代理插件的 Nginx, haproxy 等。因此，为了可视化真实的拓扑，我们提供了一种手动配置网关/代理的方法。

## 配置格式

配置内容包括网关的名字和其他实例：

```yml
gateways:
 - name: proxy0 # the name is not used for now
   instances:
     - host: 127.0.0.1 # the host/ip of this gateway instance
       port: 9099 # the port of this gateway instance, defaults to 80
```

**注意**  实例的`host`必须是客户端实际使用的，例如如果 `proxyA` 实例有两个 IP `192.168.1.110` 和 `192.168.1.111`，两个都代理目标服务，客户端连接 `192.168.1.110`，那么配置 `192.168.1.111` 作为 `host` 是无法工作的配置项。
