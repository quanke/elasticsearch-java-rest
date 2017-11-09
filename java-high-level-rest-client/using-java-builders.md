使用 Java 建造者

Java高级REST客户端依赖于 Elasticsearch 核心项目提供的不同类型的 Java Builders 对象，包括：

Query Builders
The query builders are used to create the query to execute within a search request. There is a query builder for every type of query supported by the Query DSL. Each query builder implements the QueryBuilder interface and allows to set the specific options for a given type of query. Once created, the QueryBuilder object can be set as the query parameter of SearchSourceBuilder. The Search Request page shows an example of how to build a full search request using SearchSourceBuilder and QueryBuilder objects. The Building Search Queries page gives a list of all available search queries with their corresponding QueryBuilder objects and QueryBuilders helper methods.

Aggregation Builders
Similarly to query builders, the aggregation builders are used to create the aggregations to compute during a search request execution. There is an aggregation builder for every type of aggregation (or pipeline aggregation) supported by Elasticsearch. All builders extend the AggregationBuilder class (or PipelineAggregationBuilderclass). Once created, AggregationBuilder objects can be set as the aggregation parameter of SearchSourceBuilder. There is a example of how AggregationBuilder objects are used with SearchSourceBuilder objects to define the aggregations to compute with a search query in Search Request page. The Building Aggregations page gives a list of all available aggregations with their corresponding AggregationBuilder objects and AggregationBuilders helper methods.
