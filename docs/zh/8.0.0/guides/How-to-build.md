# 如何构建项目

本文档将帮助你使用 maven 编译构建 SkyWalking 项目, 并指导你对 IDE 进行设置.

## 构建项目

**由于本项目使用了 Git 子模块, 我们不建议使用从 `GitHub` tag 或 release 页面下载的源代码来进行编译.**

### Maven代理
如果需要在代理之后执行构建，请编辑 **.mvn/jvm.config** 并放置以下属性
```properties
-Dhttp.proxyHost=proxy_ip
-Dhttp.proxyPort=proxy_port
-Dhttps.proxyHost=proxy_ip
-Dhttps.proxyPort=proxy_port 
-Dhttp.proxyUser=username
-Dhttp.proxyPassword=password
```

### 从 GitHub 构建

1. 准备 Git, JDK8+ 和 Maven3.6+
2. 克隆项目

    如果从源代码构建一个版本, 克隆时 `git clone -b [tag_name] ...` 必须指定 `tag name`.
    ```bash
        git clone --recurse-submodules https://github.com/apache/skywalking.git
        cd skywalking/
        
        或者
        
        git clone https://github.com/apache/skywalking.git
        cd skywalking/
        git submodule init
        git submodule update
    ```
    
1. 运行 `./mvnw clean package -DskipTests`
1. 所有编译出来的包都在目录 `/dist` 下 (Linux 下为 .tar.gz, Windows 下为 .zip).

### 从 Apache 发行版本源代码构建

- 什么是 **Apache 发行版本源代码**?

对于每个正式的 Apache 发行版本, 都会有一个完整且独立的源代码压缩包, 其中包含了所有的源代码,
你可以从 [SkyWalking Apache 下载页面](http://skywalking.apache.org/downloads/)下载得到. 此时没有任何与 git 相关的东西.
跟着以下步骤操作即可.

1. 准备 JDK8+ 和 Maven3.6+
2. 运行 `./mvnw clean package -DskipTests`
3. 所有编译出来的包都在目录 `/dist` 下 (Linux 下为 .tar.gz, Windows 下为 .zip).

### 高级编译

SkyWalking 是一个复杂的 Maven 项目, 包括许多模块, 其中可能包含一些编译耗时非常长的模块.
如果你只想重新编译项目的某个部分, 有以下选项可以支持:

- 编译打包 agent

>  ./mvnw package -Pagent,dist

或

> make build.agent

- 编译打包 backend

>  ./mvnw package -Pbackend,dist

或

> make build.backend

- 编译打包 UI

>  ./mvnw package -Pui,dist

或

> make build.ui


### 构建 docker 镜像

我们可以使用根目录下的 `Makefile` 文件来构建 `backend` 和 `ui` 的 docker 镜像.

- 构建所有 docker 镜像

> make docker.all

- 构建 oap 服务的 docker 镜像

> make docker.oap

- 构建 UI 的 docker 镜像

> make docker.ui

`HUB` 和 `TAG` 变量用于设置一个 docker 镜像的 `REPOSITORY` 和 `TAG`.
要得到一个名为 `bar/oap:foo` 的 OAP 镜像, 运行以下命令:

> HUB=bar TAG=foo make docker.oap

## 设置 IntelliJ IDEA
**注意**: 如果你从Github克隆代码, 请确保已完成 **[从 GitHub 构建](#)** 中的步骤1至3, 如果你从SkyWalking的官方网站下载源代码, 请确保已按照 **[从 Apache 发行版本源代码构建](#)** 中的步骤操作.
1. 将项目导入为 maven 项目
2. 运行 `./mvnw compile -Dmaven.test.skip=true` 编译项目, 生成必要的源代码(由于使用了 gRPC 和 protobuf)
3. 设置 **生成的源代码(Generated Source Code)** 目录.
    * **apm-protocol/apm-network/target/generated-sources/protobuf** 目录下的 `grpc-java` 和 `java` 目录
    * **oap-server/server-core/target/generated-sources/protobuf** 目录下的 `grpc-java` 和 `java` 目录
    * **oap-server/server-receiver-plugin/receiver-proto/target/generated-sources/protobuf** 目录下的 `grpc-java` 和 `java`
    * **oap-server/exporter/target/generated-sources/protobuf** 目录下的 `grpc-java` 和 `java`
    * **oap-server/server-configuration/grpc-configuration-sync/target/generated-sources/protobuf** 目录下的 `grpc-java` 和 `java` 
    * **oap-server/generate-tool-grammar/target/generated-sources** 目录下的 `antlr4` 
    * **oap-server/generated-analysis/target/generated-sources** 目录下的 `oal`
    
## 设置 Eclipse IDE

1. 将项目导入为 maven 项目
2. 为了支持多个源代码目录, 你需要在文件 `skywalking/pom.xml` 中添加以下配置:

```xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>build-helper-maven-plugin</artifactId>
    <version>1.8</version>
    <executions>
        <execution>
            <id>add-source</id>
            <phase>generate-sources</phase>
            <goals>
                <goal>add-source</goal>
            </goals>
            <configuration>
                <sources>
                    <source>src/java/main</source>
                    <source>apm-protocol/apm-network/target/generated-sources/protobuf</source>
                    <source>apm-collector/apm-collector-remote/collector-remote-grpc-provider/target/generated-sources/protobuf</source>
               </sources>
            </configuration>
        </execution>
    </executions>
</plugin>
```

3. 添加以下配置, 使得 Eclipse 的 M2E 插件可以支持运行方案配置

```xml
<pluginManagement>
    <plugins>
    <!--此插件的配置仅仅存放 Eclipse m2e 设置, 不会影响任何 Maven 本身的构建. -->
        <plugin>
            <groupId>org.eclipse.m2e</groupId>
            <artifactId>lifecycle-mapping</artifactId>
            <version>1.0.0</version>
            <configuration>
                <lifecycleMappingMetadata>
                    <pluginExecutions>
                        <pluginExecution>
                            <pluginExecutionFilter>
                                <groupId>org.codehaus.mojo</groupId>
                                <artifactId>build-helper-maven-plugin</artifactId>
                                <versionRange>[1.8,)</versionRange>
                                <goals>
                                    <goal>add-source</goal>
                                </goals>
                            </pluginExecutionFilter>
                        </pluginExecution>
                    </pluginExecutions>
                </lifecycleMappingMetadata>
            </configuration>
        </plugin>
    </plugins>
</pluginManagement>
```

4. 添加 Google guava 依赖到 `apm-collector-remote/collector-remote-grpc-provider/pom.xml` 文件中

```xml
<dependency>
   <groupId>com.google.guava</groupId>
   <artifactId>guava</artifactId>
   <version>24.0-jre</version>
</dependency>
```

5. 运行 `./mvnw compile -Dmaven.test.skip=true`
6. 运行 `maven update`. 执行更新前必须删除并清理项目(将会清理自动生成的 proto 类代码以及编译时生成的 Java 代码)
7. 运行 `./mvnw compile` 编译 `collector-remote-grpc-provider` 和 `apm-protocol`
8. 刷新项目
