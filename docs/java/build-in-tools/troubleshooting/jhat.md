
`jhat` 可以对 dump 出来的堆信息进行处理，以 html 页面的形式展示出来。

执行 `jhat /tmp/123.hprof`即可，默认端口是 `7000`，访问 `http://localhost:7000` 即可查看结果。

通过 `-port` 指定端口。

有时候文件过大的时候可能会出错，可以通过`-J-Xmx1024m`修改JVM最大可用内存。

其他参数详请查看官方文档。

# Read More

> [jhat](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/jhat.html)官方文档
>
> [ jhat中的OQL（对象查询语言](http://blog.csdn.net/gtuu0123/article/details/6039592)

