# 不注册本地span和退出span

从6.6.0开始，SkyWalking取消了本地span和退出span寄存器。如果旧的java代理(6.6.0之前)仍然在运行，并且注册到6.6.0+后端，您将面临以下警告消息.
```
class=RegisterServiceHandler, message = Unexpected endpoint register, endpoint isn't detected from server side.
```

这不会损害后端或导致任何问题。这提醒您，您的代理或其他客户端应该遵循新的协议要求.

您可以使用 `log4j2.xml` 轻松地过滤此警告消息.