# UI
Skywalking UI的发行版已经包含在我们的Apache官方版本中。

## 启动
启动脚本也在`/bin/webappService.sh`(.bat). UI 运行一个java进程，用的zuul框架。

## 设置
UI的设置文件是发行包里的`webapp/webapp.yml`。它由三个部分组成。

1. 监听端口
1. Backend连接信息
1. 认证设置

```yaml
server:
  port: 8080

collector:
  path: /graphql
  ribbon:
    ReadTimeout: 10000
    # Point to all backend's restHost:restPort, split by , 
    listOfServers: 10.2.34.1:12800,10.2.34.2:12800

security:
  user:
    # username
    admin:
      # password
      password: admin

```

## 另一个开源UI RocketBot

Apache SkyWalking的一个时髦且让人惊叹的UI。

Github: [https://github.com/TinyAllen/rocketbot](https://github.com/TinyAllen/rocketbot)
