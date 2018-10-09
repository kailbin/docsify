# ResourcePatternResolver

`ResourceLoader` 提供 classpath 下单资源文件的载入，而 `ResourcePatternResolver` 提供了多资源文件的载入

```java
package org.springframework.core.io.support;

import java.io.IOException;

import org.springframework.core.io.Resource;
import org.springframework.core.io.ResourceLoader;

/**
 * @since 1.0.2
 * @see org.springframework.core.io.Resource
 * @see org.springframework.core.io.ResourceLoader
 * @see org.springframework.context.ApplicationContext
 * @see org.springframework.context.ResourceLoaderAware
 */
public interface ResourcePatternResolver extends ResourceLoader {

   /**
     Pseudo URL prefix for all matching resources from the class path: "classpath*:"
     This differs from ResourceLoader's classpath URL prefix in that it
     retrieves all matching resources for a given name (e.g. "/beans.xml"),
     for example in the root of all deployed JAR files.
    */
   String CLASSPATH_ALL_URL_PREFIX = "classpath*:";

   /**
    * Resolve the given location pattern into Resource objects.
    * <p>Overlapping resource entries that point to the same physical
    * resource should be avoided, as far as possible. The result should
    * have set semantics.
    * @param locationPattern the location pattern to resolve
    * @return the corresponding Resource objects
    * @throws IOException in case of I/O errors
    */
   Resource[] getResources(String locationPattern) throws IOException;

}
```

`ResourcePatternResolver` 的继承关系如下

```text
org.springframework.core.io.ResourceLoader
|- org.springframework.core.io.support.ResourcePatternResolver
|- |- org.springframework.core.io.support.PathMatchingResourcePatternResolver
|- |- |- org.springframework.web.context.support.ServletContextResourcePatternResolver
```



```
ClassLoader defaultClassLoader = ClassUtils.getDefaultClassLoader();
System.out.println(defaultClassLoader.getClass());
URLClassLoader urlClassLoader = (URLClassLoader) defaultClassLoader;
for (URL url : urlClassLoader.getURLs()) {
    System.out.println(url);
}
```

## Read More

- [Spring ResourceLoader.getResource() & getResources()的理解](https://blog.csdn.net/supportuat/article/details/50931311)

