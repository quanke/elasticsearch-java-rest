# 兼容性

Java高级REST客户端需要Java 1.8，并依赖于Elasticsearch核心项目。 客户端版本要与客户端开发的Elasticsearch版本相同。 它接受与 `TransportClient` 相同的请求参数，并返回相同的响应对象。 如果需要将应用程序从TransportClient迁移到新的REST客户端，请参阅 [`迁移指南`](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-level-migration.html)。

高级客户端保证能够与运行在相同主版本和大于或等于次要版本的任何Elasticsearch节点进行通信。它不需要与它进行通信的弹性搜索节点相同的次要版本，因为它是向前兼容的，意味着它支持与之前开发的弹性搜索的更新版本进行通信。

5.6 客户端可以与任何 5.6.x Elasticsearch 节点进行通信。以前的 5.x 小版本，如 5.5.x，5.4.x 等不（完全）支持。

6.0 客户端能够与任何 6.x Elasticsearch 节点进行通信，而 6.1 客户端确实能够与 6.1,6.2 和以后的 6.x 版本进行通信，但与以前的 Elasticsearch 节点版本通信时可能会出现不兼容问题例如 6.1 到 6.0 之间，例如 6.1 客户端支持而 6.0 节点不知道的某些API的新请求主体字段。

建议在将Elasticsearch集群升级到新的主要版本时升级高级客户端，因为REST API突破性更改可能会导致意外的结果，具体取决于请求所击中的节点，新添加的API只能由较新版本的客户端。一旦群集中的所有节点都升级到新的主版本，则客户端应当更新。