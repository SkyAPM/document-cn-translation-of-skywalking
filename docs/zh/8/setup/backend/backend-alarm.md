# Alarm
Alarm core is driven by a collection of rules, which are defined in `config/alarm-settings.yml`.
There are three parts in alarm rule definition.
1. [Alarm rules](#rules). They define how metrics alarm should be triggered, what conditions should be considered.
1. [Webhooks](#webhook). The list of web service endpoint, which should be called after the alarm is triggered.
1. [gRPCHook](#gRPCHook). The host and port of remote gRPC method, which should be called after the alarm is triggered.

## Entity name
Define the relation between scope and entity name.
- **Service**: Service name
- **Instance**: {Instance name} of {Service name}
- **Endpoint**: {Endpoint name} in {Service name}
- **Database**: Database service name
- **Service Relation**: {Source service name} to {Dest service name}
- **Instance Relation**: {Source instance name} of {Source service name} to {Dest instance name} of {Dest service name}
- **Endpoint Relation**: {Source endpoint name} in {Source Service name} to {Dest endpoint name} in {Dest service name}

## Rules
Alarm rule is constituted by following keys
- **Rule name**. Unique name, show in alarm message. Must end with `_rule`.
- **Metrics name**. A.K.A. metrics name in oal script. Only long, double, int types are supported. See
[List of all potential metrics name](#list-of-all-potential-metrics-name).
- **Include names**. The following entity names are included in this rule. Please follow [Entity name define](#entity-name).
- **Exclude names**. The following entity names are excluded in this rule. Please follow [Entity name define](#entity-name).
- **Include names regex**. Provide a regex to include the entity names. If both setting the include name list and include name regex, both rules will take effect.
- **Exclude names regex**. Provide a regex to exclude the exclude names. If both setting the exclude name list and exclude name regex, both rules will take effect.
- **Threshold**. The target value. 
For multiple values metrics, such as **percentile**, the threshold is an array. Described like  `value1, value2, value3, value4, value5`.
Each value could the threshold for each value of the metrics. Set the value to `-` if don't want to trigger alarm by this or some of the values.  
Such as in **percentile**, `value1` is threshold of P50, and `-, -, value3, value4, value5` means, there is no threshold for P50 and P75 in percentile alarm rule.
- **OP**. Operator, support `>`, `>=`, `<`, `<=`, `=`. Welcome to contribute all OPs.
- **Period**. How long should the alarm rule should be checked. This is a time window, which goes with the
backend deployment env time.
- **Count**. In the period window, if the number of **value**s over threshold(by OP), reaches count, alarm
should send.
- **Silence period**. After alarm is triggered in Time-N, then keep silence in the **TN -> TN + period**.
By default, it is as same as **Period**, which means in a period, same alarm(same ID in same 
metrics name) will be trigger once. 


```yaml
rules:
  # Rule unique name, must be ended with `_rule`.
  endpoint_percent_rule:
    # Metrics value need to be long, double or int
    metrics-name: endpoint_percent
    threshold: 75
    op: <
    # The length of time to evaluate the metrics
    period: 10
    # How many times after the metrics match the condition, will trigger alarm
    count: 3
    # How many times of checks, the alarm keeps silence after alarm triggered, default as same as period.
    silence-period: 10
  service_percent_rule:
    metrics-name: service_percent
    # [Optional] Default, match all services in this metrics
    include-names:
      - service_a
      - service_b
    exclude-names:
      - service_c
    # Single value metrics threshold.
    threshold: 85
    op: <
    period: 10
    count: 4
  service_resp_time_percentile_rule:
    # Metrics value need to be long, double or int
    metrics-name: service_percentile
    op: ">"
    # Multiple value metrics threshold. Thresholds for P50, P75, P90, P95, P99.
    threshold: 1000,1000,1000,1000,1000
    period: 10
    count: 3
    silence-period: 5
    message: Percentile response time of service {name} alarm in 3 minutes of last 10 minutes, due to more than one condition of p50 > 1000, p75 > 1000, p90 > 1000, p95 > 1000, p99 > 1000
```

### Default alarm rules
We provided a default `alarm-setting.yml` in our distribution only for convenience, which including following rules
1. Service average response time over 1s in last 3 minutes.
1. Service success rate lower than 80% in last 2 minutes.
1. Percentile of service response time is over 1s in last 3 minutes
1. Service Instance average response time over 1s in last 2 minutes, and the instance name matches the regex.
1. Endpoint average response time over 1s in last 2 minutes.
1. Database access average response time over 1s in last 2 minutes.
1. Endpoint relation average response time over 1s in last 2 minutes.

### List of all potential metrics name
The metrics names are defined in official [OAL scripts](../../guides/backend-oal-scripts.md), right now 
metrics from **Service**, **Service Instance**, **Endpoint**, **Service Relation**, **Service Instance Relation**, **Endpoint Relation** scopes could be used in Alarm, and the **Database access** same with **Service** scope.

Submit issue or pull request if you want to support any other scope in alarm.

## Webhook
Webhook requires the peer is a web container. The alarm message will send through HTTP post by `application/json` content type. The JSON format is based on `List<org.apache.skywalking.oap.server.core.alarm.AlarmMessage>` with following key information.
- **scopeId**, **scope**. All scopes are defined in org.apache.skywalking.oap.server.core.source.DefaultScopeDefine.
- **name**. Target scope entity name. Please follow [Entity name define](#entity-name).
- **id0**. The ID of the scope entity matched the name. When using relation scope, it is the source entity ID.
- **id1**. When using relation scope, it will be the dest entity ID. Otherwise, it is empty.
- **ruleName**. The rule name you configured in `alarm-settings.yml`.
- **alarmMessage**. Alarm text message.
- **startTime**. Alarm time measured in milliseconds, between the current time and midnight, January 1, 1970 UTC.

Example as following
```json
[{
	"scopeId": 1, 
	"scope": "SERVICE",
	"name": "serviceA", 
	"id0": "12",  
	"id1": "",  
    "ruleName": "service_resp_time_rule",
	"alarmMessage": "alarmMessage xxxx",
	"startTime": 1560524171000
}, {
	"scopeId": 1,
	"scope": "SERVICE",
	"name": "serviceB",
	"id0": "23",
	"id1": "",
    "ruleName": "service_resp_time_rule",
	"alarmMessage": "alarmMessage yyy",
	"startTime": 1560524171000
}]
```

## gRPCHook
The alarm message will send through remote gRPC method by `Protobuf` content type. 
The message format with following key information which are defined in `oap-server/server-alarm-plugin/src/main/proto/alarm-hook.proto`.

Part of protocol looks as following:
```protobuf
message AlarmMessage {
    int64 scopeId = 1;
    string scope = 2;
    string name = 3;
    string id0 = 4;
    string id1 = 5;
    string ruleName = 6;
    string alarmMessage = 7;
    int64 startTime = 8;
}
```

## Update the settings dynamically
Since 6.5.0, the alarm settings can be updated dynamically at runtime by [Dynamic Configuration](dynamic-config.md),
which will override the settings in `alarm-settings.yml`.

In order to determine that whether an alarm rule is triggered or not, SkyWalking needs to cache the metrics of a time window for
each alarm rule, if any attribute (`metrics-name`, `op`, `threshold`, `period`, `count`, etc.) of a rule is changed,
the sliding window will be destroyed and re-created, causing the alarm of this specific rule to restart again.
