
# Search API

## 搜索请求

SearchRequest用于与搜索文档，聚合，建议有关的任何操作，并且还提供了在生成的文档上请求突出显示的方法。
在最基本的形式中，我们可以向请求添加一个查询：

```
SearchRequest searchRequest = new SearchRequest(); //穿件SeachRequest，Without arguments this runs against all indices.
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();// 大多数的搜索参数被添加到 SearchSourceBuilder 。它为每个进入请求体的每个东西都提供 setter 方法。
searchSourceBuilder.query(QueryBuilders.matchAllQuery());  // 添加一个 match_all 查询到 searchSourceBuilder 。

```

### 可选参数

我们先看一下SearchRequest的一些可选参数：

```
SearchRequest searchRequest = new SearchRequest("posts"); // 限制请求到某个索引上
searchRequest.types("doc");  // 限制请求的类别
```

还有一些其他有趣的可选参数：

```
searchRequest.routing("routing");  // 设置路由参数、
searchRequest.indicesOptions(IndicesOptions.lenientExpandOpen()); // 设置IndicesOptions控制如何解析不可用索引以及扩展通配符表达式
searchRequest.preference("_local"); //使用首选参数，例如，执行搜索优先选择本地分片。 默认值是随机化分片。
```

### 使用 SearchSourceBuilder

可以在SearchSourceBuilder上设置大多数控制搜索行为的选项，其中包含或多或少相当于 Rest API 的搜索请求正文中的选项。

以下是一些常见选项的示例：

```
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder(); //使用默认选项创建 SearchSourceBuilder 。
sourceBuilder.query(QueryBuilders.termQuery("user", "kimchy")); //设置查询对象。可以使任何类型的 QueryBuilder
sourceBuilder.from(0); //设置from选项，确定要开始搜索的结果索引。 默认为0。
sourceBuilder.size(5); //设置大小选项，确定要返回的搜索匹配数。 默认为10。
sourceBuilder.timeout(new TimeValue(60, TimeUnit.SECONDS)); //设置一个可选的超时时间，用于控制搜索允许的时间。
```

构建查询

搜索查询可以使用 QueryBuilder 对象创建。 对于Elasticsearch的 Query DSL 支持的每个搜索查询类型，都存在QueryBuilder。
QueryBuilder 可以使用它的构造器创建：

```
//创建一个字段“user”与文本“kimchy”相匹配的的全文匹配查询。
MatchQueryBuilder matchQueryBuilder = new MatchQueryBuilder("user", "kimchy");
```

创建后，QueryBuilder对象提供了配置搜索查询选项的方法：

```
matchQueryBuilder.fuzziness(Fuzziness.AUTO); //在匹配查询上启用模糊匹配
matchQueryBuilder.prefixLength(3);  //在匹配查询上设置前缀长度
matchQueryBuilder.maxExpansions(10); //设置最大扩展选项以控制查询的模糊过程

```

QueryBuilder 对象也可以使用 QueryBuilders 工具类创建。这个类提供了使用链式编程风格的辅助方法来创建 QueryBuilder 对象：

```
QueryBuilder matchQueryBuilder = QueryBuilders.matchQuery("user", "kimchy")
                                                .fuzziness(Fuzziness.AUTO)
                                                .prefixLength(3)
                                                .maxExpansions(10);

```

无论用于创建它的方法如何， QueryBuilder 对象必须添加到 SearchSourceBuilder 中，如下所示：

```
searchSourceBuilder.query(matchQueryBuilder);
```

“构建查询”页面列出了所有可用的搜索查询及其对应的QueryBuilder对象和QueryBuilders辅助方法。

### 指定排序

SearchSourceBuilder允许添加一个或多个SortBuilder实例。 有四个特殊的实现（Field-，Score-，GeoDistance-和ScriptSortBuilder）。

```
sourceBuilder.sort(new ScoreSortBuilder().order(SortOrder.DESC));  // 按_score降序排序（默认值）
sourceBuilder.sort(new FieldSortBuilder("_uid").order(SortOrder.ASC));  //也按_id字段排序升序
```

源过滤


默认情况下，搜索请求返回文档的内容，_source但像 Rest API 中的内容一样，您可以覆盖此行为。例如，您可以完全关闭 _source 检索：

```
sourceBuilder.fetchSource(false);
```

该方法还接受一个或多个通配符模式的数组，以便以更精细的方式控制哪些字段包含或排除：

```
String[] includeFields = new String[] {"title", "user", "innerObject.*"};
String[] excludeFields = new String[] {"_type"};
sourceBuilder.fetchSource(includeFields, excludeFields);
```

### 请求高亮

突出显示搜索结果可以通过设置来实现HighlightBuilder的 SearchSourceBuilder。可以通过向HighlightBuilder.Fielda 添加一个或多个实例来为每个字段定义不同的突出显示行为HighlightBuilder。

```
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
HighlightBuilder highlightBuilder = new HighlightBuilder();//Creates a new HighlightBuilder
HighlightBuilder.Field highlightTitle =
        new HighlightBuilder.Field("title");//Create a field highlighter for the title field
highlightTitle.highlighterType("unified");  //Set the field highlighter type
highlightBuilder.field(highlightTitle);  //Add the field highlighter to the highlight builder
HighlightBuilder.Field highlightUser = new HighlightBuilder.Field("user");
highlightBuilder.field(highlightUser);
searchSourceBuilder.highlighter(highlightBuilder);
```

有很多选项，这在Rest API文档中有详细的介绍。Rest API参数（例如，pre_tags）通常由具有相似名称的 setter（例如#preTags（String …））更改。

随后可以从 SearchResponse 中检索突出显示的文本片段。

### 请求聚合

可以通过首先创建适当的集合AggregationBuilder然后在其上设置集合来将搜索添加到搜索结果中 SearchSourceBuilder。在下面的示例中，我们terms在公司名称上创建一个聚合，其中包含公司员工平均年龄的子聚合：

```
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
TermsAggregationBuilder aggregation = AggregationBuilders.terms("by_company")
        .field("company.keyword");
aggregation.subAggregation(AggregationBuilders.avg("average_age")
        .field("age"));
searchSourceBuilder.aggregation(aggregation);
```

“ 构建聚合”页面提供了所有可用聚合以及其相应AggregationBuilder对象和AggregationBuilders帮助方法的列表。

后面我们将看到如何访问聚合的 SearchResponse。

### 请求建议

要向搜索请求添加建议，请使用SuggestionBuilder从SuggestBuilders工厂类轻松访问的其中一个实现。建议构建者需要添加到顶层SuggestBuilder，本身可以设置在 顶层SearchSourceBuilder。

```
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
SuggestionBuilder termSuggestionBuilder =
    SuggestBuilders.termSuggestion("user").text("kmichy"); //
Creates a new TermSuggestionBuilder for the user field and the text kmichy
SuggestBuilder suggestBuilder = new SuggestBuilder();
suggestBuilder.addSuggestion("suggest_user", termSuggestionBuilder); //Adds the suggestion builder and names it suggest_user
searchSourceBuilder.suggest(suggestBuilder);
```

后面我们将看到如何从 SearchResponse 获取建议。

### 自定义查询和聚合

该配置文件API可用于简档查询和聚集的执行针对特定搜索请求。为了使用它，配置文件标志必须设置为true SearchSourceBuilder：

```
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
searchSourceBuilder.profile(true);
```

一旦SearchRequest执行，相应的SearchResponse将 包含分析结果。

### 同步执行

SearchRequest以下列方式执行时，客户端SearchResponse在继续执行代码之前等待返回：

```
SearchResponse searchResponse = client.search(searchRequest);
```

### 异步执行

```
client.searchAsync(searchRequest, new ActionListener<SearchResponse>() {
    @Override
    public void onResponse(SearchResponse searchResponse) {
        //执行成功完成时调用。
    }
    @Override
    public void onFailure(Exception e) {
        //当整个SearchRequest失败时调用。
    }
});
```
## 搜索响应

通过执行搜索返回的 SearchResponse 提供了关于搜索执行本身以及对返回的文档的访问的详细信息。 首先，有关于请求执行本身的有用信息，如HTTP状态代码，执行时间或请求提前终止或超时：

```
RestStatus status = searchResponse.status();
TimeValue took = searchResponse.getTook();
Boolean terminatedEarly = searchResponse.isTerminatedEarly();
boolean timedOut = searchResponse.isTimedOut();
```

其次，响应还通过提供关于搜索影响的分片总数以及成功与不成功的分片的统计信息，提供关于分片级别执行的信息。 可能的故障也可以通过遍历ShardSearchFailures上的数组进行迭代来处理，如下例所示：

```
int totalShards = searchResponse.getTotalShards();
int successfulShards = searchResponse.getSuccessfulShards();
int failedShards = searchResponse.getFailedShards();
for (ShardSearchFailure failure : searchResponse.getShardFailures()) {
    // 故障应该在这里被处理
}
```

### 检索 SearchHits

要访问返回的文档，我们需要首先得到响应中包含的 SearchHits ：

```
SearchHits hits = searchResponse.getHits();
```

将SearchHits提供命中的所有全局信息，比如命中总数或最大分数：

```
long totalHits = hits.getTotalHits();
float maxScore = hits.getMaxScore();
```

嵌套在 SearchHits 的各个搜索结果可以被迭代访问：

```
SearchHit[] searchHits = hits.getHits();
for (SearchHit hit : searchHits) {
    // do something with the SearchHit
}
```

SearchHit可以访问基本信息，如索引，类型，文档ID 以及每个搜索匹配的分数：

```
String index = hit.getIndex();
String type = hit.getType();
String id = hit.getId();
float score = hit.getScore();
```

此外，它可以让您将文档源作为简单的JSON-String或键/值对的映射。 在该映射中，字段名为键，含字段值为值。 多值字段作为对象的列表返回，嵌套对象作为另一个键/值映射。 这些情况需要相应地执行：

```
String sourceAsString = hit.getSourceAsString();
Map<String, Object> sourceAsMap = hit.getSourceAsMap();
String documentTitle = (String) sourceAsMap.get("title");
List<Object> users = (List<Object>) sourceAsMap.get("user");
Map<String, Object> innerObject = (Map<String, Object>) sourceAsMap.get("innerObject");
Retrieving Highlighting
```

如果请求，可以从SearchHit结果中的每一个中检索突出显示的文本片段。命中对象提供对HighlightField实例的字段名称映射的访问，每个实例都包含一个或多个突出显示的文本片段：

```
SearchHits hits = searchResponse.getHits();
for (SearchHit hit : hits.getHits()) {
    Map<String, HighlightField> highlightFields = hit.getHighlightFields();
    HighlightField highlight = highlightFields.get("title"); //获取该title领域 的突出显示
    Text[] fragments = highlight.fragments();  //获取包含突出显示的字段内容的一个或多个片段
    String fragmentString = fragments[0].string();
}
```

### 检索聚合

可以SearchResponse通过首先获取聚合树的根，Aggregations对象，然后通过名称获取聚合来检索聚合。

```
Aggregations aggregations = searchResponse.getAggregations();
Terms byCompanyAggregation = aggregations.get("by_company"); //Get the by_company terms aggregation
Bucket elasticBucket = byCompanyAggregation.getBucketByKey("Elastic"); //
Avg averageAge = elasticBucket.getAggregations().get("average_age"); //Get the average_age sub-aggregation from that bucket
double avg = averageAge.getValue();
```

请注意，如果按名称访问聚合，则需要根据您请求的聚合类型指定聚合接口，否则将抛出 ClassCastException ：

```
//This will throw an exception because "by_company" is a terms aggregation but we try to retrieve it as a range aggregation
Range range = aggregations.get("by_company");
```

也可以以聚合名称键入的映射来访问所有聚合。在这种情况下，转换为适当聚合接口需要明确发生：

```
Map<String, Aggregation> aggregationMap = aggregations.getAsMap();
Terms companyAggregation = (Terms) aggregationMap.get("by_company");
```

还有getter 方法将所有顶级聚合作为列表返回：

```
List<Aggregation> aggregationList = aggregations.asList();
```

最后但并非最不重要的是，您可以迭代所有聚合，然后根据其类型决定如何进一步处理它们：

```
for (Aggregation agg : aggregations) {
    String type = agg.getType();
    if (type.equals(TermsAggregationBuilder.NAME)) {
        Bucket elasticBucket = ((Terms) agg).getBucketByKey("Elastic");
        long numberOfDocs = elasticBucket.getDocCount();
    }
}
```

### 检索建议

要从SearchResponse返回建议，请使用Suggest对象作为入口点，然后检索嵌套的建议对象：

```
Suggest suggest = searchResponse.getSuggest();// Use the Suggest class to access suggestions
TermSuggestion termSuggestion = suggest.getSuggestion("suggest_user");// Suggestions can be retrieved by name. You need to assign them to the correct type of Suggestion class (here TermSuggestion), otherwise a ClassCastException is thrown
for (TermSuggestion.Entry entry : termSuggestion.getEntries()) {// Iterate over the suggestion entries
    for (TermSuggestion.Entry.Option option : entry) {// Iterate over the options in one entry
        String suggestText = option.getText().string();
    }
}
```

### 检索分析结果

从SearchResponse使用该getProfileResults()方法检索分析结果。此方法返回Map包含执行中ProfileShardResult涉及的每个分片的对象 SearchRequest。ProfileShardResult存储在Map使用唯一地标识分析结果对应的分片的密钥。

这是一个示例代码，显示如何遍历每个分片的所有分析结果：

```
Map<String, ProfileShardResult> profilingResults = searchResponse.getProfileResults(); //Retrieve the Map of ProfileShardResult from the SearchResponse
for (Map.Entry<String, ProfileShardResult> profilingResult : profilingResults.entrySet()) {  //Profiling results can be retrieved by shard’s key if the key is known, otherwise it might be simpler to iterate over all the profiling results
    String key = profilingResult.getKey();//Retrieve the key that identifies which shard the ProfileShardResult belongs to
    ProfileShardResult profileShardResult = profilingResult.getValue();//Retrieve the ProfileShardResult for the given shard
}
```

ProfileShardResult对象本身包含一个或多个查询配置文件结果，一个针对基本的Lucene索引执行的每个查询：

```
List<QueryProfileShardResult> queryProfileShardResults = profileShardResult.getQueryProfileResults(); // 检索 QueryProfileShardResult 列表
for (QueryProfileShardResult queryProfileResult : queryProfileShardResults) {
//迭代每个 QueryProfileShardResult
}
```

每个都QueryProfileShardResult可以访问详细的查询树执行，作为ProfileResult对象列表返回 ：

```
for (ProfileResult profileResult : queryProfileResult.getQueryResults()) {//迭代配置文件结果
    String queryName = profileResult.getQueryName(); //检索Lucene查询的名称
    long queryTimeInMillis = profileResult.getTime(); //检索在执行Lucene查询时花费的时间
    List<ProfileResult> profiledChildren = profileResult.getProfiledChildren();//
检索子查询的配置文件结果（如果有）
}
```

Rest API文档包含有关查询分析信息描述的Profiling Queries的更多信息。

该 QueryProfileShardResult 还可以访问了Lucene的收藏家的分析信息：

```
CollectorResult collectorResult = queryProfileResult.getCollectorResult();  //检索Lucene收集器的分析结果
String collectorName = collectorResult.getName();  //检索Lucene 的名字
Long collectorTimeInMillis = collectorResult.getTime();//以毫秒计算的时间用于执行Lucene集合
List<CollectorResult> profiledChildren = collectorResult.getProfiledChildren();//
```

检索子集合的个人资料结果（如果有）
Rest API文档包含有关Lucene收集器的分析信息的更多信息 。

以与查询树执行非常相似的方式，QueryProfileShardResult对象可以访问详细的聚合树执行：

```
AggregationProfileShardResult aggsProfileResults = profileShardResult.getAggregationProfileResults(); // 检索 AggregationProfileShardResult
for (ProfileResult profileResult : aggsProfileResults.getProfileResults()) { //迭代聚合配置文件结果
    String aggName = profileResult.getQueryName(); //Retrieve the type of the aggregation (corresponds to Java class used to execute the aggregation)
    long aggTimeInMillis = profileResult.getTime();//Retrieve the time in millis spent executing the Lucene collector
    List<ProfileResult> profiledChildren = profileResult.getProfiledChildren(); //Retrieve the profile results for the sub-aggregations (if any)
}
```
Rest API文档包含更多关关于 Profiling Aggregations 的信息