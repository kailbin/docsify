# rr

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

