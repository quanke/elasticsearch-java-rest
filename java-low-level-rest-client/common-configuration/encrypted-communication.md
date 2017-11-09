# 加密传输

可以通过`HttpClientConfigCallback`配置加密传输。 参数`org.apache.http.impl.nio.client.HttpAsyncClientBuilder` 公开了多个方法来配置加密传输：`setSSLContext`，`setSSLSessionStrategy`和`setConnectionManager`，以下是一个例子：


```

KeyStore truststore = KeyStore.getInstance("jks");
try (InputStream is = Files.newInputStream(keyStorePath)) {
    truststore.load(is, keyStorePass.toCharArray());
}
SSLContextBuilder sslBuilder = SSLContexts.custom().loadTrustMaterial(truststore, null);
final SSLContext sslContext = sslBuilder.build();
RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "https"))
        .setHttpClientConfigCallback(new RestClientBuilder.HttpClientConfigCallback() {
            @Override
            public HttpAsyncClientBuilder customizeHttpClient(HttpAsyncClientBuilder httpClientBuilder) {
                return httpClientBuilder.setSSLContext(sslContext);
            }
        });
```

如果没有提供明确的配置，则使用[系统默认配置](http://docs.oracle.com/javase/7/docs/technotes/guides/security/jsse/JSSERefGuide.html#CustomizingStores)。


