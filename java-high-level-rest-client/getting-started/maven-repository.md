
# Maven 仓库

高级 Java REST 客户端被托管在 Maven 中央仓库里。所需的最低Java版本为 1.8。

高级 REST 客户端与 elasticsearch 的发行周期相同。可以使用期望的版本进行替换。

## Maven 配置

若使用 Maven 作依赖管理，你可以这样配置依赖。将下列内容添加到你的 pom.xml 文件里：

```
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
    <version>5.6.0</version>
</dependency>
```

## Gradle 配置

若使用 gradle 作依赖管理，你可以这样配置依赖。将下列内容添加到你的 build.gradle 文件里：

```
dependencies {
    compile 'org.elasticsearch.client:elasticsearch-rest-high-level-client:5.6.0'
}
```