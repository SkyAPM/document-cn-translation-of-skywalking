# 服务器端的跟踪采样

当我们运行一个分布式跟踪系统时，跟踪会给我们带来详细的信息，但在存储消耗很大。开放服务器端跟踪采样机制，
服务度量、服务实例、端点和拓扑和以前一样，都是准确的，只是不把所有的跟踪数据存起来。

当然，即使您打开采样，跟踪也会尽可能保持一致。**一致**意味着，一旦跟踪段已被agent收集并报告，
后端将尽力不破坏跟踪。查看[推荐](#推荐)来了解我们称它为`as consistent as possible`和`do their best to don't break the trace`的原因。

## 设定采样率

在**receiver-trace**接受者中，你可以找到`sampleRate`设置。

```yaml
receiver-trace:
  default:
    ...
    sampleRate: ${SW_TRACE_SAMPLE_RATE:10000} # 采样率精度为1/10000。 10000表示默认为100％采样。
```

`sampleRate` 可让你设置Backend的采样率。
采样率精度为1/10000。10000表示默认为100%样本。 

## 推荐

您可以给的不同的后端实例设置不同的'sampleRate'值，但我们建议你设置成一样。

当你设置不同的值时，

* Backend-Instance**A**.sampleRate = 35
* Backend-Instance**B**.sampleRate = 55

我们假设agent向Backend报告了所有跟踪段，然后，全局35%的跟踪将被收集并保存在一致/完整的存储中，所有跨度都是如此。
报告给后端实例**B**的20%跟踪段也将保存在存储器中，但可能会遗漏一些跟踪段， 因为它们被报告给后端实例**A**并被忽略。
