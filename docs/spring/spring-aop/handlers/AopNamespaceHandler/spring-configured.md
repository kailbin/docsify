# &lt;aop: spring-configured/&gt;

当我们使用 Spring 容器的时候，一般是从 Spring 容器中获取 Bean，Spring 会自动进行 依赖注入。

`<aop:spring-configured/>` 的作用是，提供了能**为任何对象进行依赖注入的能力**，场景是在脱离容器管理而创建的对象时进行依赖注入，如：

- 脱离Spring 容器进行 事务管理
- 其他... 详见 spring-aspects.xxx.jar:/META-INF/aop.xml 中的支持



原理是 AspectJ 的 编译期 或 类加载时 织入，new 对象前已经在 class字节码中 织入了相关代码。

所以需要 AspectJ 提供的 `ajc` 编译器进行编译（也可通过 Maven 插件进行编译），也可 通过 LTW 机制进行使用。

## AspectJ Maven 插件进行编译（编译期织入）

官方：http://www.mojohaus.org/aspectj-maven-plugin/index.html

示例：

```xml
<plugins>
    <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>aspectj-maven-plugin</artifactId>
        <version>1.11</version>
        <configuration>
            <outxmlfile>META-INF/aop.xml</outxmlfile>
            <aspectLibraries>
                <aspectLibrary>
                    <groupId>org.springframework</groupId>
                    <artifactId>spring-aspects</artifactId>
                </aspectLibrary>
            </aspectLibraries>
            <complianceLevel>1.8</complianceLevel>
            <source>1.8</source>
            <target>1.8</target>
            <forceAjcCompile>true</forceAjcCompile>
        </configuration>
        <executions>
            <execution>
                <goals>
                    <goal>compile</goal>
                    <goal>test-compile</goal>
                </goals>
            </execution>
        </executions>
        <dependencies>
            <dependency>
                <groupId>org.aspectj</groupId>
                <artifactId>aspectjrt</artifactId>
                <version>${aspectj.version}</version>
            </dependency>
            <dependency>
                <groupId>org.aspectj</groupId>
                <artifactId>aspectjtools</artifactId>
                <version>${aspectj.version}</version>
            </dependency>
        </dependencies>
    </plugin>
</plugins>
```

## LTW(类加载期织入)

详见：[&lt;context: load-time-weaver/&gt;](spring-context/handlers/ContextNamespaceHandler/load-time-weaver.md)



## 使用步骤

- 添加依赖 `org.springframework:spring-aspects`（间接依赖 `org.aspectj:aspectjweaver` ）
- 添加依赖 `org.springframework:spring-tx`（依赖事务这个 jar 的原因是 `spring-aspects.jar:/META-INF/aop.xml` 中配置的  xxx.aj 使用到了 spring-tx 中的类）
- 为类加上 `@Configurable` 注解 进行标记
- 在类中 为被依赖的字段 加上 `@Autowired` 或 `@Resource` 注解（与 Spring 常规使用方式类似）

### 示例

被依赖的类，受 Spring 管理

```java
@Service
public class SomeService {
}
```

不受 Spring 管理的类

```java
@Configurable
public class NewClass {
    @Autowired
    private SomeService someService;
    
    public void test() {
        // 查看是否是 null
        System.out.println(someService);
    }
}
```

编译期织入方式运行

使用 `aspectj-maven-plugin` 插件，执行 `mvn clean test`

```java
@ComponentScan
@EnableSpringConfigured // AOP 方式依赖注入
@ContextConfiguration(classes = SpringDITest.class)
@RunWith(SpringJUnit4ClassRunner.class)
public class SpringDITest {

    @Test
    public void testDI() {
    	// SomeService@49139829
        System.out.println(new NewClass().someService);
    }
}
```

类加载期方式运行

添加 javaagent 参数 `-javaagent:~/.m2/repository/org/springframework/spring-instrument/4.3.20.RELEASE/spring-instrument-4.3.20.RELEASE.jar`

```java
@ComponentScan
@EnableSpringConfigured // AOP 方式依赖注入
@EnableLoadTimeWeaving // LTW
@ContextConfiguration(classes = SpringDITest.class)
@RunWith(SpringJUnit4ClassRunner.class)
public class SpringDITest {

    @Test
    public void testDI() {
        System.out.println(new NewClass().someService);
    }

}
```



## 开启方式

- `<aop:spring-configured/>` 或 `<content:spring-configured/>`
- `@EnableSpringConfigured` 注解
- `<bean class="org.springframework.beans.factory.aspectj.AnnotationBeanConfigurerAspect" factory-method="aspectOf"/>`



## 参考资料

- [6.8.1. 在Spring中使用AspectJ进行domain object的依赖注入](http://shouce.jb51.net/spring/aop.html#aop-atconfigurable)
- [示例项目：spring-aspectj-compiletime-weaving](https://github.com/Jotschi/spring-aspectj-compiletime-weaving)
- [使用Spring AOP向领域模型注入依赖](https://blog.csdn.net/zjerryj/article/details/77133267)

