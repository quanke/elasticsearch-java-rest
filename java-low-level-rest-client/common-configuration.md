# 通用配置

正如[初始化](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-low-usage-initialization.html)中所解释的那样，`RestClientBuilder`支持同时提供`RequestConfigCallback`和`HttpClientConfigCallback`，它们允许任何定制的`Apache Async Http Client `。 这些回调可以修改客户端的某些特定行为，而不会覆盖`RestClient`初始化的其他任何默认配置。 本节介绍了一些需要对低级Java REST客户端进行额外配置的常见方案。