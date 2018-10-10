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

   String CLASSPATH_ALL_URL_PREFIX = "classpath*:"
   // 新增方法，相比 ResourceLoader 可以加载多个资源
   Resource[] getResources(String locationPattern) throws IOException;
}
```

`ResourcePatternResolver` 的继承关系如下：

```ba&#39;s
ResourcePatternResolver (org.springframework.core.io.support)
    PathMatchingResourcePatternResolver (org.springframework.core.io.support)
    ApplicationContext (org.springframework.context)
        ConfigurableApplicationContext (org.springframework.context)
        	AbstractApplicationContext (org.springframework.context.support)
				...
```

以 `PathMatchingResourcePatternResolver` 为例，`getResources` 方法的实现如下

```java
@Override
public Resource[] getResources(String locationPattern) throws IOException {
    Assert.notNull(locationPattern, "Location pattern must not be null");
    // 处理 classpath*:
    if (locationPattern.startsWith(CLASSPATH_ALL_URL_PREFIX)) {
        // 是否包含通配符
        if (getPathMatcher().isPattern(locationPattern.substring(CLASSPATH_ALL_URL_PREFIX.length()))) {
            // 切割出classpath*: 递归调用 getResources 方法
            return findPathMatchingResources(locationPattern);
        }
        else {
            //
            // 区别在这 -> 对空字符串的处理是返回 .../target/classes 和 urlClassLoader.getURLs()，即包含jar包
            //
            return findAllClassPathResources(locationPattern.substring(CLASSPATH_ALL_URL_PREFIX.length()));
        }
    }
    // 处理 classpath:
    else {
        int prefixEnd = (locationPattern.startsWith("war:") ? locationPattern.indexOf("*/") + 1 :
                         locationPattern.indexOf(':') + 1);
        // 是否包含通配符
        if (getPathMatcher().isPattern(locationPattern.substring(prefixEnd))) {
            // 切割出classpath: 递归调用 getResources 方法
            return findPathMatchingResources(locationPattern);
        }
        else {
            //
            // 区别在这 -> 对空字符串的处理是 .../target/classes
            //
            return new Resource[] {getResourceLoader().getResource(locationPattern)};
        }
    }
}
```



# 所涉及到的知识点

## 获取类路径下所有 jar

```java
URLClassLoader urlClassLoader = (URLClassLoader) ClassUtils.getDefaultClassLoader();
// 包含 JDK 和 Maven 依赖的 jar 和 classpath跟路径，如：
// file:/Library/Java/JavaVirtualMachines/jdk1.8.0_131.jdk/Contents/Home/jre/lib/rt.jar
// file:/Users/kail/.m2/repository/org/springframework/spring-core/4.3.19.RELEASE/spring-core-4.3.19.RELEASE.jar
// file:/Users/kail/IdeaProjects/xxx/target/classes/ 
URL[] urLs = urlClassLoader.getURLs();
```

## 使用到的 Spring 工具类

- [AntPathMatcher](/spring-core/util/AntPathMatcher) 按照Ant的风格来来进行模糊匹配
- [ResourceUtils](/spring-core/util/ResourceUtils) 判断 `URL` 的类型，如：`isJarFileURL` 方法来判断 `URL` 是否是一个 jar 包
	- 如果 jar 包，通过 `JarFile` 遍历包内的所有文件，通过 [AntPathMatcher](/spring-core/util/AntPathMatcher) 进行匹配
	- 如果是本地文件路径，直接循环遍历 类路径下的所有文件，通过 [AntPathMatcher](/spring-core/util/AntPathMatcher) 进行匹配
	- ...

## classpath: 和 classpath*: 的区别
- `classpath:` 不包含jar包的遍历，只遍历本地文件系统
- `classpath*:` 遍历 jar包内的文件 和 本地文件系统
- `:` 后面是 Ant的风格 的匹配规则，都通过 [AntPathMatcher](/spring-core/util/AntPathMatcher.md) 进行匹配，两者没有区别



