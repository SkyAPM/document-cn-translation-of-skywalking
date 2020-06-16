# SkyWalking跨进程传播的头部协议
* 版本 1.0

跨进程传播的头部协议用于通过利用以下功能来传输自定义数据：[跨进程传播的头部协议](Skywalking-Cross-Process-Propagation-Headers-Protocol-v3.md). 

这是用于语言跟踪程序实现的可选和附加协议。所有跟踪器实现都可以考虑实现此目的。
跨进程相关标头密钥为 `sw8-correlation`。该值是 `encoded(key):encoded(value)` 列表，其中元素由 `,` 分割, 例如 `base64(string key):base64(string value),base64(string key2):base64(string value2)`.

## 语言API的建议
推荐的不同语言API实现。

1. `TraceContext#putCorrelation` 和 `TraceContext#getCorrelation` 建议使用键/值字符串编写和读取关联上下文。
2. 如果不存在，则应添加密钥。
3. 以后的写入应覆盖先前的值。
4. 所有键的总数应小于3，每个值的长度应小于128字节。
5. 当跟踪上下文跨线程和进程传播时，上下文也应传播。
