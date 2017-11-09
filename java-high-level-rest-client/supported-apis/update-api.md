
# Update API

## 更新请求

UpdateRequest 要求下列参数：

```
UpdateRequest request = new UpdateRequest(
        "posts", // Index
        "doc",  // Type
        "1"); // Document id
```

Update API允许通过使用脚本或传递部分文档来更新现有文档。

### 使用脚本更新

该脚本可以是一个内联脚本：

```
Map<String, Object> parameters = singletonMap("count", 4); // 脚本参数以一个 Map 对象提供。
Script inline = new Script(ScriptType.INLINE, "painless", "ctx._source.field += params.count", parameters);  // 使用 painless 语言创建内联脚本，并传入参数值。
request.script(inline);  //  将脚本传递给更新请求对象
```

或者是一个存储脚本：

```
// 引用一个名为 increment-field 的painless 的存储脚本
Script stored =
        new Script(ScriptType.STORED, "painless", "increment-field", parameters);   
request.script(stored); // 给请求设置脚本
```

### 使用局部文档更新

使用局部文档更新时，局部文档将与现有文档合并。

局部文档可以以不同的方式提供：

```
UpdateRequest request = new UpdateRequest("posts", "doc", "1");
String jsonString = "{" +
        "\"updated\":\"2017-01-01\"," +
        "\"reason\":\"daily update\"" +
        "}";
request.doc(jsonString, XContentType.JSON); // 以一个 JSON 格式的字符串提供的局部文档源
```

```
Map<String, Object> jsonMap = new HashMap<>();
jsonMap.put("updated", new Date());
jsonMap.put("reason", "daily update");
UpdateRequest request = new UpdateRequest("posts", "doc", "1")
        .doc(jsonMap); // 以一个可自动转换为 JSON 格式的 Map 提供局部文档源
```

```
XContentBuilder builder = XContentFactory.jsonBuilder();
builder.startObject();
{
    builder.field("updated", new Date());
    builder.field("reason", "daily update");
}
builder.endObject();
UpdateRequest request = new UpdateRequest("posts", "doc", "1")
        .doc(builder); // 以 XContentBuilder 对象提供局部文档源， Elasticsearch 构建辅助器将生成 JSON 格式。
```
```
UpdateRequest request = new UpdateRequest("posts", "doc", "1")
        .doc("updated", new Date(),
             "reason", "daily update"); // 以 Object 键值对提供局部文档源，它将被转换为 JSON 格式。
```

### 更新或插入

如果文档不存在，可以使用upsert方法定义一些将作为新文档插入的内容：

```
String jsonString = "{\"created\":\"2017-01-01\"}";
request.upsert(jsonString, XContentType.JSON);  // 以字符串提供更新插入的文档源
与局部文档更新类似，可以使用接受 String， Map ， XContentBuilder 或 Object 键值对的方式使用upsert 方法更新或插入文档的内容。
```

### 可选参数

提供下列可选参数：

```
request.routing("routing");  // 路由值
request.parent("parent");  //Parent 值
request.timeout(TimeValue.timeValueMinutes(2)); // TimeValue 类型的等待主分片可用的超时时间
request.timeout("2m");  // 字符串类型的等待主分片可用的超时时间
request.setRefreshPolicy(WriteRequest.RefreshPolicy.WAIT_UNTIL);  // Refresh policy as a WriteRequest.RefreshPolicy instance
request.setRefreshPolicy("wait_for");  // Refresh policy as a String
request.retryOnConflict(3);  //	How many times to retry the update operation if the document to update has been changed by another operation between the get and indexing phases of the update operation
request.fetchSource(true); //Enable source retrieval, disabled by default
request.version(2); // 版本号
request.detectNoop(false);  // Disable the noop detection
request.scriptedUpsert(true); // Indicate that the script must run regardless of whether the document exists or not, ie the script takes care of creating the document if it does not already exist.
request.docAsUpsert(true); // Indicate that the partial document must be used as the upsert document if it does not exist yet.
request.waitForActiveShards(2);  //Sets the number of shard copies that must be active before proceeding with the update operation.
request.waitForActiveShards(ActiveShardCount.ALL); //Number of shard copies provided as a ActiveShardCount: can be ActiveShardCount.ALL, ActiveShardCount.ONE or ActiveShardCount.DEFAULT (default)
```

```
String[] includes = new String[]{"updated", "r*"};
String[] excludes = Strings.EMPTY_ARRAY;
request.fetchSource(new FetchSourceContext(true, includes, excludes)); // Configure source inclusion for specific fields
```

```
String[] includes = Strings.EMPTY_ARRAY;
String[] excludes = new String[]{"updated"};
request.fetchSource(new FetchSourceContext(true, includes, excludes)); //Configure source exclusion for specific fields
```

### 同步执行

```
UpdateResponse updateResponse = client.update(request);
```

### 异步执行

```
client.updateAsync(request, new ActionListener<UpdateResponse>() {
    @Override
    public void onResponse(UpdateResponse updateResponse) {
        //	Called when the execution is successfully completed. The response is provided as an argument.
    }
    @Override
    public void onFailure(Exception e) {
        // Called in case of failure. The raised exception is provided as an argument.
    }
});
```

## 更新响应

返回的UpdateResponse允许获取执行操作的相关信息，如下所示：

```
String index = updateResponse.getIndex();
String type = updateResponse.getType();
String id = updateResponse.getId();
long version = updateResponse.getVersion();
if (updateResponse.getResult() == DocWriteResponse.Result.CREATED) {
//Handle the case where the document was created for the first time (upsert)
} else if (updateResponse.getResult() == DocWriteResponse.Result.UPDATED) {
//	Handle the case where the document was updated
} else if (updateResponse.getResult() == DocWriteResponse.Result.DELETED) {
//Handle the case where the document was deleted
} else if (updateResponse.getResult() == DocWriteResponse.Result.NOOP) {
//Handle the case where the document was not impacted by the update, ie no operation (noop) was executed on the document
}
```
当通过 fetchSource 方法在UpdateRequest 里设置了启用接收源，相应对象将包含被更新的文档源：

```
GetResult result = updateResponse.getGetResult(); //Retrieve the updated document as a GetResult
if (result.isExists()) {
    String sourceAsString = result.sourceAsString(); //Retrieve the source of the updated document as a String
    Map<String, Object> sourceAsMap = result.sourceAsMap(); //Retrieve the source of the updated document as a Map<String, Object>
    byte[] sourceAsBytes = result.source(); //Retrieve the source of the updated document as a byte[]
} else {
    //Handle the scenario where the source of the document is not present in the response (this is the case by default)
}
```
这也可以用了检查分片故障：

```
ReplicationResponse.ShardInfo shardInfo = updateResponse.getShardInfo();
if (shardInfo.getTotal() != shardInfo.getSuccessful()) {
    //Handle the situation where number of successful shards is less than total shards
}
if (shardInfo.getFailed() > 0) {
    for (ReplicationResponse.ShardInfo.Failure failure : shardInfo.getFailures()) {
        String reason = failure.reason(); //Handle the potential failures
    }
}
```
当对一个不存在的文档执行 UpdateRequest 时，响应将包含 404 状态码，并抛出一个需要如下所示处理的 ElasticsearchException 异常：

```
UpdateRequest request = new UpdateRequest("posts", "type", "does_not_exist").doc("field", "value");
try {
    UpdateResponse updateResponse = client.update(request);
} catch (ElasticsearchException e) {
    if (e.status() == RestStatus.NOT_FOUND) {
        // 处理由于文档不存在导致的异常。
    }
}
```
如果有文档版本冲突，也会抛出 ElasticsearchException:

```
UpdateRequest request = new UpdateRequest("posts", "doc", "1")
        .doc("field", "value")
        .version(1);
try {
    UpdateResponse updateResponse = client.update(request);
} catch(ElasticsearchException e) {
    if (e.status() == RestStatus.CONFLICT) {
        // 表明异常是由于返回来了版本冲突错误导致的
    }
}
```