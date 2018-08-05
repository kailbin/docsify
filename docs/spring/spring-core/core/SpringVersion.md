# SpringVersion

该类用来获取 Spring 版本

实现如下：

```java
package org.springframework.core;

/**
 * Class that exposes the Spring version. Fetches the
 * "Implementation-Version" manifest attribute from the jar file.
 *
 * @since 1.1
 */
public class SpringVersion {

	public static String getVersion() {
		Package pkg = SpringVersion.class.getPackage();
		return (pkg != null ? pkg.getImplementationVersion() : null);
	}

}

```

读取的是 `spring-core-4.3.10.RELEASE.jar/META-INF/MANIFEST.MF` 文件中的 `Implementation-Version` 内容

```txt
Manifest-Version: 1.0
Implementation-Title: spring-core
Implementation-Version: 4.3.10.RELEASE
Created-By: 1.8.0_121 (Oracle Corporation)
```



## Read More

- JDK6 Doc： [java.lang.Package](http://tool.oschina.net/uploads/apidocs/jdk-zh/java/lang/Package.html)