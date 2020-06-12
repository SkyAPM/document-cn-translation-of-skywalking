# 在 WebSphere 上安装 Agent 时, 抛出异常 IllegalStateException

此 FAQ 来自 [社区讨论和反馈](https://github.com/apache/skywalking/issues/2652).

该用户在 WebSphere 7.0.0.11, JDK 1.8_20160719 和 JDK 1.7.0_20150407 上安装 SkyWalking Java 探针,然后出现了以下错误日志

```shell
WARN 2019-05-09 17:01:35:905 SkywalkingAgent-1-GRPCChannelManager-0 ProtectiveShieldMatcher : Byte-buddy occurs exception when match type.
java.lang.IllegalStateException: Cannot resolve type description for java.security.PrivilegedAction
at org.apache.skywalking.apm.dependencies.net.bytebuddy.pool.TypePool$Resolution$Illegal.resolve(TypePool.java:144)
at org.apache.skywalking.apm.dependencies.net.bytebuddy.pool.TypePool$Default$WithLazyResolution$LazyTypeDescription.delegate(TypePool.java:1392)
at org.apache.skywalking.apm.dependencies.net.bytebuddy.description.type.TypeDescription$AbstractBase$OfSimpleType$WithDelegation.getInterfaces(TypeDescription.java:8016)
at org.apache.skywalking.apm.dependencies.net.bytebuddy.description.type.TypeDescription$Generic$OfNonGenericType.getInterfaces(TypeDescription.java:3621)
at org.apache.skywalking.apm.dependencies.net.bytebuddy.matcher.HasSuperTypeMatcher.hasInterface(HasSuperTypeMatcher.java:53)
at org.apache.skywalking.apm.dependencies.net.bytebuddy.matcher.HasSuperTypeMatcher.hasInterface(HasSuperTypeMatcher.java:54)
at org.apache.skywalking.apm.dependencies.net.bytebuddy.matcher.HasSuperTypeMatcher.matches(HasSuperTypeMatcher.java:38)
at org.apache.skywalking.apm.dependencies.net.bytebuddy.matcher.HasSuperTypeMatcher.matches(HasSuperTypeMatcher.java:15)
at org.apache.skywalking.apm.dependencies.net.bytebuddy.matcher.ElementMatcher$Junction$Conjunction.matches(ElementMatcher.java:107)
at org.apache.skywalking.apm.dependencies.net.bytebuddy.matcher.ElementMatcher$Junction$Disjunction.matches(ElementMatcher.java:147)
at org.apache.skywalking.apm.dependencies.net.bytebuddy.matcher.ElementMatcher$Junction$Disjunction.matches(ElementMatcher.java:147)
at org.apache.skywalking.apm.dependencies.net.bytebuddy.matcher.ElementMatcher$Junction$Disjunction.matches(ElementMatcher.java:147)
at org.apache.skywalking.apm.dependencies.net.bytebuddy.matcher.ElementMatcher$Junction$Disjunction.matches(ElementMatcher.java:147)
at org.apache.skywalking.apm.dependencies.net.bytebuddy.matcher.ElementMatcher$Junction$Disjunction.matches(ElementMatcher.java:147)
...
```

该问题已经确定是 WebSphere 上的访问权限导致.
可以按如下解决

1. 设置探针的 owner 为 WebSphere 的 owner
2. 在文件 `server.policy` 中添加 `grant codeBase "file:${agent_dir}/-" { permission java.security.AllPermission; };`".
