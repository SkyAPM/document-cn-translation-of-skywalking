# 应用性能指数


应用性能指数(APDEX)是基于设置阈值的响应时间度量。它表示令人满意与不满意的响应时间比率。响应时间是指从一个请求到返回的完整请求。
 
用户定义一个响应时间阈值T，在T或更短时间内处理的所有用户的请求响应。
 
举个例子, 如果T是1.2秒，响应在0.5秒内完成，那么用户就满意了。所有响应
大于1.2秒的用户不满意请求。超过4.8(默认T*4)秒的响应会让用户感到沮丧.

指标值T定义在文件'service-apdex-threshold.yml'或者 查看[动态配置](dynamic-config.md). 
“default”项将应用配置中未定义的服务.

## 配置格式

配置必须包含服务名称和相关的维度定义:

```yml
# default threshold is 500ms
default: 500
# example:
# the threshold of service "tomcat" is 1s
# tomcat: 1000
# the threshold of service "springboot1" is 50ms
# springboot1: 50
```
