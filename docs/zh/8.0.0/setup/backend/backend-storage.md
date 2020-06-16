# Backend存储
Skywalking存储可插拔，我们提供了以下存储解决方案，你可以通过更改'application.yml`轻松使用。

```yaml
storage:
  selector: ${SW_STORAGE:elasticsearch7}
```

原生支持的存储
- H2
- ElasticSearch 6, 7
- MySQL
- TiDB
- InfluxDB

支持存储的重分发版本。
- ElasticSearch 5

## H2
要激活h2作为存储，设置存储提供者为内存中的 **H2**。默认的在发布的包中。
请阅读[h2官方文件]（http://www.h2 database.com/html/features.html）中的`Database URL Overview`，
你可以在**Embedded**, **Server** and **Mixed** 模式下将目标设置为H2。

设置如下
```yaml
storage:
  selector: ${SW_STORAGE:h2}
  h2:
    driver: org.h2.jdbcx.JdbcDataSource
    url: jdbc:h2:mem:skywalking-oap-db
    user: sa
```

## ElasticSearch
- 为了激活ElasticSearch 6作为存储，将存储提供程序设置为 **elasticsearch**
- 为了激活ElasticSearch 7作为存储，将存储提供程序设置为 **elasticsearch7**

**需要ElasticSearch 6.3.2或更高版本。HTTP RestHighLevelClient被用来连接服务器**

- ElasticSearch 6.3.2 ~ 7.0.0 (包含), 请下载 `apache-skywalking-bin.tar.gz` or `apache-skywalking-bin.zip`,
- ElasticSearch 7.0.0 ~ 8.0.0 (包含), 请下载 `apache-skywalking-bin-es7.tar.gz` or `apache-skywalking-bin-es7.zip`.

ElasticSearch 6或7 共享相同的配置，如下所示:

```yaml
storage:
  selector: ${SW_STORAGE:elasticsearch}
  elasticsearch:
    nameSpace: ${SW_NAMESPACE:""}
    clusterNodes: ${SW_STORAGE_ES_CLUSTER_NODES:localhost:9200}
    protocol: ${SW_STORAGE_ES_HTTP_PROTOCOL:"http"}
    trustStorePath: ${SW_SW_STORAGE_ES_SSL_JKS_PATH:""}
    trustStorePass: ${SW_SW_STORAGE_ES_SSL_JKS_PASS:""}
    user: ${SW_ES_USER:""}
    password: ${SW_ES_PASSWORD:""}
    secretsManagementFile: ${SW_ES_SECRETS_MANAGEMENT_FILE:""} # Secrets management file in the properties format includes the username, password, which are managed by 3rd party tool.
    dayStep: ${SW_STORAGE_DAY_STEP:1} # Represent the number of days in the one minute/hour/day index.
    indexShardsNumber: ${SW_STORAGE_ES_INDEX_SHARDS_NUMBER:2}
    indexReplicasNumber: ${SW_STORAGE_ES_INDEX_REPLICAS_NUMBER:0}
    # Batch process setting, refer to https://www.elastic.co/guide/en/elasticsearch/client/java-api/5.5/java-docs-bulk-processor.html
    bulkActions: ${SW_STORAGE_ES_BULK_ACTIONS:1000} # Execute the bulk every 1000 requests
    flushInterval: ${SW_STORAGE_ES_FLUSH_INTERVAL:10} # flush the bulk every 10 seconds whatever the number of requests
    concurrentRequests: ${SW_STORAGE_ES_CONCURRENT_REQUESTS:2} # the number of concurrent requests
    resultWindowMaxSize: ${SW_STORAGE_ES_QUERY_MAX_WINDOW_SIZE:10000}
    metadataQueryMaxSize: ${SW_STORAGE_ES_QUERY_MAX_SIZE:5000}
    segmentQueryMaxSize: ${SW_STORAGE_ES_QUERY_SEGMENT_SIZE:200}
    profileTaskQueryMaxSize: ${SW_STORAGE_ES_QUERY_PROFILE_TASK_SIZE:200}
    advanced: ${SW_STORAGE_ES_ADVANCED:""}
```

为了使用ElasticSearch 7，注释/删除“storage/ ElasticSearch”部分，并找到相应的配置部分(“storage/elasticsearch7”)，
取消注释以启用它。

### ElasticSearch 6与Https SSL加密通信.

example: 

```yaml
storage:
  selector: ${SW_STORAGE:elasticsearch}
  elasticsearch:
    # nameSpace: ${SW_NAMESPACE:""}
    user: ${SW_ES_USER:""} # User needs to be set when Http Basic authentication is enabled
    password: ${SW_ES_PASSWORD:""} # Password to be set when Http Basic authentication is enabled
    clusterNodes: ${SW_STORAGE_ES_CLUSTER_NODES:localhost:443}
    trustStorePath: ${SW_SW_STORAGE_ES_SSL_JKS_PATH:"../es_keystore.jks"}
    trustStorePass: ${SW_SW_STORAGE_ES_SSL_JKS_PASS:""}
    protocol: ${SW_STORAGE_ES_HTTP_PROTOCOL:"https"}
    indexShardsNumber: ${SW_STORAGE_ES_INDEX_SHARDS_NUMBER:2}
    indexReplicasNumber: ${SW_STORAGE_ES_INDEX_REPLICAS_NUMBER:0}
    # Batch process setting, refer to https://www.elastic.co/guide/en/elasticsearch/client/java-api/5.5/java-docs-bulk-processor.html
    bulkActions: ${SW_STORAGE_ES_BULK_ACTIONS:2000} # Execute the bulk every 2000 requests
    bulkSize: ${SW_STORAGE_ES_BULK_SIZE:20} # flush the bulk every 20mb
    flushInterval: ${SW_STORAGE_ES_FLUSH_INTERVAL:10} # flush the bulk every 10 seconds whatever the number of requests
    concurrentRequests: ${SW_STORAGE_ES_CONCURRENT_REQUESTS:2} # the number of concurrent requests
    advanced: ${SW_STORAGE_ES_ADVANCED:""}
```
- 文件在' trustStorePath '被监听，一旦它被改变，ElasticSearch客户端将做重新连接.
- `trustStorePass` 可以在运行时进行更改[**ElasticSearch认证的秘密管理文件**](#secrets-management-file-of-elasticsearch-authentication).

### 每日指数时间配置
每日指数(`storage/elasticsearch/dayStep`, default 1) 表示索引创建期间。在此期间，将保存几天(dayStep值)的度量.

大多数情况下，用户不需要手动更改值。由于SkyWalking是为了观测大型分布式系统而设计的。
但是在某些特定的情况下，用户希望设置一个很长的TTL值，比如超过60天，但是由于生产环境中的低流量，他们的ElasticSearch集群并不强大。
这个值可以增加到5(或更多)，如果用户能够确保单个索引能够支持最近的指标和跟踪(本例中是5)。

例如，if dayStep == 11, 
1. 数据  [2000-01-01, 2000-01-11] 将被写入 index-20000101.
2. 数据  [2000-01-12, 2000-01-22] 将被写入 index-20000112.

注意，TTL删除会受到这些因素的影响。你应该在TTL中增加一天的步数。例如您想要TTL == 30天和dayStep == 10，您实际上需要设置TTL = 40

### ElasticSearch认证的秘密管理文件
“secretsManagementFile”的值应该指向秘密管理文件的绝对路径。
该文件以属性格式包含ElasticSearch服务器的用户名、密码和JKS密码。
```properties
user=xxx
password=yyy
trustStorePass=zzz
```

在应用程序中使用' user, password, trustStorePass '配置的主要区别。yaml的文件是，OAP服务器正在监视**Secrets管理文件**。
通过第三方工具，如[Vault](https://github.com/hashicorp/vault)，
存储提供程序将使用新的用户名、密码和JKS密码来建立连接并关闭旧的连接。如果信息存在于文件中，
“用户/密码”将被覆盖。


### 弹性搜索索引的高级配置

您可以添加“JSON”格式的高级配置，以设置“ElasticSearch索引设置”遵循[ElasticSearch doc](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-modules.html)

例如, set [translog](https://www.elastic.co/guide/en/elasticsearch/reference/master/index-modules-translog.html) settings:

```yaml
storage:
  elasticsearch:
    # ......
    advanced: ${SW_STORAGE_ES_ADVANCED:"{\"index.translog.durability\":\"request\",\"index.translog.sync_interval\":\"5s\"}"}
```

### 建议的ElasticSearch服务器端配置
您可以添加以下配置到' elasticsearch。，根据你的环境设置值。

```yml
# In tracing scenario, consider to set more than this at least.
thread_pool.index.queue_size: 1000 # Only suitable for ElasticSearch 6
thread_pool.write.queue_size: 1000 # Suitable for ElasticSearch 6 and 7

# When you face query error at trace page, remember to check this.
index.max_result_window: 1000000
```

我们强烈建议您从ElasticSearch官方文档中阅读更多关于这些配置的信息。
这对ElasticSearch的性能影响很大。


### ElasticSearch 6与Zipkin扩展
这个实现共享了大部分的“elasticsearch”，只是扩展到支持zipkin span存储。
它有相同的配置。

```yaml
storage:
  selector: ${SW_STORAGE:zipkin-elasticsearch}
  zipkin-elasticsearch:
    nameSpace: ${SW_NAMESPACE:""}
    clusterNodes: ${SW_STORAGE_ES_CLUSTER_NODES:localhost:9200}
    protocol: ${SW_STORAGE_ES_HTTP_PROTOCOL:"http"}
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

### ElasticSearch 6与Jaeger扩展
这个实现共享了大部分的“elasticsearch”，只是扩展到支持Jaeger存储。
它有相同的配置。
```yaml
storage:
  selector: ${SW_STORAGE:jaeger-elasticsearch}
  jaeger-elasticsearch:
    nameSpace: ${SW_NAMESPACE:""}
    clusterNodes: ${SW_STORAGE_ES_CLUSTER_NODES:localhost:9200}
    protocol: ${SW_STORAGE_ES_HTTP_PROTOCOL:"http"}
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


### 关于命名空间
当设置名称空间时，ElasticSearch中所有索引的名称将使用它作为前缀.


## MySQL
MySQL作为存储，设置存储提供程序为 **mysql**. 

**注意:** MySQL驱动程序不允许出现在Apache官方发行版和源代码中。
请自行下载MySQL驱动。将连接驱动程序jar复制到' oap-libs '。

```yaml
storage:
  selector: ${SW_STORAGE:mysql}
  mysql:
    properties:
      jdbcUrl: ${SW_JDBC_URL:"jdbc:mysql://localhost:3306/swtest"}
      dataSource.user: ${SW_DATA_SOURCE_USER:root}
      dataSource.password: ${SW_DATA_SOURCE_PASSWORD:root@1234}
      dataSource.cachePrepStmts: ${SW_DATA_SOURCE_CACHE_PREP_STMTS:true}
      dataSource.prepStmtCacheSize: ${SW_DATA_SOURCE_PREP_STMT_CACHE_SQL_SIZE:250}
      dataSource.prepStmtCacheSqlLimit: ${SW_DATA_SOURCE_PREP_STMT_CACHE_SQL_LIMIT:2048}
      dataSource.useServerPrepStmts: ${SW_DATA_SOURCE_USE_SERVER_PREP_STMTS:true}
    metadataQueryMaxSize: ${SW_STORAGE_MYSQL_QUERY_MAX_SIZE:5000}
```
所有连接相关的设置，包括链接url，用户名和密码都在' application.yml '中。
其他一些设置，请按照[HikariCP](https://github.com/brettwooldridge/HikariCP)连接池文档进行所有设置。

## TiDB
目前测试的TiDB版本为2.0.9,Mysql客户端驱动程序为8.0.13。
TiDB作为存储，设置存储提供程序为**mysql**。

```yaml
storage:
  selector: ${SW_STORAGE:mysql}
  mysql:
    properties:
      jdbcUrl: ${SW_JDBC_URL:"jdbc:mysql://localhost:3306/swtest"}
      dataSource.user: ${SW_DATA_SOURCE_USER:root}
      dataSource.password: ${SW_DATA_SOURCE_PASSWORD:root@1234}
      dataSource.cachePrepStmts: ${SW_DATA_SOURCE_CACHE_PREP_STMTS:true}
      dataSource.prepStmtCacheSize: ${SW_DATA_SOURCE_PREP_STMT_CACHE_SQL_SIZE:250}
      dataSource.prepStmtCacheSqlLimit: ${SW_DATA_SOURCE_PREP_STMT_CACHE_SQL_LIMIT:2048}
      dataSource.useServerPrepStmts: ${SW_DATA_SOURCE_USE_SERVER_PREP_STMTS:true}
    metadataQueryMaxSize: ${SW_STORAGE_MYSQL_QUERY_MAX_SIZE:5000}
```
所有连接相关的设置，包括链接url，用户名和密码都在' application.yml '中。
这些设置可以参考上面的*MySQL*配置。

## InfluxDB
流感数据库存储提供了一个时间序列数据库作为新的存储选项。

```yaml
storage:
  selector: ${SW_STORAGE:influxdb}
  influxdb:
    url: ${SW_STORAGE_INFLUXDB_URL:http://localhost:8086}
    user: ${SW_STORAGE_INFLUXDB_USER:root}
    password: ${SW_STORAGE_INFLUXDB_PASSWORD:}
    database: ${SW_STORAGE_INFLUXDB_DATABASE:skywalking}
    actions: ${SW_STORAGE_INFLUXDB_ACTIONS:1000} # the number of actions to collect
    duration: ${SW_STORAGE_INFLUXDB_DURATION:1000} # the time to wait at most (milliseconds)
    fetchTaskLogMaxSize: ${SW_STORAGE_INFLUXDB_FETCH_TASK_LOG_MAX_SIZE:5000} # the max number of fetch task log in a request
```
所有连接相关的设置，包括链接url，用户名和密码都在' application.yml '中。元数据存储提供程序的设置可以参考上面的**H2/MySQL**的配置。

## ElasticSearch 5
ElasticSearch 5 和ElasticSearch 6的java客户端版本不兼容, 所有它不在原生发布中。.
[OpenSkywalking/SkyWalking-With-Es5x-Storage](https://github.com/OpenSkywalking/SkyWalking-With-Es5x-Storage) 包括这个发布版本。

## 更多存储解决方案扩展
参考开发者指南[Project Extensions document](../../guides/README.md#project-extensions)中的[Storage extension development guide](../../guides/storage-extention.md) 
