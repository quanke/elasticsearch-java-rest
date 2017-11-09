# 初始化

RestHighLevelClient 实例的构建需要一个 REST 低级客户端就像下面这样：

```
RestHighLevelClient client =
    new RestHighLevelClient(lowLevelRestClient); //lowLevelRestClient： 我们之前创建的 [REST低级客户端](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-low-usage-initialization.html) 实例
```

在本文档中关于Java高级客户端的的其余部分里，`RestHighLevelClient` 实例将以 `client` 被引用。
