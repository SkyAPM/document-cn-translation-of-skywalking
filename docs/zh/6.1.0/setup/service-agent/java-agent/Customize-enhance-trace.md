## 支持自定义增强
`apm-customize-enhance-plugin`为可选插件

## 介绍
SkyWalking提供了[Java agent插件开发指南](https://github.com/apache/incubator-skywalking/blob/master/docs/en/guides/Java-Plugin-Development-Guide.md)帮助开发者们构建新的插件。

这个插件不是为替代某个插件而设计，而是为了用户使用方便。这个插件的行为跟[@Trace toolkit](Application-toolkit-trace.md)很相似，但是不需要对代码进行修改，而且功能更强大，比如提供了tag和log。

## 如何配置
实现对类的自定义增强需要2步。

1. 激活插件，将插件从`optional-plugins/apm-customize-enhance-plugin.jar`移动到`plugin/apm-customize-enhance-plugin.jar`。
2. 在agent.config中配置`plugin.customize.enhance_file`，指明增强规则文件，比如`/absolute/path/to/customize_enhance.xml`。
3. 在`customize_enhance.xml`中配置增强规则。
	```xml
	<?xml version="1.0" encoding="UTF-8"?>
	<enhanced>
	    <class class_name="test.apache.skywalking.testcase.customize.service.TestService1">
	        <method method="staticMethod()" operation_name="/is_static_method" static="true"/>
	        <method method="staticMethod(java.lang.String,int.class,java.util.Map,java.util.List,[Ljava.lang.Object;)" operation_name="/is_static_method_args" static="true">
	            <operation_name_suffix>arg[0]</operation_name_suffix>
	            <operation_name_suffix>arg[1]</operation_name_suffix>
	            <operation_name_suffix>arg[3].[0]</operation_name_suffix>
	            <tag key="tag_1">arg[2].['k1']</tag>
	            <tag key="tag_2">arg[4].[1]</tag>
	            <log key="log_1">arg[4].[2]</log>
	        </method>
	        <method method="method()" static="false"/>
	        <method method="method(java.lang.String,int.class)" operation_name="/method_2" static="false">
	            <operation_name_suffix>arg[0]</operation_name_suffix>
	            <tag key="tag_1">arg[0]</tag>
	            <log key="log_1">arg[1]</log>
	        </method>
	        <method method="method(test.apache.skywalking.testcase.customize.model.Model0,java.lang.String,int.class)" operation_name="/method_3" static="false">
	            <operation_name_suffix>arg[0].id</operation_name_suffix>
	            <operation_name_suffix>arg[0].model1.name</operation_name_suffix>
	            <operation_name_suffix>arg[0].model1.getId()</operation_name_suffix>
	            <tag key="tag_os">arg[0].os.[1]</tag>
	            <log key="log_map">arg[0].getM().['k1']</log>
	        </method>
	    </class>
	    <class class_name="test.apache.skywalking.testcase.customize.service.TestService2">
	        <method method="staticMethod(java.lang.String,int.class)" operation_name="/is_2_static_method" static="true">
	            <tag key="tag_2_1">arg[0]</tag>
	            <log key="log_1_1">arg[1]</log>
	        </method>
	        <method method="method([Ljava.lang.Object;)" operation_name="/method_4" static="false">
	            <tag key="tag_4_1">arg[0].[0]</tag>
	        </method>
	        <method method="method(java.util.List,int.class)" operation_name="/method_5" static="false">
	            <tag key="tag_5_1">arg[0].[0]</tag>
	            <log key="log_5_1">arg[1]</log>
	        </method>
	    </class>
	</enhanced>
	```

- 文件中的配置说明。

	| 配置  | 说明 |
	|:----------------- |:---------------|
	| class_name | 要被增强的类 |
	| method | 类的拦截器方法 |
	| operation_name | 如果进行了配置，将用它替代默认的operation_name |
	| operation_name\_suffix | 表示在operation_name后添加动态数据 |
	| static | 方法是否为静态方法 |
	| tag | 将在local span中添加一个tag。key的值需要在XML节点上表示。|
	| log | 将在local span中添加一个log。key的值需要在XML节点上表示。|
	| arg[x]   | 表示输入的参数值。比如args[0]表示第一个参数。 |
	| .[x]     | 当正在被解析的对象是Array或List，你可以用这个表达式得到对应index上的对象。|
	| .['key'] | 当正在被解析的对象是Map, 你可以用这个表达式得到map的key。|

