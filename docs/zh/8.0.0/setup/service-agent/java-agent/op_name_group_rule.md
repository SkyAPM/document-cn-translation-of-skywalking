# 操作名称分组规则
agent 自动检测出的操作名称是变化莫测的, 有时，目标应用程序在其中携带参数，这主要是由于URI中包含的参数.
大多数情况下这些操作名也称为端点名.
例如 /api/checkTicket/tk/{userToken}.

我们利用框架中的参数形成路径解决了大多数情况，比如SpringMVC、Webflux等. 
在这种情况下，通过这种方式是无法检测到的，因此我们必须要求用户手动设置组规则.

与其他 agent 配置一样，支持通过 agent.config 设置所有规则，系统属性和系统环境.
- 配置格式, `plugin.opgroup.`插件名称`.rule[`规则名称`]`=正则表达式

支持具有不同键值，即单个插件有多个配置项，例如
1. plugin.opgroup.resttemplate.rule[/rule1]=/path1
2. plugin.opgroup.resttemplate.rule[/rule2]=/path2
3. plugin.opgroup.resttemplate.rule[/rule2]=/path3

我们有以下插件支持操作名称分组.

| Plugin | Config Key | Example |
|:----:|:-----:|:----|
|RestTemplate| plugin.opgroup.resttemplate.rule | plugin.opgroup.resttemplate.rule[/user/auth/{token}]=`\/user\/auth\/.*` |