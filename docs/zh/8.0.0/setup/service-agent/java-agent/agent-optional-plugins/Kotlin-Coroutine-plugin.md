# Skywalking支持Kotlin插件
Skywalking支持Kotlin编写的自动工具支持插件.

## 简述
SkyWalking 提供跟踪上下文在线程内的传播。为了支持Kotlin协同程序，我们提供了这个附加插件

## 实现原理
我们知道，Kotlin coroutine通过“CoroutineDispatcher”来切换执行线程。

01. 在分派延续之前，创建当前上下文的快照.
02. 然后在线程切换后创建协程跨度，标记跨度与快照.
03. 在新线程中创建的每个新span都是这个协同程序span的子跨度。我们可以用轨迹把这些联系起来。
04. 在原来的runnable执行之后，我们需要停止协同程序跨度来清理线程状态。

## 一些截图
### 运行没有插件
我们运行一个基于Kotlin协同程序的gRPC服务器，而不需要这个协同程序插件。
您可以发现，一个调用(client -> server1 -> server2)被分割为两条跟踪路径。

01. 没有退出span的Server1和server2链路.
![不包含kotlin插件](http://skywalking.apache.org/screenshots/7.0.0/kotlin/coroutine/without-coroutine-plugin-server1.jpg)
02. Server2链路.
![不包含kotlin插件](http://skywalking.apache.org/screenshots/7.0.0/kotlin/coroutine/without-coroutine-plugin-server2.jpg)

### 使用插件运行
无需手动更改代码，只需安装插件。我们可以发现跨是连在一起的。我们可以得到一个客户电话的所有信息.
![包含kotlin插件](http://skywalking.apache.org/screenshots/7.0.0/kotlin/coroutine/run-with-coroutine-plugin.jpg)
