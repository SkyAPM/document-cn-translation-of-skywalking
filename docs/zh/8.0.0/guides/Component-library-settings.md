# 组件库设置

组件库设置文件是关于如何在要监控的应用中使用自有的或第三方库的介绍.

在代理或 SDK 中, 无论库名是以 ID 形式还是字符串(文本, 如 SpringMVC)形式进行收集的,
收集器都会将数据格式化为 ID, 以提供更好的性能和更低的存储需求.

同时, 收集器还会根据组件库来推测远程服务类型, 比如: 如果组件库是 MySQL 驱动库, 那么
收集器则会推测远程服务是一个 MySQL 服务器.

基于以上两方面原因, 收集器要求在组件库设置文件中必须要有以下两部分配置:

1. 组件库 ID, 名称以及语言.
2. 基于本地库的远程服务器映射.

**所有组件名称及 ID 都必须在组件库设置文件中定义.**

## 组件库 ID

组件库 ID 定义了在要监控的应用程序中使用到的所有组件库的名称和 ID
这是一种双向映射, 代理或 SDK 可以在上行数据中使用 ID 值来代表对应的组件.

- 名称: 在代理和用户界面中使用的组件名称
- id: 唯一标识. 所有使用过的 ID 在发布后都将会被保留(不可再利用)
- 语言: 编程语言可能使用了这个组件, 如果有多种语言应该使用 `,` 分隔

### ID 规则

- Java 和多种语言共享的: (0, 3000]
- .NET 平台保留: (3000, 4000]
- Node.js 平台保留: (4000, 5000]
- Go 语言保留: (5000, 6000]
- Lua reserved: [6000, 7000)
- Python reserved: [7000, 8000)
- PHP reserved: [8000, 9000)

示例:

```yaml
Tomcat:
  id: 1
  languages: Java
HttpClient:
  id: 2
  languages: Java,C#,Node.js
Dubbo:
  id: 3
  languages: Java
H2:
  id: 4
  languages: Java
```

## 远程服务器映射

远程服务类型会通过本地库进行推断. 此推断映射是基于组件库名称的.

- 键: 客户端组件库名称
- 值: 服务端组件名称

```yaml
Component-Server-Mappings:
  Jedis: Redis
  StackExchange.Redis: Redis
  Redisson: Redis
  Lettuce: Redis
  Zookeeper: Zookeeper
  SqlClient: SqlServer
  Npgsql: PostgreSQL
  MySqlConnector: Mysql
  EntityFrameworkCore.InMemory: InMemoryDatabase
```
