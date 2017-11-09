# 发送请求


一旦创建了 `RestClient`，就可以通过调用其中一个`performRequest`或`performRequestAsync`方法来发送请求。 `performRequest` 方法是同步的，并直接返回`Response`，这意味着客户端将阻塞并等待返回的响应。 `performRequestAsync` 返回  `void`，并接受一个额外的`ResponseListener`作为参数，这意味着它们是异步执行的。 提供的监听器将在请求完成或失败时通知。

```
Response response = restClient.performRequest("GET", "/"); //最简单的发送一个请求
```


```
Map<String, String> params = Collections.singletonMap("pretty", "true");
Response response = restClient.performRequest("GET", "/", params);  //发送一个带参数的请求
```

```
Map<String, String> params = Collections.emptyMap();
String jsonString = "{" +
            "\"user\":\"kimchy\"," +
            "\"postDate\":\"2013-01-30\"," +
            "\"message\":\"trying out Elasticsearch\"" +
        "}";
HttpEntity entity = new NStringEntity(jsonString, ContentType.APPLICATION_JSON);// org.apache.http.HttpEntity  为了让Elasticsearch 能够解析,需要设置ContentType。
Response response = restClient.performRequest("PUT", "/posts/doc/1", params, entity); 

```



```
Map<String, String> params = Collections.emptyMap();
HttpAsyncResponseConsumerFactory.HeapBufferedResponseConsumerFactory consumerFactory =
        new HttpAsyncResponseConsumerFactory.HeapBufferedResponseConsumerFactory(30 * 1024 * 1024); //[ org.apache.http.nio.protocol.HttpAsyncResponseConsumer](http://hc.apache.org/httpcomponents-core-ga/httpcore-nio/apidocs/org/apache/http/nio/protocol/HttpAsyncResponseConsumer.html) ,
Response response = restClient.performRequest("GET", "/posts/_search", params, null, consumerFactory); 

```


```
ResponseListener responseListener = new ResponseListener() {
    @Override
    public void onSuccess(Response response) {
        // 请求成功回调
    }

    @Override
    public void onFailure(Exception exception) {
        //请求失败时回调
    }
};
restClient.performRequestAsync("GET", "/", responseListener); //发送异步请求

```


```
Map<String, String> params = Collections.singletonMap("pretty", "true");
restClient.performRequestAsync("GET", "/", params, responseListener); // 发送带参数的异步请求

```

```
String jsonString = "{" +
        "\"user\":\"kimchy\"," +
        "\"postDate\":\"2013-01-30\"," +
        "\"message\":\"trying out Elasticsearch\"" +
        "}";
HttpEntity entity = new NStringEntity(jsonString, ContentType.APPLICATION_JSON);
restClient.performRequestAsync("PUT", "/posts/doc/1", params, entity, responseListener); 

```

```
HttpAsyncResponseConsumerFactory.HeapBufferedResponseConsumerFactory consumerFactory =
        new HttpAsyncResponseConsumerFactory.HeapBufferedResponseConsumerFactory(30 * 1024 * 1024);
restClient.performRequestAsync("GET", "/posts/_search", params, null, consumerFactory, responseListener); 

```

以下是如何发送异步请求的基本示例：

```
final CountDownLatch latch = new CountDownLatch(documents.length);
for (int i = 0; i < documents.length; i++) {
    restClient.performRequestAsync(
            "PUT",
            "/posts/doc/" + i,
            Collections.<String, String>emptyMap(),
            //let's assume that the documents are stored in an HttpEntity array
            documents[i],
            new ResponseListener() {
                @Override
                public void onSuccess(Response response) {
                    
                    latch.countDown();//处理返回响应
                }

                @Override
                public void onFailure(Exception exception) {
                    
                    latch.countDown();//处理失败响应，exception 里带错误码
                }
            }
    );
}
latch.await();
```

上面列出的每一种方法都支持通过`Header varargs`参数和请求一起`headers`，如下例所示：

```
Response response = restClient.performRequest("GET", "/", new BasicHeader("header", "value"));

```
```
Header[] headers = {
        new BasicHeader("header1", "value1"),
        new BasicHeader("header2", "value2")
};
restClient.performRequestAsync("GET", "/", responseListener, headers);
```