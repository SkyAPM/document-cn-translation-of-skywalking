# 端点分组参数化
在大多数情况下，端点应该通过语言代理、服务网格可观察性解决方案或仪表系统的配置自动检测。
有一些特殊的情况，特别是当人们使用REST样式的URI时，应用程序代码将参数放在端点名称中，例如将订单id放在URI中，如/prod/ORDER123和/prod/ORDER123。
但从逻辑上讲，人们希望他们可以有一个端点名，如prod/{order-id}。这是为端点分组参数化而设计的特性。
目前，用户可以通过静态YAML文件中的 endpoint_name_grouping 来设置分组规则。或使用[动态配置](dynamic-config.md)初始化和更新端点分组规则。
# 配置格式
无论是静态本地文件还是动态配置值，它们都共享相同的YAML格式。
```yaml
grouping:
  # Endpoint of the service would follow the following rules
  - service-name: serviceA
    rules:
      # Logic name when the regex expression matched.
      - endpoint-name: /prod/{id}
        regex: \/prod\/.+
```