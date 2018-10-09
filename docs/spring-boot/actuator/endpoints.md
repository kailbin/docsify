

# Endpoints


Actuator endpoints 允许对你的应用进行监控和交互(interact). Spring Boot包括许多内置的 endpoints，您也可以添加自己的 endpoints. 例如，`/health`endpoints 提供基本的应用程序健康信息。

endpoints 暴露的方式将取决于您选择的技术类型。 大多数应用程序选择HTTP监控，其中 endpoints 的ID被映射到一个URL。 例如，默认情况下，健康 endpoints 将被映射到 `/health`。


|  ID  |  Description  |  Sensitive Default  |
|:------|--------------|:-------------:|
|  `actuator`  |  Provides a hypermedia-based “discovery page” for the other endpoints. Requires Spring HATEOAS to be on the classpath.   |  true  |
|  `auditevents`  |  为当前应用暴露审计事件（audit events）信息  |  true  |
|  `autoconfig`  |  展示自动化配置报告 ，所有自动化配置的候选项， 为什么被应用或者不被应用等.  |  true  |
|  `beans`  |  显示应用程序中所有Spring bean的完整列表  |  true  |
|  `configprops`  |  显示所有`@ConfigurationProperties`的整理列表。  |  true  |
|  `dump`  |  执行线程转储.  |  true  |
|  `env`  |  暴露 Spring 的 ConfigurableEnvironment 中的属性.  |  true  |
|  `flyway`  |  Shows any Flyway database migrations that have been applied.  |  true  |
|  `health`  |  展示应用健康信息 (当应用是安全(secure)时, 在未经身份验证访问时，只有一个简单的 `status` 字段 ，身份验证之后或者关掉安全选项，可以获取到一个完整的列表).  |  false  |  
|  `info`  |  展示自定义的应用信息.  |  false  |
|  `loggers`  |  显示和**修改**应用程序中 logger 的配置.  |  true  |
|  `liquibase`  |  Shows any Liquibase database migrations that have been applied.  |  true  |
|  `metrics`  |  显示当前应用程序的 `metrics` 信息.  |  true  |
|  `mappings`  |  显示所有 `@RequestMapping` 的路径列表.  |  true  |
|  `shutdown`  |  允许应用程序优雅的关闭 (默认是不开启的).  |  true  |
|  `trace`  |  显示追踪信息 (默认显示最近100个HTTP请求).  |  true  |


如果使用Spring MVC，还可以使用以下附加端点:

|  ID  |  Description  |  Sensitive Default  |
|:------|--------------|:-------------:|
|  `docs`  |  Displays documentation, including example requests and responses, for the Actuator’s endpoints. Requires spring-boot-actuator-docs to be on the classpath.  |  false  |
|  `heapdump`  |  返回一个`GZip`压缩的`hprof`堆转储文件。 |  true  |
|  `jolokia`  |  通过HTTP公开JMX bean(当Jolokia在类路径上时).  |  true  |
|  `logfile`  |  返回日志文件的内容(如果 `logging.file` 或 `logging.path` 属性已经设置的话). 支持使用HTTP范围头来检索日志文件内容的一部分。  |  true  |


## 依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

## 关闭安全
```
# 关闭安全认证（生产环境不建议关闭）
management.security.enabled=false
```

## 

## Read More
- [47. Endpoints](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#production-ready-endpoints)