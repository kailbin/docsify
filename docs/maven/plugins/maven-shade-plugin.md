
# maven-shade-plugin

该插件可以将一个项目的 所有文件和依赖，打到一个jar包中。
需要注意是，**所有的 依赖项 及其配置 都会被拆解**放置，**依赖 jar 解压后，如果文件同名，可能会相互覆盖**。


``` xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>3.1.0</version>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <!-- 不生成 dependency-reduced-pom.xml 文件 -->
                <createDependencyReducedPom>false</createDependencyReducedPom>

                <transformers>
                    <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                        <manifestEntries>
                            <!-- 指定Main 方法 -->
                            <Main-Class>xyz.kail.demo.Main</Main-Class>
                            <!-- 自定义信息 -->
                            <X-Compile-Source-JDK>1.7</X-Compile-Source-JDK>
                            <X-Compile-Target-JDK>1.7</X-Compile-Target-JDK>
                        </manifestEntries>
                    </transformer>
                </transformers>

            </configuration>
        </execution>
    </executions>
</plugin>
```

## 查看帮助
``` bash
mvn help:describe -Dplugin=shade -Ddetail=true -Dgoal=shade
```


## Read More

- [Executable JAR](http://maven.apache.org/plugins/maven-shade-plugin/examples/executable-jar.html)
- [Selecting Contents for Uber JAR](http://maven.apache.org/plugins/maven-shade-plugin/examples/includes-excludes.html)
- [Apache Maven Shade Plugin](http://maven.apache.org/plugins/maven-shade-plugin/index.html)