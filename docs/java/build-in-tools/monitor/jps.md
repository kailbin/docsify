# 简介

jps 可以列出当前系统的Java进程

# 语法：
`jps [ options ] [ hostid ]`

`jps [-q] [-mlvV] [<hostid>]`    

# options
``` bash
-l          显示main方法包名 或者 应用jar包的全路径
-m          显示main方法参数
-v          显示JVM参数

-help       帮助信息
-q          只列出进程ID

-V          通过 flags 文件传递给JVM的参数 (the .hotspotrc file or the file specified by the -XX:Flags=<filename> argument)
-J          给jps命令传递JVM参数，例如：jps -J-Xms2m 分配了 2M 起始内存
```

# hostid
如果监控本地 JVM进程的话就是 进程ID；
如果进行远程监控，详请查看 **[jstatd 例子](build-in-tools/jstatd.md)部分**


# 官方文档

 > jps - Java Virtual Machine Process Status Tool [JDK7](https://docs.oracle.com/javase/7/docs/technotes/tools/share/jps.html)/[JDK8](http://docs.oracle.com/javase/8/docs/technotes/tools/windows/jps.html)

JDK7 和 JDK8 对 jps 的解释还是稍有不同的。  




