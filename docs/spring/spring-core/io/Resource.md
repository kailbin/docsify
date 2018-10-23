# Resource

Resource 是 Spring **对于资源文件的统一访问接口**。

```java
package org.springframework.core.io;

/**
 * @since 28.12.2003
 * @see #getInputStream()
 * @see #getURL()
 * @see #getURI()
 * @see #getFile()
 * @see WritableResource
 * @see ContextResource
 * @see UrlResource
 * @see ClassPathResource
 * @see FileSystemResource
 * @see PathResource
 * @see ByteArrayResource
 * @see InputStreamResource
 */
public interface Resource extends InputStreamSource {

	/**
	 * 判断对应的资源是否真的存在
	 */
	boolean exists();

	/**
	 * 用于判断对应资源的内容是否可读。
	 * 需要注意的是当其结果为true的时候，其内容未必真的可读，但如果返回false，则其内容必定不可读
	 */
	boolean isReadable();

	/**
	 * 用于判断当前资源是否代表一个已打开的输入流，
	 * 如果结果为true，则表示当前资源的输入流不可多次读取，且在读取以后需要关闭，以防止内存泄露
	 */
	boolean isOpen();

	/**
	 * 返回当前资源对应的URL
	 * 如果当前资源不能解析为一个URL则会抛出异常。如ByteArrayResource就不能解析为一个URL
	 */
	URL getURL() throws IOException;

	/**
	 * 返回当前资源对应的URI
	 * 如果当前资源不能解析为一个URI则会抛出异常
	 * @since 2.5
	 */
	URI getURI() throws IOException;

	/**
	 * 返回当前资源对应的File。如果当前资源不能以绝对路径解析为一个File则会抛出异常
	 */
	File getFile() throws IOException;

	/**
	 * 资源文本大小
	 */
	long contentLength() throws IOException;

	/**
	 * 定义该资源后更新的时间戳
	 */
	long lastModified() throws IOException;

	/**
	 * 创建一个资源，相对对该资源未知
	 */
	Resource createRelative(String relativePath) throws IOException;

	/**
	 * 定义此资源的文件名，通常是最后一个路径的一部分：例如，“myfile.txt”。
	 * 如果此类资源没有文件名，则返回 null
	 */
	String getFilename();

	/**
	 * 返回此资源的描述，用于在处理资源错误时输出
	 * 鼓励使用该方法的返回值来作为 toString 的实现
	 */
	String getDescription();

}

```



其主要实现有

```bash
Resource (org.springframework.core.io)
    ContextResource (org.springframework.core.io)
    WritableResource (org.springframework.core.io)
        FileSystemResource (org.springframework.core.io)
        PathResource (org.springframework.core.io)
    AbstractResource (org.springframework.core.io)
        AbstractFileResolvingResource (org.springframework.core.io)
            UrlResource (org.springframework.core.io)
            ClassPathResource (org.springframework.core.io)
        InputStreamResource (org.springframework.core.io)
        FileSystemResource (org.springframework.core.io)
        VfsResource (org.springframework.core.io)
        ByteArrayResource (org.springframework.core.io)
        PathResource (org.springframework.core.io)

```

**Resource** 可以单独使用，也可以通过 **ResourceLoader** 使用。

```java
import org.springframework.core.io.*;

import java.io.IOException;

public class Main {

    public static void main(String[] args) throws IOException {
        ResourceLoader resourceLoader = new ClassRelativeResourceLoader(Main.class);
        Resource resource = resourceLoader.getResource("/Main.class");
        // class path resource [/Main.class]
        System.out.println(resource.toString());
        // class path resource [/Main.class]
        System.out.println(resource.getDescription());
        // true
        System.out.println(resource.exists());
        // class org.springframework.core.io.ClassRelativeResourceLoader$ClassRelativeContextResource
        System.out.println(resource.getClass());
        // 1314
        System.out.println(resource.contentLength());
        // true
        System.out.println(resource.exists());
    }
}

```







- [通过Spring Resource接口获取资源](http://elim.iteye.com/blog/2016305)
- [Spring ResourceLoader.getResource() & getResources()的理解](https://blog.csdn.net/supportuat/article/details/50931311)