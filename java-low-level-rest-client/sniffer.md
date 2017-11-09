# 嗅探器

这是个小型库，可以允许从一个正在运行的 `Elasticsearch` 集群上自动发现节点并将节点列表更新到已经存在的 `RestClient` 实例上。
它默认使用 `Nodes Info api ` 检索属于集群的节点，并使用`jackson`解析获取的`json`响应。

与`Elasticsearch 2.x`及以上兼容。