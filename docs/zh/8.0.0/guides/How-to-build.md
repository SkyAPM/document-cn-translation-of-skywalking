# How to build project
This document helps people to compile and build the project in your maven and set your IDE.

## Build Project
**Because we are using Git submodule, we recommend don't use `GitHub` tag or release page to download source codes for compiling.**

### Maven behind Proxy
If you need to execute build behind the proxy, edit the *.mvn/jvm.config* and put the follow properties:
```properties
-Dhttp.proxyHost=proxy_ip
-Dhttp.proxyPort=proxy_port
-Dhttps.proxyHost=proxy_ip
-Dhttps.proxyPort=proxy_port 
-Dhttp.proxyUser=username
-Dhttp.proxyPassword=password
```

### Build from GitHub
1. Prepare git, JDK8 and maven3
1. Clone project

    If you want to build a release from source codes, provide a `tag name` by using `git clone -b [tag_name] ...` while cloning.
    
    ```bash
    git clone --recurse-submodules https://github.com/apache/skywalking.git
    cd skywalking/
    
    OR
    
    git clone https://github.com/apache/skywalking.git
    cd skywalking/
    git submodule init
    git submodule update
    ```
   
1. Run `./mvnw clean package -DskipTests`
1. All packages are in `/dist` (.tar.gz for Linux and .zip for Windows).

### Build from Apache source code release
- What is `Apache source code release`?

For each official Apache release, there is a complete and independent source code tar, which is including all source codes. You could download it from [SkyWalking Apache download page](http://skywalking.apache.org/downloads/). No git related stuff required when compiling this. Just follow these steps.

1. Prepare JDK8 and maven3
1. Run `./mvnw clean package -DskipTests`
1. All packages are in `/dist`.(.tar.gz for Linux and .zip for Windows).

### Advanced compile
SkyWalking is a complex maven project, including many modules, which could cause long compiling time. 
If you just want to recompile part of the project, you have following options
- Compile agent and package
>  ./mvnw package -Pagent,dist

or

> make build.agent

- Compile backend and package
>  ./mvnw package -Pbackend,dist

or

> make build.backend

- Compile UI and package
>  ./mvnw package -Pui,dist

or

> make build.ui


### Build docker images
We can build docker images of `backend` and `ui` with `Makefile` located in root folder.

Refer to [Build docker image](../../../docker) for more details.

## Setup your IntelliJ IDEA
**NOTICE**: If you clone the codes from GitHub, please make sure that you had finished step 1 to 3 in section **[Build from GitHub](#build-from-github)**, if you download the source codes from the official website of SkyWalking, please make sure that you had followed the steps in section **[Build from Apache source code release](#build-from-apache-source-code-release)**.

1. Import the project as a maven project
1. Run `./mvnw compile -Dmaven.test.skip=true` to compile project and generate source codes. Because we use gRPC and protobuf.
1. Set **Generated Source Codes** folders.
    * `grpc-java` and `java` folders in **apm-protocol/apm-network/target/generated-sources/protobuf**
    * `grpc-java` and `java` folders in **oap-server/server-core/target/generated-sources/protobuf**
    * `grpc-java` and `java` folders in **oap-server/server-receiver-plugin/receiver-proto/target/generated-sources/protobuf**
    * `grpc-java` and `java` folders in **oap-server/exporter/target/generated-sources/protobuf**
    * `grpc-java` and `java` folders in **oap-server/server-configuration/grpc-configuration-sync/target/generated-sources/protobuf**
    * `antlr4` folder in **oap-server/oal-grammar/target/generated-sources**
    
## Setup your Eclipse IDE
**NOTICE**: If you clone the codes from GitHub, please make sure that you had finished step 1 to 3 in section **[Build from GitHub](#build-from-github)**, if you download the source codes from the official website of SkyWalking, please make sure that you had followed the steps in section **[Build from Apache source code release](#build-from-apache-source-code-release)**.

1. Import the project as a maven project
1. For supporting multiple source directories, you need to add the following configuration in `skywalking/pom.xml` file:
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
1. Add the following configuration under to let eclipse's M2e plug-in supports execution's solution configuration
    ```xml
    <pluginManagement>
        <plugins>
        <!--This plugin's configuration is used to store Eclipse m2e settings 
        only. It has no influence on the Maven build itself. -->
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
1. Adding Google guava dependency to apm-collector-remote/collector-remote-grpc-provider/pom.xml files
    ```xml
    <dependency>
       <groupId>com.google.guava</groupId>
       <artifactId>guava</artifactId>
       <version>24.0-jre</version>
    </dependency>
    ```
1. Run `./mvnw compile -Dmaven.test.skip=true`
1. Run `maven update`. Must remove the clean projects item before maven update(This will be clear the proto conversion Java file generated by the compiling)
1. Run `./mvnw compile` compile collector-remote-grpc-provider and apm-protocol
1. Refresh project
