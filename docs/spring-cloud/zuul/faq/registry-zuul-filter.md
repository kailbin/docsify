# 如何注册 Zuul 过滤器

从 [官方示例](https://github.com/Netflix/zuul/blob/1.x/zuul-simple-webapp/src/main/java/com/netflix/zuul/StartServer.java) 可以看出

## 注册静态 Filter

``` java
FilterRegistry registry = FilterRegistry.instance();
registry.put("filterName", new ZuulFilter() {
    ...
});
```

## 注册 Groovy Filter

``` java
FilterLoader.getInstance().setCompiler(new GroovyCompiler());

String scriptRoot = System.getProperty("zuul.filter.root", "");
if (scriptRoot.length() > 0) {
    scriptRoot = scriptRoot + File.separator;
}
try {
    FilterFileManager.setFilenameFilter(new GroovyFileFilter());
    // 每隔5秒遍历一下文件，查看文件是否更新
    FilterFileManager.init(5, scriptRoot + "pre", scriptRoot + "route", scriptRoot + "post");
} catch (Exception e) {
    throw new RuntimeException(e);
}
```

## Spring Cloud 是如何注册的

### ZuulServerAutoConfiguration.ZuulFilterConfiguration 类

```java
...

public class ZuulServerAutoConfiguration {
	
    ...

	@Configuration
	protected static class ZuulFilterConfiguration {

        /**
         * 如果Spring 容器中 ZuulFilter 有多个实现类，这里会注入 实现类 Bean 的名称 和 具体的实现类，
         * 即所有内置 和 自定义的 Filter 都会注入到 filters 变量
         */
		@Autowired
		private Map<String, ZuulFilter> filters;

		@Bean
		public ZuulFilterInitializer zuulFilterInitializer(
				CounterFactory counterFactory, TracerFactory tracerFactory
        ) {
			...
            // 传入所有 filters ，在 ZuulFilterInitializer 中进行注册
			return new ZuulFilterInitializer(this.filters, counterFactory, tracerFactory, filterLoader, filterRegistry);
		}

	}
    
    ...
        
}
```

### ZuulFilterInitializer 类

```java
public class ZuulFilterInitializer {
	
    ...
    
	private final FilterRegistry filterRegistry;

	...

	@PostConstruct
	public void contextInitialized() {
		
		...
		
		for (Map.Entry<String, ZuulFilter> entry : this.filters.entrySet()) {
            // 循环所有 filters 进行注册
			filterRegistry.put(entry.getKey(), entry.getValue());
		}
	}
    
    ...
    
}
```

