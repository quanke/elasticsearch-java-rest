
# Clear Scroll API

使用 Search Scroll API 的搜索上下文在超过滚动时间时，会自动清除。但是，建议当不再需要使用 Clear Scroll API 后尽可能快的释放搜索上下文。

## Clear Scroll 请求

ClearScrollRequest 可以像如下方式创建:

```
ClearScrollRequest request = new ClearScrollRequest();  // 创建一个新的 ClearScrollRequest
request.addScrollId(scrollId); // 添加一个滚动id到要清除的滚动标志列表里
https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-clear-scroll.html
```