# ResourceLoader

``` text
ResourceLoader (org.springframework.core.io) interface
    DefaultResourceLoader (org.springframework.core.io) class
    ResourcePatternResolver (org.springframework.core.io.support) interface
        ...
```

ResourceLoader 是一个接口，定义了从 类路径 和 文件系统 中加载资源。
作用主要是 **对各种资源路径（location）进行封装，返回 `org.springframework.core.io.Resource` 接口的实现，通过统一的方式进行访问**

``` java
package org.springframework.core.io;

import org.springframework.util.ResourceUtils;

public interface ResourceLoader {

	/** Pseudo URL prefix for loading from the class path: "classpath:" */
	String CLASSPATH_URL_PREFIX = ResourceUtils.CLASSPATH_URL_PREFIX;

	Resource getResource(String location);

	ClassLoader getClassLoader();

}
```

ResourceLoader 有一个默认实现 DefaultResourceLoader，大致策略是：
- 如果注册的有 `ProtocolResolver` 并且其返回的资源 非null，直接通过 `ProtocolResolver` 获取资源
- 如果以 `/` 开头，直接返回 `ClassPathContextResource`
- 如果以 `classpath:` 开头，如果是则去掉`classpath:` 前缀返回对应的 `ClassPathResource`
- 否则就把它当做一个 `URL` 来处理，封装成一个 `UrlResource` 进行返回
- 如果当成 `URL` 处理也失败的话就把对应的资源当成是一个 `ClassPathContextResource` 进行返回。

