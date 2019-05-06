# IP和端口设置
Backend使用IP和端口绑定，以支持具有多个IP的操作系统。
绑定/监听IP和端口由核心模块指定。
```yaml
core:
  default:
    restHost: 0.0.0.0
    restPort: 12800
    restContextPath: /
    gRPCHost: 0.0.0.0
    gRPCPort: 11800
```
有两对IP/port，分别提供给gRPC和HTTP的rest服务。

- 大多数agent和探针使用GRPC服务以获得更好的性能和代码可读性。
- 少数agent使用REST服务，因为该语言可能不支持GRPC。
- 用户界面使用REST服务，但数据始终采用graphql格式。


## 注意
### IP绑定
如果有些用户不熟悉IP绑定，您应该知道，当IP绑定完成后，客户端只能使用此IP访问服务。例如，绑定了`172.09.13.28`，即使是在此计算机中，必须使用`172.09.13.28`而不是`127.0.0.1`或`localhost`来访问服务。

### 模块提供者指定的IP和端口
核心模块的IP和端口默认有核心模块提供。但某些模块提供者可能提供其它的IP和端口设置，这很正常。比如一些接收模块。.


