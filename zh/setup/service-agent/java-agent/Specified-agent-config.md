# 通过系统属性定位agent配置文件

## 支持的版本 

5.0.0-RC+

## 什么是通过系统属性定位agent配置文件？
默认地，agent会定位到`/config`目录下的`agent.config`。
如果用户通过系统属性设置了指定的agent配置文件，agent将会根据设置去加载文件。
另外，这个功能与[配置覆盖](Setting-override.md)并不冲突。

## 覆盖优先级
指定的agent配置文件 > 默认的agent配置文件
 
## 如何使用 

指定的配置文件的内容格式必须与默认的配置文件相同。 


**使用 `System.Properties(-D)` 设置指定的配置文件路径**
 
 ```
 -Dskywalking_config=/path/to/agent.config
 ```
 `/path/to/agent.config`  是指定的配置文件的绝对路径。
