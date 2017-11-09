
# Search Scroll API

Scroll API可用于从搜索请求中检索大数量的结果。

为了使用滚动，需要按照给定的顺序执行以下步骤。

## 初始化搜索滚动上下文

包含一个 scroll 参数的初始化搜索请求必须通过执行 Search API 初始化滚动回话。 在处理此SearchRequest时，Elasticsearch将检测滚动参数的存在，并使搜索上下文保持相应的时间间隔。

```
SearchRequest searchRequest = new SearchRequest("posts");
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
searchSourceBuilder.query(matchQuery("title", "Elasticsearch"));
searchSourceBuilder.size(size); //Create the SearchRequest and its corresponding SearchSourceBuilder. Also optionally set the size to control how many results to retrieve at a time.
searchRequest.source(searchSourceBuilder);
searchRequest.scroll(TimeValue.timeValueMinutes(1L)); // Set the scroll interval
SearchResponse searchResponse = client.search(searchRequest);
String scrollId = searchResponse.getScrollId(); // Read the returned scroll id, which points to the search context that’s being kept alive and will be needed in the following search scroll call
SearchHits hits = searchResponse.getHits();  // Retrieve the first batch of search hits
```

## 检索所有相关文档

其次， 接收到的滚动标识符必须被设置到下一个新的滚动间隔的 SearchScrollRequest， 并通过 searchScroll 方法发送。Elasticsearch会使用新的滚动标识符返回另一批结果。 然后可以在随后的SearchScrollRequest中使用此新的滚动标识符来检索下一批结果，等等。应该循环重复此过程，直到不再返回结果，这意味着滚动已经用尽，并且已经检索到所有匹配的文档。

```
SearchScrollRequest scrollRequest = new SearchScrollRequest(scrollId);  //Create the SearchScrollRequest by setting the required scroll id and the scroll interval
scrollRequest.scroll(TimeValue.timeValueSeconds(30));
SearchResponse searchScrollResponse = client.searchScroll(scrollRequest);
scrollId = searchScrollResponse.getScrollId();   //	Read the new scroll id, which points to the search context that’s being kept alive and will be needed in the following search scroll call
hits = searchScrollResponse.getHits(); //Retrieve another batch of search hits <4>
assertEquals(3, hits.getTotalHits());
assertEquals(1, hits.getHits().length);
assertNotNull(scrollId);
```

## 清除滚动上下文

最后，可以使用Clear Scroll API删除最后一个滚动标识符，以释放搜索上下文。 当滚动到期时，会自动发生，但最佳实践是当滚动会话结束后尽快释放资源。

## 可选参数

构建 SearchScrollRequest 是可以选择使用以下参数：

```
scrollRequest.scroll(TimeValue.timeValueSeconds(60L));  // 	Scroll interval as a TimeValue
scrollRequest.scroll("60s"); // Scroll interval as a String
```

## 同步执行

```
SearchResponse searchResponse = client.searchScroll(scrollRequest);
```

## 异步执行

```
client.searchScrollAsync(scrollRequest, new ActionListener<SearchResponse>() {
    @Override
    public void onResponse(SearchResponse searchResponse) {
        // 当执行成功的时候调用，响应对象以参数的形式传入
    }
    @Override
    public void onFailure(Exception e) {
        // 失败时调用，异常以参数形式传入
    }
});
```

## 响应

与 Search API 一样，滚动搜索 API 也返回一个 SearchResponse 对象

### 完整示例

下面是一个滚动搜索的完整示例：
```
final Scroll scroll = new Scroll(TimeValue.timeValueMinutes(1L));
SearchRequest searchRequest = new SearchRequest("posts");
searchRequest.scroll(scroll);
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
searchSourceBuilder.query(matchQuery("title", "Elasticsearch"));
searchRequest.source(searchSourceBuilder);
SearchResponse searchResponse = client.search(searchRequest); // 通过发送初始化 SearchRequest 来初始化搜索上下文
String scrollId = searchResponse.getScrollId();
SearchHit[] searchHits = searchResponse.getHits().getHits();
while (searchHits != null && searchHits.length > 0) { //在一个循环中通过调用 Search Scroll api 检索所有搜索命中结果，知道没有文档返回为止。
    //创建一个新的SearchScrollRequest，持有最近一次返回的滚动标识符和滚动间隔
    SearchScrollRequest scrollRequest = new SearchScrollRequest(scrollId);
    scrollRequest.scroll(scroll);
    searchResponse = client.searchScroll(scrollRequest);
    scrollId = searchResponse.getScrollId();
    searchHits = searchResponse.getHits().getHits();
//处理返回的搜索结果
}
ClearScrollRequest clearScrollRequest = new ClearScrollRequest(); //一旦滚动完成，清除滚动上下文
clearScrollRequest.addScrollId(scrollId);
ClearScrollResponse clearScrollResponse = client.clearScroll(clearScrollRequest);
boolean succeeded = clearScrollResponse.isSucceeded();
```
