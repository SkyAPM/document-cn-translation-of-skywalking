# ElasticSearch 
一些新用户可能会遇到* ElasticSearch性能不如预期的好。例如，一段时间后不能拥有最新数据。或 `ERROR CODE 429`。
```
    Suppressed: org.elasticsearch.client.ResponseException: method [POST], host [http://127.0.0.1:9200], URI [/service_instance_inventory/type/6_tcc-app-gateway-77b98ff6ff-crblx.cards_0_0/_update?refresh=true&timeout=1m], status line [HTTP/1.1 429 Too Many Requests]
{"error":{"root_cause":[{"type":"remote_transport_exception","reason":"[elasticsearch-0][10.16.9.130:9300][indices:data/write/update[s]]"}],"type":"es_rejected_execution_exception","reason":"rejected execution of org.elasticsearch.transport.TransportService$7@19a5cf02 on EsThreadPoolExecutor[name = elasticsearch-0/write, queue capacity = 200, org.elasticsearch.common.util.concurrent.EsThreadPoolExecutor@389297ad[Running, pool size = 2, active threads = 2, queued tasks = 200, completed tasks = 147611]]"},"status":429}
        at org.elasticsearch.client.RestClient$SyncResponseListener.get(RestClient.java:705) ~[elasticsearch-rest-client-6.3.2.jar:6.3.2]
        at org.elasticsearch.client.RestClient.performRequest(RestClient.java:235) ~[elasticsearch-rest-client-6.3.2.jar:6.3.2]
        at org.elasticsearch.client.RestClient.performRequest(RestClient.java:198) ~[elasticsearch-rest-client-6.3.2.jar:6.3.2]
        at org.elasticsearch.client.RestHighLevelClient.performRequest(RestHighLevelClient.java:522) ~[elasticsearch
```

此时可以添加以下配置到 `elasticsearch.yml` 中, 根据你的环境设置相应的值.
```yml
# In tracing scenario, consider to set more than this at least.
thread_pool.index.queue_size: 1000
thread_pool.write.queue_size: 1000

# When you face query error at trace page, remember to check this.
index.max_result_window: 1000000
```

阅读ElasticSearch官方文档以获取更多信息。
