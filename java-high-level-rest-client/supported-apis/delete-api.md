
# Delete API

## 删除请求对象

DeleteRequest 需要下列参数：

```
DeleteRequest request = new DeleteRequest(
        "posts",    // index
        "doc",     //Type
        "1");  // Document id

```
可选参数

## 提供下列可选参数：

```
request.routing("routing");  // 路由值
request.parent("parent");  //Parent 值
request.timeout(TimeValue.timeValueMinutes(2)); // TimeValue 类型的等待主分片可用的超时时间
request.timeout("2m");  // 字符串类型的等待主分片可用的超时时间
request.setRefreshPolicy(WriteRequest.RefreshPolicy.WAIT_UNTIL);  // Refresh policy as a WriteRequest.RefreshPolicy instance
request.setRefreshPolicy("wait_for");  // Refresh policy as a String
request.version(2);   // Version
request.versionType(VersionType.EXTERNAL);  // Version type
```

## 同步执行

```
DeleteResponse deleteResponse = client.delete(request);
```

## 异步执行

```
client.deleteAsync(request, new ActionListener<DeleteResponse>() {
    @Override
    public void onResponse(DeleteResponse deleteResponse) {
        //Called when the execution is successfully completed. The response is provided as an argument
    }
    @Override
    public void onFailure(Exception e) {
        //Called in case of failure. The raised exception is provided as an argument
    }
});
```

## 删除操作的响应

返回的 DeleteResponse 对象允许通过执行以下操作获取相关信息：

```
String index = deleteResponse.getIndex();
String type = deleteResponse.getType();
String id = deleteResponse.getId();
long version = deleteResponse.getVersion();
ReplicationResponse.ShardInfo shardInfo = deleteResponse.getShardInfo();
if (shardInfo.getTotal() != shardInfo.getSuccessful()) {
// Handle the situation where number of successful shards is less than total shards
}
if (shardInfo.getFailed() > 0) {
    for (ReplicationResponse.ShardInfo.Failure failure : shardInfo.getFailures()) {
        String reason = failure.reason();  // Handle the potential failures
    }
}
```
也可以检查文档是否被发现：

```
DeleteRequest request = new DeleteRequest("posts", "doc", "does_not_exist");
DeleteResponse deleteResponse = client.delete(request);
if (deleteResponse.getResult() == DocWriteResponse.Result.NOT_FOUND) {
    //如果被删除的文档没有被找到，做某些操作
}
```

如果有版本冲突，将抛出 ElasticsearchException 异常信息：

```
try {
    DeleteRequest request = new DeleteRequest("posts", "doc", "1").version(2);
    DeleteResponse deleteResponse = client.delete(request);
} catch (ElasticsearchException exception) {
    if (exception.status() == RestStatus.CONFLICT) {
        //由于版本冲突错误导致的异常
    }
}
```