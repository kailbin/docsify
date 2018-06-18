
`jinfo` 可以**打印**或者**修改** Java进程 的配置信息。配置信息包括 Java 系统属性 和 JVM flags（ Java system properties and Java Virtual Machine (JVM) command-line flags）。

# 查看帮助

`jinfo`、`jinfo -h`、`jinfo -help` 三种方法都可以打印出帮助信息。

``` bash
$ jinfo
Usage:
    jinfo [option] <pid>
        (to connect to running process)
    jinfo [option] <executable <core>
        (to connect to a core file)
    jinfo [option] [server_id@]<remote server IP or hostname>
        (to connect to remote debug server)

where <option> is one of:
    -flag <name>         to print the value of the named VM flag
    -flag [+|-]<name>    to enable or disable the named VM flag
    -flag <name>=<value> to set the named VM flag to the given value
    -flags               to print VM flags
    -sysprops            to print Java system properties
    <no option>          to print both of the above
    -h | -help           to print this help message

```

# 系统配置

`jinfo <pid> `
 
或者指定属性配置

`jinfo <pid> | grep java.version`

# command-line flags

**获取**
``` bash
$ jcmd 56227 VM.flags 
56227:
-XX:InitialHeapSize=268435456 -XX:MaxHeapSize=805306368 -XX:+UseCompressedOops -XX:+UseParallelGC 
```

``` bash
$ jinfo -flag MaxHeapSize 56227
-XX:MaxHeapSize=805306368
```

## 参考

> 官方文档 [jinfo](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/jinfo.html)

