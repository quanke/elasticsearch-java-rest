
# Get API

## 获取请求对象

GetRequest 需要以下参数：

```
GetRequest getRequest = new GetRequest(
        "posts", // 索引
        "doc",  // 类别
        "1");   // 文档id
```

### 可选参数

提供以下可选参数：

```
request.fetchSourceContext(new FetchSourceContext(false)); //禁用检索源，默认为启用

```
String[] includes = new String[]{"message", "*Date"};
String[] excludes = Strings.EMPTY_ARRAY;
FetchSourceContext fetchSourceContext = new FetchSourceContext(true, includes, excludes);
request.fetchSourceContext(fetchSourceContext); //设置源包含的特定域

```



```
String[] includes = Strings.EMPTY_ARRAY;
String[] excludes = new String[]{"message"};
FetchSourceContext fetchSourceContext = new FetchSourceContext(true, includes, excludes);
request.fetchSourceContext(fetchSourceContext);
Configure source exclusion for specific fields
```

```
request.storedFields("message");  //Configure retrieval for specific stored fields (requires fields to be stored separately in the mappings)
GetResponse getResponse = client.get(request);
String message = (String) getResponse.getField("message").getValue();  //Retrieve the message stored field (requires the field to be stored separately in the mappings)
```
```
request.routing("routing");  //Routing value
request.parent("parent"); //Parent value
request.preference("preference"); //Preference value
request.realtime(false); //Set realtime flag to false (true by default)
request.refresh(true); //Perform a refresh before retrieving the document (false by default)
request.version(2); //Version
request.versionType(VersionType.EXTERNAL);  //Version type
```

### 同步执行

```
GetResponse getResponse = client.get(getRequest);
```

### 异步执行

```
client.getAsync(request, new ActionListener<GetResponse>() {
    @Override
    public void onResponse(GetResponse getResponse) {
//Called when the execution is successfully completed. The response is provided as an argument.
    }
    @Override
    public void onFailure(Exception e) {
//Called in case of failure. The raised exception is provided as an argument.
    }
});
```

## 获取响应

The returned GetResponse allows to retrieve the requested document along with its metadata and eventually stored fields.

```
String index = getResponse.getIndex();
String type = getResponse.getType();
String id = getResponse.getId();
if (getResponse.isExists()) {
    long version = getResponse.getVersion();
    String sourceAsString = getResponse.getSourceAsString(); //Retrieve the document as a String
    Map<String, Object> sourceAsMap = getResponse.getSourceAsMap(); //Retrieve the document as a Map<String, Object>
    byte[] sourceAsBytes = getResponse.getSourceAsBytes();  //Retrieve the document as a byte[]
} else {
//Handle the scenario where the document was not found. Note that although the returned response has 404 status code, a valid GetResponse is returned rather than an exception thrown. Such response does not hold any source document and its isExists method returns false.
}
```

当针对不存在的索引执行获取请求时，响应有404状态码，抛出一个 ElasticsearchException 异常，需要如下处理：

```
GetRequest request = new GetRequest("does_not_exist", "doc", "1");
try {
    GetResponse getResponse = client.get(request);
} catch (ElasticsearchException e) {
    if (e.status() == RestStatus.NOT_FOUND) {
        // 处理因为索引不存在而抛出的异常，
    }
}
```

如果请求了特定文档版本，但现有文档具有不同的版本号，则会引发版本冲突：

```
try {
    GetRequest request = new GetRequest("posts", "doc", "1").version(2);
    GetResponse getResponse = client.get(request);
} catch (ElasticsearchException exception) {
    if (exception.status() == RestStatus.CONFLICT) {
        //表示返回了版本冲突错误引发异常
    }
}
```