# 用法

一旦创建了`RestClient`实例，如 [`初始化`](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-low-usage-initialization.html)中所示，可以将嗅探器与之相关联。 `Sniffer` 将使用关联的 `RestClient` 定期（默认为每5分钟）从集群中获取当前可用的节点列表，并通过调用 `RestClient＃setHosts` 来更新它们。

```
RestClient restClient = RestClient.builder(
        new HttpHost("localhost", 9200, "http"))
        .build();
Sniffer sniffer = Sniffer.builder(restClient).build();
```

关闭 `Sniffer` 非常重要，如此嗅探器后台线程才能正取关闭并释放他持有的资源。 `Sniffer` 对象应该与 `RestClient` 具有相同的生命周期，并在客户端之前关闭：

```
sniffer.close();
restClient.close();
```

`Sniffer` 默认每5分钟更新一次节点列表。这个周期可以如下方式通过提供一个参数（毫秒数）自定义设置：

```
RestClient restClient = RestClient.builder(
        new HttpHost("localhost", 9200, "http"))
        .build();
Sniffer sniffer = Sniffer.builder(restClient)
        .setSniffIntervalMillis(60000).build();
```

也可以在发生故障是启用嗅探，这意味着每次故障后将直接获取并更新节点列表，而不是等到下一次正常的更新周期。此种情况时， SniffOnFailureListener 需要首先被创建，并将实例在 RestClient 创建时提供给它。 同样的，在之后创建 Sniffer 时，他需要被关联到同一个 SniffOnFailureListener 实例上，这个实例将在每个故障发生后被通知到，然后调用 Sniffer 去执行额外的嗅探行为。

```
SniffOnFailureListener sniffOnFailureListener = new SniffOnFailureListener();
RestClient restClient = RestClient.builder(new HttpHost("localhost", 9200))
        .setFailureListener(sniffOnFailureListener) //为 RestClient 实例设置故障监听器
        .build();
Sniffer sniffer = Sniffer.builder(restClient
        .setSniffAfterFailureDelayMillis(30000) /* 故障后嗅探，不仅意味着每次故障后会更新节点，也会添加普通计划外的嗅探行为，默认情况是故障之后1分钟后，假设节点将恢复正常，那么我们希望尽可能快的获知。如上所述，周期可以通过 `setSniffAfterFailureDelayMillis` 方法在创建 Sniffer 实例时进行自定义设置。 需要注意的是，当没有启用故障监听时，这最后一个配置参数不会生效 */
        .build();
sniffOnFailureListener.setSniffer(sniffer); // 将 嗅探器关联到嗅探故障监听器上
```

`Elasticsearch Nodes Info api` 不会返回连接节点使用的协议，而只有他们的 `host:port `键值对，因此默认使用 `http`。如果需要使用 `https` ，必须手动创建和提供 `ElasticsearchHostsSniffer` 实例，如下所示：

```
RestClient restClient = RestClient.builder(
        new HttpHost("localhost", 9200, "http"))
        .build();
HostsSniffer hostsSniffer = new ElasticsearchHostsSniffer(
        restClient,
        ElasticsearchHostsSniffer.DEFAULT_SNIFF_REQUEST_TIMEOUT,
        ElasticsearchHostsSniffer.Scheme.HTTPS);
Sniffer sniffer = Sniffer.builder(restClient)
        .setHostsSniffer(hostsSniffer).build();

```

使用同样的方式，可以自定义设置 sniffRequestTimeout参数，该参数默认值为 1 秒。这是一个调用 Nodes Info api 时作为 querystring 参数的超时参数，这样当服务端超市时，仍然会返回一个有效响应，虽然它可能仅包含属于集群的一部分节点，其他节点会在随后响应。

```
RestClient restClient = RestClient.builder(
        new HttpHost("localhost", 9200, "http"))
        .build();
HostsSniffer hostsSniffer = new ElasticsearchHostsSniffer(
        restClient,
        TimeUnit.SECONDS.toMillis(5),
        ElasticsearchHostsSniffer.Scheme.HTTP);
Sniffer sniffer = Sniffer.builder(restClient)
        .setHostsSniffer(hostsSniffer).build();
```

同样的，一个自定义的 HostsSniffer 实现可以提供一个高级用法功能，比如可以从 Elasticsearch 之外的来源获取主机：

```
RestClient restClient = RestClient.builder(
        new HttpHost("localhost", 9200, "http"))
        .build();
HostsSniffer hostsSniffer = new HostsSniffer() {
    @Override
    public List<HttpHost> sniffHosts() throws IOException {
        return null; // 从外部源获取主机
    }
};
Sniffer sniffer = Sniffer.builder(restClient)
        .setHostsSniffer(hostsSniffer).build();
```