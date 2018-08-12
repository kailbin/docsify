# ResourceUtils



```java
package org.springframework.util;

import java.io.File;
import java.io.FileNotFoundException;
import java.net.MalformedURLException;
import java.net.URI;
import java.net.URISyntaxException;
import java.net.URL;
import java.net.URLConnection;

/**
 * @since 1.1.5
 * @see org.springframework.core.io.Resource
 * @see org.springframework.core.io.ClassPathResource
 * @see org.springframework.core.io.FileSystemResource
 * @see org.springframework.core.io.UrlResource
 * @see org.springframework.core.io.ResourceLoader
 */
public abstract class ResourceUtils {

	public static final String CLASSPATH_URL_PREFIX = "classpath:";
	public static final String FILE_URL_PREFIX = "file:";
	public static final String JAR_URL_PREFIX = "jar:";
	public static final String WAR_URL_PREFIX = "war:";
    
    ...
	
    /** Separator between JAR URL and file path within the JAR: "!/" */
	public static final String JAR_URL_SEPARATOR = "!/";
	/** Special separator between WAR URL and jar part on Tomcat */
	public static final String WAR_URL_SEPARATOR = "*/";


	/**
	 * 检验字符串是否 符合 URL 规范
	 * 1. 如果字符串以 classpath: 开头，返回true
	 * 2. new URL(resourceLocation) , 如果不抛异常返回 true
	 */
	public static boolean isUrl(String resourceLocation) { .. }

	/**
	 * 把 资源路径解析成 URL 对象
	 * 1. 如果字符串以 classpath: 开头，ClassLoader.getSystemResource(path)
	 * 2. new URL(resourceLocation)
	 */
	public static URL getURL(String resourceLocation) throws FileNotFoundException { .. }

	/**
	 * 1. 如果字符串以 classpath: 开头，getFile(url, 异常描述的一部分)
	 * 2. getFile(new URL(resourceLocation))
	 */
	public static File getFile(String resourceLocation) throws FileNotFoundException { .. }

	public static File getFile(URL resourceUrl) throws FileNotFoundException  { .. }

	public static File getFile(URL resourceUrl, String description) throws FileNotFoundException { .. }

	public static File getFile(URI resourceUri) throws FileNotFoundException { .. }

	public static File getFile(URI resourceUri, String description) throws FileNotFoundException { .. }

	/**
	 * 如果协议是 "file", "vfsfile" or "vfs" 返回 true
	 */
	public static boolean isFileURL(URL url) { .. }

	/**
	 * 如果协议是 "jar", "war, ""zip", "vfszip" or "wsjar"  返回 true
	 */
	public static boolean isJarURL(URL url) { .. }

	/**
	 * 如果协议是 "file" 并且 ".jar" 为后缀名.
	 * @since 4.1
	 */
	public static boolean isJarFileURL(URL url) { .. }

	/**
	 * 获取 Jar 的资源路径
	 * jar:file:/tmp/myjar.jar!/myentry.txt   --> file:/tmp/myjar.jar
	 * /tmp/asd.jar!/mypath/myjar.jar  -->  file:/tmp/asd.jar
	 */
	public static URL extractJarFileURL(URL jarUrl) throws MalformedURLException { .. }

	/**
	 * 获取 压缩包 的资源路径
	 * jar:file:/tmp/myjar.jar!/myentry.txt   --> file:/tmp/myjar.jar
	 * /tmp/asd.jar!/mypath/myjar.jar  -->  file:/tmp/asd.jar
	 * war:file:...mywar.war*/WEB-INF/lib/myjar.jar!/myentry.txt  --> file:...mywar.war
     *    
	 * @since 4.1.8
	 */
	public static URL extractArchiveURL(URL jarUrl) throws MalformedURLException { .. }

	/**
	 * toURI(url.toString())
	 */
	public static URI toURI(URL url) throws URISyntaxException { .. }

	public static URI toURI(String location) throws URISyntaxException {
		return new URI(StringUtils.replace(location, " ", "%20"));
	}

	public static void useCachesIfNecessary(URLConnection con) {
		con.setUseCaches(con.getClass().getSimpleName().startsWith("JNLP"));
	}

}

```

