# Backend存储
Skywalking存储可插拔，我们提供了以下存储解决方案，你可以通过更改'application.yml`轻松使用。

原生支持的存储e
- H2
- ElasticSearch 6
- MySQL
- TiDB

支持存储的重分发版本。
- ElasticSearch 5


## H2
要激活h2作为存储，设置存储提供者为内存中的 **H2**。默认的在发布的包中。
请阅读[h2官方文件]（http://www.h2 database.com/html/features.html）中的`Database URL Overview`，您可以在**Embedded**, **Server** and **Mixed** 模式下将目标设置为H2。

设置片段示例
```yaml
storage:
  h2:
    driver: org.h2.jdbcx.JdbcDataSource
    url: jdbc:h2:mem:skywalking-oap-db
    user: sa
```

## ElasticSearch 6
要激活ElasticSearch 6作为存储, 设置存储提供者为**elasticsearch**。

**需要ElasticSearch 6.3.0或更高版本。HTTP RestHighLevelClient被用来连接服务器**

设置片段示例

```yaml
storage:
  elasticsearch:
    # nameSpace: ${SW_NAMESPACE:""}
    # user: ${SW_ES_USER:""} # User needs to be set when Http Basic authentication is enabled
    # password: ${SW_ES_PASSWORD:""} # Password to be set when Http Basic authentication is enabled
    clusterNodes: ${SW_STORAGE_ES_CLUSTER_NODES:localhost:9200}
    indexShardsNumber: ${SW_STORAGE_ES_INDEX_SHARDS_NUMBER:2}
    indexReplicasNumber: ${SW_STORAGE_ES_INDEX_REPLICAS_NUMBER:0}
    # Batch process setting, refer to https://www.elastic.co/guide/en/elasticsearch/client/java-api/5.5/java-docs-bulk-processor.html
    bulkActions: ${SW_STORAGE_ES_BULK_ACTIONS:2000} # Execute the bulk every 2000 requests
    bulkSize: ${SW_STORAGE_ES_BULK_SIZE:20} # flush the bulk every 20mb
    flushInterval: ${SW_STORAGE_ES_FLUSH_INTERVAL:10} # flush the bulk every 10 seconds whatever the number of requests
    concurrentRequests: ${SW_STORAGE_ES_CONCURRENT_REQUESTS:2} # the number of concurrent requests
```

### ElasticSearch 6 with Zipkin trace extension
这个实现共享了大部分的 `elasticsearch`，只是扩展到支持Zipkin-Span存储。它有所以相同的配置。
```yaml
storage:
  zipkin-elasticsearch:
    nameSpace: ${SW_NAMESPACE:""}
    clusterNodes: ${SW_STORAGE_ES_CLUSTER_NODES:localhost:9200}
    user: ${SW_ES_USER:""}
    password: ${SW_ES_PASSWORD:""}
    indexShardsNumber: ${SW_STORAGE_ES_INDEX_SHARDS_NUMBER:2}
    indexReplicasNumber: ${SW_STORAGE_ES_INDEX_REPLICAS_NUMBER:0}
    # Batch process setting, refer to https://www.elastic.co/guide/en/elasticsearch/client/java-api/5.5/java-docs-bulk-processor.html
    bulkActions: ${SW_STORAGE_ES_BULK_ACTIONS:2000} # Execute the bulk every 2000 requests
    bulkSize: ${SW_STORAGE_ES_BULK_SIZE:20} # flush the bulk every 20mb
    flushInterval: ${SW_STORAGE_ES_FLUSH_INTERVAL:10} # flush the bulk every 10 seconds whatever the number of requests
    concurrentRequests: ${SW_STORAGE_ES_CONCURRENT_REQUESTS:2} # the number of concurrent requests
```

### ElasticSearch 6 with Jaeger trace extension
这个实现共享了大部分的 `elasticsearch`，只是扩展到支持Jaeger -Span存储。它有所以相同的配置。
```yaml
storage:
  jaeger-elasticsearch:
    nameSpace: ${SW_NAMESPACE:""}
    clusterNodes: ${SW_STORAGE_ES_CLUSTER_NODES:localhost:9200}
    user: ${SW_ES_USER:""}
    password: ${SW_ES_PASSWORD:""}
    indexShardsNumber: ${SW_STORAGE_ES_INDEX_SHARDS_NUMBER:2}
    indexReplicasNumber: ${SW_STORAGE_ES_INDEX_REPLICAS_NUMBER:0}
    # Batch process setting, refer to https://www.elastic.co/guide/en/elasticsearch/client/java-api/5.5/java-docs-bulk-processor.html
    bulkActions: ${SW_STORAGE_ES_BULK_ACTIONS:2000} # Execute the bulk every 2000 requests
    bulkSize: ${SW_STORAGE_ES_BULK_SIZE:20} # flush the bulk every 20mb
    flushInterval: ${SW_STORAGE_ES_FLUSH_INTERVAL:10} # flush the bulk every 10 seconds whatever the number of requests
    concurrentRequests: ${SW_STORAGE_ES_CONCURRENT_REQUESTS:2} # the number of concurrent requests
```


### About Namespace
设置名称空间后，ElasticSearch中所有索引的名称都将使用它作为前缀。

### About Authentication
我们只支持[basic authentication](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/6.6/_basic_authentication.html)。如果需要，可以设置`user` and `password`.。
有关如何启用HTTP基本身份验证，请阅读这个https://brudtkuhl.com/blog/securing-elasticsearch/。


## MySQL
要激活MySQL作为存储, 设置存储提供者为**mysql**。

**注意:** Apache官方发行版和源代码中不允许使用MySQL驱动程序。请自己下载MySQL驱动程序。将连接驱动程序jar复制到`oap-libs`。

```yaml
storage:
  mysql:
```

所有与连接相关的设置，包括链接URL、用户名和密码都在`datasource-settings.properties`。
设置文件参考[HikariCP](https://github.com/brettwooldridge/HikariCP) 连接池文档。.

## TiDB
目前的测试版tidb版本为2.0.9和MySQL客户端驱动版本为8.0.13。
要激活TiDB作为存储,设置存储提供者为**mysql**。

```yaml
storage:
  mysql:
```

所有与连接相关的设置，包括链接URL、用户名和密码都在`datasource-settings.properties`。所有配置都可参考上面的*MySQL*。

## ElasticSearch 5
ElasticSearch 5 和ElasticSearch 6的java客户端版本不兼容, 所有它不在原生发布中。.
[OpenSkywalking/SkyWalking-With-Es5x-Storage](https://github.com/OpenSkywalking/SkyWalking-With-Es5x-Storage) 包括这个发布版本。

## More storage solution extension
参考开发者指南[Project Extensions document](../../guides/README.md#project-extensions)中的[Storage extension development guide](../../guides/storage-extention.md) 
