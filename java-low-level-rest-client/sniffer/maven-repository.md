# Maven 仓库

REST 客户端嗅探器与 elasticsearch 的发行周期相同。可以使用期望的嗅探器版本替换，但必须是 5.0.0-alpha4 之后的版本。嗅探器版本与其通信的 Elasticsearch 版本之间没有关联。嗅探器支持从 elasticsearch 2.x 及以上的版本上获取节点列表。

## Maven 配置

若使用 Maven 作依赖管理，你可以这样配置依赖。将下列内容添加到你的 pom.xml 文件里：

```

<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-client-sniffer</artifactId>
    <version>5.6.4</version>
</dependency>
```

## Gradle 配置

若使用 gradle 作依赖管理，你可以这样配置依赖。将下列内容添加到你的 build.gradle 文件里：

```
dependencies {
    compile 'org.elasticsearch.client:elasticsearch-rest-client-sniffer:5.6.0'
}
```

