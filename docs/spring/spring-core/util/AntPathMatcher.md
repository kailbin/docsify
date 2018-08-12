# AntPathMatcher

整个 Spring框架 的路径解析都是按照Ant的风格来的，如： package 匹配，Spring MVC 的路径匹配

## Ant 风格匹配规则

### 通配符

- `?` 匹配任何单字符
- `*` 匹配0或者任意数量的字符
- `**` 匹配0或者更多的目录



### 最长匹配原则

URL请求`/app/dir/file.jsp`，现在存在两个路径匹配模式`/**/*.jsp`和`/app/dir/*.jsp`，那么会根据模式 `/app/dir/*.jsp` 来匹配



### 示例

| URL路径              | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| `/app/*.x`           | 所有在app路径下的.x文件                                      |
| `/app/p?ttern`       | `/app/pattern` 和 `/app/pXttern`,但是不包括 `/app/pttern`    |
| `/**/example`        | `/app/example`, `/app/foo/example`, 和 `/example`            |
| `/app/**/dir/file.*` | `/app/dir/file.jsp`, `/app/foo/dir/file.html`,`/app/foo/bar/dir/file.pdf`, 和 `/app/dir/file.java` |
| `/**/*.jsp`          | 任何的 `.jsp` 文件                                           |

> [Ant 风格 是什么?](http://www.10tiao.com/html/261/201608/2652126095/1.html)



## PathMatcher 的 AntPathMatcher 实现

```java
package org.springframework.util;

import java.util.Comparator;
import java.util.Map;

/**
 *
 * Used by :
 * @link org.springframework.core.io.support.PathMatchingResourcePatternResolver
 * @link org.springframework.web.servlet.handler.AbstractUrlHandlerMapping
 * @link org.springframework.web.servlet.mvc.multiaction.PropertiesMethodNameResolver
 * @link org.springframework.web.servlet.mvc.WebContentInterceptor
 *
 * @since 1.2
 * @see AntPathMatcher
 */
public interface PathMatcher {

	boolean isPattern(String path);

	/**
	 * 最常用的方法， 判断匹配模式与路径是否匹配
	 */
	boolean match(String pattern, String path);

    boolean matchStart(String pattern, String path);

	/**
	 * '/docs/cvs/commit.html' and '/docs/cvs/commit.html -> ''
	 * 'docs/*' and '/docs/cvs/commit' -> 'cvs/commit'
	 * '/docs/cvs/*.html' and '/docs/cvs/commit.html' -> 'commit.html'
	 * '/docs/**' and '/docs/cvs/commit' -> 'cvs/commit'
	 * '/docs/**\/*.html' and '/docs/cvs/commit.html' -> 'cvs/commit.html'
	 * '/*.html' and '/docs/cvs/commit.html' -> 'docs/cvs/commit.html'
	 * '*.html' and '/docs/cvs/commit.html' -> '/docs/cvs/commit.html'
	 * '*' and '/docs/cvs/commit.html' -> '/docs/cvs/commit.html'
	 */
	String extractPathWithinPattern(String pattern, String path);

	/**
	 * Spring Web MVC 中经常用到，例子如下：
	 * 
	 * "/check/{id:[0-9]+}", "/check/123"  --> {id=123}
	 * "/check/{name}", "/check/kail"  --> {name=kail}
	 */
	Map<String, String> extractUriTemplateVariables(String pattern, String path);

    /**
	 * 用来计算最佳匹配
	 */
	Comparator<String> getPatternComparator(String path);

	/**
	 * 把两个 匹配模式 合并成 一个
	 * ("/**/lo?in.do", "/**/login.do")  -->  /**/login.do
     * ("/**/*.xml", "/**/*.properties")  -->  Cannot combine patterns: /**/*.xml vs /**/*.properties    
	 */
	String combine(String pattern1, String pattern2);

}

```



## Read More

- [SpringMVC路径匹配规则AntPathMatcher](https://blog.csdn.net/qq_21251983/article/details/53034425)
- [[Spring源码解析 - AntPathMatcher](https://www.cnblogs.com/leftthen/p/5212221.html)]