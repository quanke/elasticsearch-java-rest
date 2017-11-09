# 线程数

`Apache Http Async Client` 默认启动一个调度线程，连接管理器使用多个`worker`线程,线程的数量和CPU核数量相同（等于 `Runtime.getRuntime().availableProcessors() Runtime.getRuntime().availableProcessors()`返回的数量），线程数可以修改如下：

```
RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200))
        .setHttpClientConfigCallback(new RestClientBuilder.HttpClientConfigCallback() {
            @Override
            public HttpAsyncClientBuilder customizeHttpClient(HttpAsyncClientBuilder httpClientBuilder) {
                return httpClientBuilder.setDefaultIOReactorConfig(
                        IOReactorConfig.custom().setIoThreadCount(1).build());
            }
        });
```