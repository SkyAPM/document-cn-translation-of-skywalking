### 现象
- maven构建时，protoc-plugins插件出现如下错误：
```
[ERROR] Failed to execute goal org.xolstice.maven.plugins:protobuf-maven-plugin:0.5.0:compile-custom (default) on project apm-network: Unable to copy the file to \skywalking\apm-network\target\protoc-plugins: \skywalking\apm-network\target\protoc-plugins\protoc-3.3.0-linux-x86_64.exe (The process cannot access the file because it is being used by another process) -> [Help 1]```
```

### 原因
- Protobuf编译器依赖于glibc环境，而系统上没有安装它或安装的旧版本。

### 解决方法
- 检查并升级最新版本glibc库，若使用容器镜像环境推荐含有最新版本glibc的alpine系统。请参考官方手册：http://www.gnu.org/software/libc/documentation.html
