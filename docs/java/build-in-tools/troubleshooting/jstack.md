
`jstack` 可以打印出 线程的状态、调用栈、锁资源 等信息。 可以用于分析死锁、性能瓶颈等问题。用法相对简单但非常有用。

# 参数

```` bash
-F          当 jstack [-l] <pid> 无相应的时候可以使用该参数强制dump

-l          打印关于锁的附加信息,例如属于java.util.concurrent的ownable synchronizers列表

-m          打印 java 和 native 代码的所有栈信息

-h|-help    打印帮助信息
````

# Read More

> [jstack](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/jstack.html)官方文档
>
> [JAVA线程dump的分析 --- jstack pid](http://www.blogjava.net/jzone/articles/303979.html)




