### 问题描述
- 导入skywalking工程到eclipse,遇到如下异常
> Software being installed: Checkstyle configuration plugin for
> M2Eclipse 1.0.0.201705301746
> (com.basistech.m2e.code.quality.checkstyle.feature.feature.group 
> 1.0.0.201705301746) Missing requirement: Checkstyle configuration plugin for M2Eclipse 1.0.0.201705301746
> (com.basistech.m2e.code.quality.checkstyle.feature.feature.group 
> 1.0.0.201705301746) requires 'net.sf.eclipsecs.core 5.2.0' but it could not be found

### 原因
没有安装Eclipse Checkstyle Plug-in插件

### 解决方法
下载Eclipse Checkstyle Plug-in插件（版本：8.7.0.201801131309），地址：[https://sourceforge.net/projects/eclipse-cs/?source=typ_redirect](https://sourceforge.net/projects/eclipse-cs/?source=typ_redirect)。
插件说明:
此插件将JAVA的代码审计功能集成到了Eclipse中。该插件向用户实时反馈违反编码风格和可能出错的代码构造规则。