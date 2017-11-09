# 读取响应

返回`Response`对象（`performRequest`,方法返回，`ResponseListener#onSuccess(Response)`接收）

```
Response response = restClient.performRequest("GET", "/");
RequestLine requestLine = response.getRequestLine(); //请求信息
HttpHost host = response.getHost(); //返回response host信息
int statusCode = response.getStatusLine().getStatusCode(); //返回状态行，获取状态码
Header[] headers = response.getHeaders(); //response headers ，也可以通过名字获取 `getHeader(String)`
String responseBody = EntityUtils.toString(response.getEntity()); //response  org.apache.http.HttpEntity 对象

```

请求时可能抛出一下异常（或者 `ResponseListener#onFailure(Exception)` 参数接收错误信息）

- `IOException`

通信问题（例如:`SocketTimeoutException`）

- `ResponseException`

返回了一个 response，但是它的状态码显示了一个错误（不是2xx）。`ResponseException` 说明连接是通的。

> 对于返回404状态码的请求，不会抛出 `ResponseException`，因为这是一个预期的响应，只是表示找不到该资源。 除非`ignore`参数包含404，否则所有其他HTTP方法（例如GET）都会为404响应抛出 `ResponseException`。`ignore`是一个特殊的客户端参数，不会发送到`Elasticsearch`，并且包含以逗号分隔的错误状态码列表。 它允许控制某些错误状态代码是否应该被视为预期的响应，而不是一个例外。 这对`get api`来说很有用，因为它可以在缺少文档时返回404，在这种情况下，响应主体不会包含错误，而是通常的`get api`响应，而不是文档，因为它没有找到。


注意: 低级别的客户端不公开任何 helper jsonmarshalling和un-marshalling。 用户可以自由使用他们喜欢的库。

底层的Apache异步Http客户端附带不同的[`org.apache.http.HttpEntity`](https://hc.apache.org/httpcomponents-core-ga/httpcore/apidocs/org/apache/http/HttpEntity.html)实现，可以使用不同的格式（流，字节数组，字符串等）提供请求主体。 至于读取响应主体，`HttpEntity＃getContent`方法很方便，它返回来自先前缓冲的响应主体`InputStream`。 可以提供一个自定义的`org.apache.http.nio.protocol.HttpAsyncResponseConsumer`来控制如何读取和缓冲字节，作为替代。

