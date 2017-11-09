# 超时

配置请求超时可以通过构建器构建`RestClient`时提供`RequestConfigCallback`实例来完成。 该接口有一个方法接收`org.apache.http.client.config.RequestConfig.Builder`的一个实例作为参数，并具有相同的返回类型。 请求配置生成器可以修改，然后返回。 在下面的例子中，我们增加了连接超时（默认为1秒）和socket超时（默认为30秒）。 也调整最大重试超时时间（默认为30秒）。

```
RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200))
        .setRequestConfigCallback(new RestClientBuilder.RequestConfigCallback() {
            @Override
            public RequestConfig.Builder customizeRequestConfig(RequestConfig.Builder requestConfigBuilder) {
                return requestConfigBuilder.setConnectTimeout(5000)
                        .setSocketTimeout(60000);
            }
        })
        .setMaxRetryTimeoutMillis(60000);
```