# 初始化


`RestClient` 实例可以通过相应的 `RestClientBuilder` 类来构建,通过静态方法 `RestClient#builder(HttpHost...)` 创建。唯一必需的参数是服务的host和端口（默认9200，切记不要使用9300），以 [`HttpHost`](https://hc.apache.org/httpcomponents-core-ga/httpcore/apidocs/org/apache/http/HttpHost.html) 实例的方式提供给建造者，如下所示：

```
RestClient restClient = RestClient.builder(
        new HttpHost("localhost", 9200, "http"),
        new HttpHost("localhost", 9201, "http")).build();
```

`RestClient` 类是线程安全的，理想状态下它与使用它的应用程序具有相同的生命周期。当不再使用时强烈建议关闭它：

```
restClient.close();
```

`RestClientBuilder` 在构建 `RestClient` 实例时可以设置以下的可选配置参数：

```
RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "http"));
Header[] defaultHeaders = new Header[]{
        new BasicHeader("header", "value")
    };
builder.setDefaultHeaders(defaultHeaders); // 设置默认头文件，避免每个请求都必须指定。
```

```
RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "http"));

builder.setMaxRetryTimeoutMillis(10000);//设置在同一请求进行多次尝试时应该遵守的超时时间。默认值为30秒，与默认`socket`超时相同。 如果自定义设置了`socket`超时，则应该相应地调整最大重试超时。

```


```
RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "http"));
builder.setFailureListener(new RestClient.FailureListener() {
    @Override
    public void onFailure(HttpHost host) {
        //设置每次节点发生故障时收到通知的侦听器。内部嗅探到故障时被启用。
    }
});
```

```
RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "http"));
builder.setRequestConfigCallback(new RestClientBuilder.RequestConfigCallback() {
        @Override
        public RequestConfig.Builder customizeRequestConfig(RequestConfig.Builder requestConfigBuilder) {
            return requestConfigBuilder.setSocketTimeout(10000); // 设置修改默认请求配置的回调（例如：请求超时，认证，或者其他 [org.apache.http.client.config.RequestConfig.Builder](https://hc.apache.org/httpcomponents-client-ga/httpclient/apidocs/org/apache/http/client/config/RequestConfig.Builder.html) 设置）。
        }
    });

```



```
RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "http"));
builder.setHttpClientConfigCallback(new RestClientBuilder.HttpClientConfigCallback() {
    @Override
    public HttpAsyncClientBuilder customizeHttpClient(HttpAsyncClientBuilder httpClientBuilder) {
        return httpClientBuilder.setProxy(new HttpHost("proxy", 9000, "http")); //设置修改 http 客户端配置的回调（例如：ssl 加密通讯，或其他任何 [org.apache.http.impl.nio.client.HttpAsyncClientBuilder](http://hc.apache.org/httpcomponents-asyncclient-dev/httpasyncclient/apidocs/org/apache/http/impl/nio/client/HttpAsyncClientBuilder.html) 设置）
    }
});

```
