
# Bulk API

注意： Java高级REST客户端提供批量处理器来协助大量请求

## Bulk 请求

BulkRequest 可以被用在使用单个请求执行多个 索引，更新 和/或 删除 操作的情况下。

它要求至少要一个操作被添加到 Bulk 请求上：

```
BulkRequest request = new BulkRequest();  // 创建 BulkRequest
request.add(new IndexRequest("posts", "doc", "1")  // 添加第一个 IndexRequest 到 Bulk 请求上， 参看 [Index API](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-document-index.html) 获取更多关于如何构建 IndexRequest 的信息.
        .source(XContentType.JSON,"field", "foo"));
request.add(new IndexRequest("posts", "doc", "2")   // 添加第二个 IndexRequest
        .source(XContentType.JSON,"field", "bar"));
request.add(new IndexRequest("posts", "doc", "3")  // 添加第三个 IndexRequest
        .source(XContentType.JSON,"field", "baz"));
```

> 警告: Bulk API仅支持以JSON或SMILE编码的文档。 以任何其他格式提供文件将导致错误。

不同的操作类型也可以添加到同一个BulkRequest中：

```
BulkRequest request = new BulkRequest();
request.add(new DeleteRequest("posts", "doc", "3")); //
Adds a DeleteRequest to the BulkRequest. See Delete API for more information on how to build DeleteRequest.
request.add(new UpdateRequest("posts", "doc", "2")
        .doc(XContentType.JSON,"other", "test")); //
Adds an UpdateRequest to the BulkRequest. See Update API for more information on how to build UpdateRequest.
request.add(new IndexRequest("posts", "doc", "4")   //Adds an IndexRequest using the SMILE format
        .source(XContentType.JSON,"field", "baz"));
```

### 可选参数

提供下列可选参数：

```
request.timeout(TimeValue.timeValueMinutes(2));
request.timeout("2m");
Timeout to wait for the bulk request to be performed as a TimeValue
Timeout to wait for the bulk request to be performed as a String
request.setRefreshPolicy(WriteRequest.RefreshPolicy.WAIT_UNTIL);
request.setRefreshPolicy("wait_for");                            
Refresh policy as a WriteRequest.RefreshPolicy instance
Refresh policy as a String
request.waitForActiveShards(2);
request.waitForActiveShards(ActiveShardCount.ALL);
Sets the number of shard copies that must be active before proceeding with the index/update/delete operations.
Number of shard copies provided as a ActiveShardCount: can be ActiveShardCount.ALL, ActiveShardCount.ONE or ActiveShardCount.DEFAULT (default)
```

### 同步执行

```
BulkResponse bulkResponse = client.bulk(request);
```

### 异步执行

```
client.bulkAsync(request, new ActionListener<BulkResponse>() {
    @Override
    public void onResponse(BulkResponse bulkResponse) {
        //Called when the execution is successfully completed. The response is provided as an argument and contains a list of individual results for each operation that was executed. Note that one or more operations might have failed while the others have been successfully executed.
    }
    @Override
    public void onFailure(Exception e) {
        //Called when the whole BulkRequest fails. In this case the raised exception is provided as an argument and no operation has been executed.
    }
});
```

## Bulk 响应

返回的BulkResponse包含有关执行操作的信息，并允许对每个结果进行迭代，如下所示：

```
for (BulkItemResponse bulkItemResponse : bulkResponse) { //迭代所有操作的结果
    DocWriteResponse itemResponse = bulkItemResponse.getResponse(); //Retrieve the response of the operation (successful or not), can be IndexResponse, UpdateResponse or DeleteResponse which can all be seen as DocWriteResponse instances
    if (bulkItemResponse.getOpType() == DocWriteRequest.OpType.INDEX
            || bulkItemResponse.getOpType() == DocWriteRequest.OpType.CREATE) {
                //	Handle the response of an index operation
        IndexResponse indexResponse = (IndexResponse) itemResponse;
    } else if (bulkItemResponse.getOpType() == DocWriteRequest.OpType.UPDATE) {
        //Handle the response of a update operation
        UpdateResponse updateResponse = (UpdateResponse) itemResponse;
    } else if (bulkItemResponse.getOpType() == DocWriteRequest.OpType.DELETE) {
        //	Handle the response of a delete operation
        DeleteResponse deleteResponse = (DeleteResponse) itemResponse;
    }
}
```
批量响应提供了一种快速检查一个或多个操作是否失败的方法：

```
if (bulkResponse.hasFailures()) { // 只要有一个操作失败了，这个方法就返回 true
}
```

在这种情况下，有必要迭代所有运算结果，以检查操作是否失败，如果是，则检索相应的故障：

```
for (BulkItemResponse bulkItemResponse : bulkResponse) {
    if (bulkItemResponse.isFailed()) { //Indicate if a given operation failed
        BulkItemResponse.Failure failure = bulkItemResponse.getFailure(); //Retrieve the failure of the failed operation
    }
}
```

## Bulk 处理器

BulkProcessor通过提供允许索引/更新/删除操作在添加到处理器时透明执行的实用程序类来简化Bulk API的使用。

为了执行请求，BulkProcessor需要3个组件：


- RestHighLevelClient

这个客户端用来执行 BulkRequest 并接收 BulkResponse 。

- BulkProcessor.Listener

这个监听器会在每个 BulkRequest 执行之前和之后被调用，或者当 BulkRequest 失败时调用。

- ThreadPool

BulkRequest执行是使用这个池的线程完成的，允许BulkProcessor以非阻塞的方式工作，并允许在批量请求执行的同时接受新的索引/更新/删除请求。

然后 BulkProcessor.Builder 类可以被用来构建新的 BulkProcessor ：
```
ThreadPool threadPool = new ThreadPool(settings); // 使用已有的 Settings 对象创建 ThreadPool。
BulkProcessor.Listener listener = new BulkProcessor.Listener() { // 创建 BulkProcessor.Listener
    @Override
    public void beforeBulk(long executionId, BulkRequest request) {
        //This method is called before each execution of a BulkRequest
    }
    @Override
    public void afterBulk(long executionId, BulkRequest request, BulkResponse response) {
        //	This method is called after each execution of a BulkRequest
    }
    @Override
    public void afterBulk(long executionId, BulkRequest request, Throwable failure) {
        //This method is called when a BulkRequest failed
    }
};
BulkProcessor bulkProcessor = new BulkProcessor.Builder(client::bulkAsync, listener, threadPool)
        .build(); // 通过调用 BulkProcessor.Builder 的 build() 方法创建 BulkProcessor。 RestHighLevelClient.bulkAsync() 将被用来执行 BulkRequest。
```

BulkProcessor.Builder 提供了方法来配置 BulkProcessor 应该如何处理请求的执行：

```
BulkProcessor.Builder builder = new BulkProcessor.Builder(client::bulkAsync, listener, threadPool);
builder.setBulkActions(500);  //Set when to flush a new bulk request based on the number of actions currently added (defaults to 1000, use -1 to disable it)
builder.setBulkSize(new ByteSizeValue(1L, ByteSizeUnit.MB));  //	Set when to flush a new bulk request based on the size of actions currently added (defaults to 5Mb, use -1 to disable it)
builder.setConcurrentRequests(0); //Set the number of concurrent requests allowed to be executed (default to 1, use 0 to only allow the execution of a single request)
builder.setFlushInterval(TimeValue.timeValueSeconds(10L)); //	Set a flush interval flushing any BulkRequest pending if the interval passes (defaults to not set)
builder.setBackoffPolicy(BackoffPolicy.constantBackoff(TimeValue.timeValueSeconds(1L), 3)); //Set a constant back off policy that initially waits for 1 second and retries up to 3 times. See BackoffPolicy.noBackoff(), BackoffPolicy.constantBackoff() and BackoffPolicy.exponentialBackoff() for more options.
```
一旦创建了BulkProcessor，可以向其添加请求：

```
IndexRequest one = new IndexRequest("posts", "doc", "1").
        source(XContentType.JSON, "title", "In which order are my Elasticsearch queries executed?");
IndexRequest two = new IndexRequest("posts", "doc", "2")
        .source(XContentType.JSON, "title", "Current status and upcoming changes in Elasticsearch");
IndexRequest three = new IndexRequest("posts", "doc", "3")
        .source(XContentType.JSON, "title", "The Future of Federated Search in Elasticsearch");
bulkProcessor.add(one);
bulkProcessor.add(two);
bulkProcessor.add(three);
```

这些请求将由 BulkProcessor 执行，它负责为每个批量请求调用 BulkProcessor.Listener 。
监听器提供方法接收 BulkResponse 和 BulkResponse ：

```
BulkProcessor.Listener listener = new BulkProcessor.Listener() {
    @Override
    public void beforeBulk(long executionId, BulkRequest request) {
        int numberOfActions = request.numberOfActions(); //Called before each execution of a BulkRequest, this method allows to know the number of operations that are going to be executed within the BulkRequest
        logger.debug("Executing bulk [{}] with {} requests", executionId, numberOfActions);
    }
    @Override
    public void afterBulk(long executionId, BulkRequest request, BulkResponse response) {
        if (response.hasFailures()) {
            //在每个 BulkRequest 执行之后调用，此方法允许获知 BulkResponse 是否包含错误
            logger.warn("Bulk [{}] executed with failures", executionId);
        } else {
            logger.debug("Bulk [{}] completed in {} milliseconds", executionId, response.getTook().getMillis());
        }
    }
    @Override
    public void afterBulk(long executionId, BulkRequest request, Throwable failure) {
        logger.error("Failed to execute bulk", failure); //如果 BulkRequest 执行失败则调用，此方法可获知失败情况。
    }
};
```
一旦将所有请求都添加到BulkProcessor，其实例需要使用两种可用的关闭方法之一关闭。

一旦所有请求都被添加到了 BulkProcessor, 它的实例就需要使用两种可用的关闭方法之一进行关闭。

awaitClose() 可以被用来等待到所有请求都被处理，或者到指定的等待时间：

```
boolean terminated = bulkProcessor.awaitClose(30L, TimeUnit.SECONDS);
```

如果所有批量请求完成，则该方法返回 true ，如果在完成所有批量请求之前等待时间过长，则返回 false 。

close() 方法可以被用来立即关闭 BulkProcessor ：

```
bulkProcessor.close();
```

在关闭处理器之前，两个方法都会刷新已经被天教导处理器的请求，并禁止添加任何新的请求。
