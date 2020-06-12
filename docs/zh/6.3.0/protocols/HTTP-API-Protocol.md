# HTTP API 协议
HTTP API协议定义了 API 数据格式，包括 API 请求和响应数据格式.
它们封装了官方的 HTTP1.1. 详情请阅读 [SkyWalking Trace Data Protocol v3](Trace-Data-Protocol-v3.md).

## 实例管理

有关数据格式的详细信息可前往 [Instance Management](https://github.com/apache/skywalking-data-collect-protocol/blob/master/management/Management.proto).

- 上报服务实例属性

> POST http://localhost:12800/v3/management/reportProperties

输入:

```json
{
	"service": "User Service Name",
	"serviceInstance": "User Service Instance Name",
	"properties": [{
		"language": "Lua"
	}]
}
```

输出 JSON 数组:

```json
{}
```

- ping 服务实例

> POST http://localhost:12800/v3/management/keepAlive

输入:

```json
{
	"service": "User Service Name",
	"serviceInstance": "User Service Instance Name"
}
```

输出:

```json
{}
```

## 追踪上报

有关数据格式的详细信息可前往 [Instance Management](https://github.com/apache/skywalking-data-collect-protocol/blob/master/language-agent/Tracing.proto).
上报追踪段数据有两种方法，上报一个追踪段或批量上报追踪段数组.

### POST http://localhost:12800/v3/segment

发送一个JSON格式的追踪段对象.

输入:

```json
{
	"traceId": "a12ff60b-5807-463b-a1f8-fb1c8608219e",
	"serviceInstance": "User_Service_Instance_Name",
	"spans": [{
		"operationName": "/ingress",
		"startTime": 1588664577013,
		"endTime": 1588664577028,
		"spanType": "Exit",
		"spanId": 1,
		"isError": false,
		"parentSpanId": 0,
		"componentId": 6000,
		"peer": "upstream service",
		"spanLayer": "Http"
	}, {
		"operationName": "/ingress",
		"startTime": 1588664577013,
		"tags": [{
			"key": "http.method",
			"value": "GET"
		}, {
			"key": "http.params",
			"value": "http://localhost/ingress"
		}],
		"endTime": 1588664577028,
		"spanType": "Entry",
		"spanId": 0,
		"parentSpanId": -1,
		"isError": false,
		"spanLayer": "Http",
		"componentId": 6000
	}],
	"service": "User_Service_Name",
	"traceSegmentId": "a12ff60b-5807-463b-a1f8-fb1c8608219e"
}
```
 输出:
 
 ```json

```

### POST http://localhost:12800/v3/segments

发送一个JSON格式的追踪段对象列表.

输入:

```json
[{
	"traceId": "a12ff60b-5807-463b-a1f8-fb1c8608219e",
	"serviceInstance": "User_Service_Instance_Name",
	"spans": [{
		"operationName": "/ingress",
		"startTime": 1588664577013,
		"endTime": 1588664577028,
		"spanType": "Exit",
		"spanId": 1,
		"isError": false,
		"parentSpanId": 0,
		"componentId": 6000,
		"peer": "upstream service",
		"spanLayer": "Http"
	}, {
		"operationName": "/ingress",
		"startTime": 1588664577013,
		"tags": [{
			"key": "http.method",
			"value": "GET"
		}, {
			"key": "http.params",
			"value": "http://localhost/ingress"
		}],
		"endTime": 1588664577028,
		"spanType": "Entry",
		"spanId": 0,
		"parentSpanId": -1,
		"isError": false,
		"spanLayer": "Http",
		"componentId": 6000
	}],
	"service": "User_Service_Name",
	"traceSegmentId": "a12ff60b-5807-463b-a1f8-fb1c8608219e"
}, {
	"traceId": "f956699e-5106-4ea3-95e5-da748c55bac1",
	"serviceInstance": "User_Service_Instance_Name",
	"spans": [{
		"operationName": "/ingress",
		"startTime": 1588664577250,
		"endTime": 1588664577250,
		"spanType": "Exit",
		"spanId": 1,
		"isError": false,
		"parentSpanId": 0,
		"componentId": 6000,
		"peer": "upstream service",
		"spanLayer": "Http"
	}, {
		"operationName": "/ingress",
		"startTime": 1588664577250,
		"tags": [{
			"key": "http.method",
			"value": "GET"
		}, {
			"key": "http.params",
			"value": "http://localhost/ingress"
		}],
		"endTime": 1588664577250,
		"spanType": "Entry",
		"spanId": 0,
		"parentSpanId": -1,
		"isError": false,
		"spanLayer": "Http",
		"componentId": 6000
	}],
	"service": "User_Service_Name",
	"traceSegmentId": "f956699e-5106-4ea3-95e5-da748c55bac1"
}]
```
 输出:
 
 ```json

```