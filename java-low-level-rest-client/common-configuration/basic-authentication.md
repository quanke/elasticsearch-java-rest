# 基本认证

构建`RestClient`时配置`HttpClientConfigCallback`来配置基本认证。 该接口有一个方法接收`org.apache.http.impl.nio.client.HttpAsyncClientBuilder`的一个实例作为参数，并具有相同的返回类型。 `httpClientBuilder` 被修改，然后返回。 在以下示例中，设置了基本身份验证。

```
final CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
credentialsProvider.setCredentials(AuthScope.ANY,
        new UsernamePasswordCredentials("user", "password"));

RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200))
        .setHttpClientConfigCallback(new RestClientBuilder.HttpClientConfigCallback() {
            @Override
            public HttpAsyncClientBuilder customizeHttpClient(HttpAsyncClientBuilder httpClientBuilder) {
                return httpClientBuilder.setDefaultCredentialsProvider(credentialsProvider);
            }
        });
```

Preemptive 身份验证可以禁用，这意味着每个发送出去的请求没有授权头，当收到`HTTP 401`响应时，将重新发送与基本身份验证头完全相同的请求。 可以通过`HttpAsyncClientBuilder`来禁用：


```
final CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
credentialsProvider.setCredentials(AuthScope.ANY,
        new UsernamePasswordCredentials("user", "password"));

RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200))
        .setHttpClientConfigCallback(new RestClientBuilder.HttpClientConfigCallback() {
            @Override
            public HttpAsyncClientBuilder customizeHttpClient(HttpAsyncClientBuilder httpClientBuilder) {
                httpClientBuilder.disableAuthCaching(); //禁用 preemptive 身份验证
                return httpClientBuilder.setDefaultCredentialsProvider(credentialsProvider);
            }
        });
```