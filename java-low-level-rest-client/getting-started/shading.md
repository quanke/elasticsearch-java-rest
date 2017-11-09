# Shading


为了避免版本冲突，可以在单个JAR文件（有时称为“uber JAR”或“fat JAR”）中将客户端中的依赖项隐藏并打包在客户端中。 `Shading JAR `可以通过`Gradle`和`Maven`的第三方插件完成。

请注意，Shading JAR也有影响。 例如，Shading Commons Logging层意味着第三方日志记录后端也需要被shaded 。

## Maven 配置

你可以这样配置 Maven Shade 插件 。将下列内容添加到你的 pom.xml 文件里：

```
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.1.0</version>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals><goal>shade</goal></goals>
                    <configuration>
                        <relocations>
                            <relocation>
                                <pattern>org.apache.http</pattern>
                                <shadedPattern>hidden.org.apache.http</shadedPattern>
                            </relocation>
                            <relocation>
                                <pattern>org.apache.logging</pattern>
                                <shadedPattern>hidden.org.apache.logging</shadedPattern>
                            </relocation>
                            <relocation>
                                <pattern>org.apache.commons.codec</pattern>
                                <shadedPattern>hidden.org.apache.commons.codec</shadedPattern>
                            </relocation>
                            <relocation>
                                <pattern>org.apache.commons.logging</pattern>
                                <shadedPattern>hidden.org.apache.commons.logging</shadedPattern>
                            </relocation>
                        </relocations>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

## Gradle 配置

你可以这样配置 Gradle [ShadowJar](https://github.com/johnrengelman/shadow) 插件. 添加以下内容到你的 build.gradle 文件中：

```
shadowJar {
    relocate 'org.apache.http', 'hidden.org.apache.http'
    relocate 'org.apache.logging', 'hidden.org.apache.logging'
    relocate 'org.apache.commons.codec', 'hidden.org.apache.commons.codec'
    relocate 'org.apache.commons.logging', 'hidden.org.apache.commons.logging'
}

```