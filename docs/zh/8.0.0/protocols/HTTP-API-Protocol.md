# HTTP API协议

HTTP API协议定义了API数据格式，包括API请求和响应数据格式。
链路跟踪Trace数据采用 HTTP1.1格式，更详情内容请参见 [SkyWalking Trace Data Protocol v3](Trace-Data-Protocol-v3.md). 

## 实例管理

有关数据格式的详细信息参见 [Instance Management](https://github.com/apache/skywalking-data-collect-protocol/blob/master/management/Management.proto).

- 报表服务实例属性

> POST http://localhost:12800/v3/management/reportProperties

Input:

```json
{
	"service": "User Service Name",
	"serviceInstance": "User Service Instance Name",
	"properties": [{
		"language": "Lua"
	}]
}
```

Output JSON Array:

```json
{}
```

- Service instance ping

> POST http://localhost:12800/v3/management/keepAlive

Input:

```json
{
	"service": "User Service Name",
	"serviceInstance": "User Service Instance Name"
}
```

OutPut:

```json
{}
```

## 跟踪报告

有关数据格式的详细信息参见 [Instance Management](https://github.com/apache/skywalking-data-collect-protocol/blob/master/language-agent/Tracing.proto).
有两种方式报告段数据，每个请求一个段或批量模式下的段数组。

### POST http://localhost:12800/v3/segment

Send a single segment object with JSON format.

Input:

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
 OutPut:
 
 ```json

```

### POST http://localhost:12800/v3/segments

Send a segment object list with JSON format.

Input:

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
 OutPut:
 
 ```json

```