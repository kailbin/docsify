
# maven-dependency-plugin


## 将依赖的jar包拷贝到 lib 目录下面

``` xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <executions>
        <execution>
            <id>copy</id>
            <phase>package</phase>
            <goals>
                <goal>copy-dependencies</goal>
            </goals>
            <configuration>
                <outputDirectory>
                    <!--拷贝到构建目录的lib文件夹下 -->
                    ${project.build.directory}/lib
                </outputDirectory>
            </configuration>
        </execution>
    </executions>
</plugin>
```

## 查看树形依赖

```
mvn dependency:tree
```

## 查看依赖列表

```
mvn dependency:list
```



## 其他

- `dependency:copy`  用来拷贝某一个或多个特定文件到指定目录；

- `dependency:copy-dependencies`  用来拷贝依赖的文件到指定目录；

- `dependency:unpack`  与 copy 类似，但会解压文件；

- `dependency:unpack-dependencies` 与 copy-dependencies 类似，但会解压文件；




## Read More
- [Apache Maven Dependency Plugin](http://maven.apache.org/plugins/maven-dependency-plugin/)
- [查看maven项目的依赖关系 mvn dependency:tree](https://www.cnblogs.com/ghj1976/p/5336923.html)