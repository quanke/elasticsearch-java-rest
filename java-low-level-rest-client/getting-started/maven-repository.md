# Maven Repository

托管在
[ Maven Central](http://search.maven.org/#search%7Cga%7C1%7Cg%3A%22org.elasticsearch.client%22)

Java 版本最低要求 1.7 

> `low-level` REST 客户端与 `elasticsearch` 的发布周期相同。可以使用版本替换，但必须是 5.0.0-alpha4 之后的版本。客户端版本与 `Elasticsearch` 服务版本之间没有关联。`low-level` REST 客户端兼容所有 Elasticsearch 版本。


## Maven 配置

使用 Maven 作依赖管理,将下列内容添加到你的 pom.xml 文件里：

```
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-client</artifactId>
    <version>5.6.4</version>
</dependency>
```

## Gradle 配置

使用 gradle 作依赖管理，将下列内容添加到你的 build.gradle 文件里：

```
dependencies {
    compile 'org.elasticsearch.client:elasticsearch-rest-client:5.6.4'
}
```