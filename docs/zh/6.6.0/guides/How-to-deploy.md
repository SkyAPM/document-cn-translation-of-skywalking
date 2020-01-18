# 版本下载
```
安装包名: apache-skywalking-apm-es7-6.6.0.zip
下载地址: http://skywalking.apache.org/downloads/

wget http://mirror.bit.edu.cn/apache/skywalking/6.6.0/apache-skywalking-apm-es7-6.6.0.zip
```

# 部署
```
cd /opt/
unzip apache-skywalking-apm-es7-6.6.0.zip
```

# 配置
## 服务端配置
```
默认使用H2内存数据库，这里改成mysql，存放监控数据

由于license冲突，skywalking的license不含mysql的依赖包，需要自行下载：
cd oap-libs
wget http://central.maven.org/maven2/mysql/mysql-connector-java/5.1.48/mysql-connector-java-5.1.48.jar

安装mysql并设置账号密码test/123456（略）
```

## 后端存储
```
创建一个空的swtest数据库即可,sql:
CREATE DATABASE `swtest` /*!40100 DEFAULT CHARACTER SET utf8 */

sky服务端配置：
cat /opt/apache-skywalking-apm-bin-es7/conf/application.yml
  mysql:
    properties:
      jdbcUrl: ${SW_JDBC_URL:"jdbc:mysql://localhost:3306/swtest"}
      dataSource.user: ${SW_DATA_SOURCE_USER:test}
      dataSource.password: ${SW_DATA_SOURCE_PASSWORD:123456}
```

## 采集端配置:
```
所有服务在本地搭建无需配置
```

# 监控面板
```
按照上述步骤，本地启动浏览器打开(skywalking服务器ip假定为x.x.x.x)：
http://x.x.x.x:8080/
```

# 链路数据采集
## 下载采集服务：
```
cd /opt/
wget http://mirror.bit.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.zip
unzip apache-maven-3.6.3-bin.zip

git clone https://github.com/SkyAPMTest/spring-cloud-example.git 
cd spring-cloud-example
/opt/apache-maven-3.6.3-bin/bin/mvn clean package
```

## 启动微服务
```
nohup java -javaagent:/opt/apache-skywalking-apm-bin-es7/agent/skywalking-agent.jar -Dskywalking.agent.service_name=eureka -jar /opt/spring-cloud-example/projectA/target/eureka-service-0.0.1-SNAPSHOT.jar  2>&1 > /opt/logs/eureka.log &

nohup java -javaagent:/opt/apache-skywalking-apm-bin-es7/agent/skywalking-agent.jar -Dskywalking.agent.service_name=projecta -jar /opt/spring-cloud-example/projectA/target/projectA-1.0-SNAPSHOT.jar  2>&1 > /opt/logs/projecta.log &

nohup java -javaagent:/opt/apache-skywalking-apm-bin-es7/agent/skywalking-agent.jar -Dskywalking.agent.service_name=projectb -jar /opt/spring-cloud-example/projectA/target/projectB-1.0-SNAPSHOT.jar  2>&1 > /opt/logs/projectb.log &

nohup java -javaagent:/opt/apache-skywalking-apm-bin-es7/agent/skywalking-agent.jar -Dskywalking.agent.service_name=projectc -jar /opt/spring-cloud-example/projectA/target/projectC-1.0-SNAPSHOT.jar  2>&1 > /opt/logs/projectc.log &
```

## 调用接口产生流量
```
curl http://localhost:8764/projectA/aaa
curl http://localhost:8762/projectB/bbb
curl http://localhost:8763/projectC/ccc
```

## 查看链路追踪效果
```
在windows环境打开浏览器，访问:
http://x.x.x.x:8080/trace
```
