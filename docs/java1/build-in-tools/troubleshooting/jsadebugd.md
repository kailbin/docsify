
`jsadebugd`依附到一个Java进程，开启一个rmi服务，担当一个调试服务器的作用，可供 `jinfo`、`jmap`、`jstack` 命令拉取远程机器上的信息。

# 开启服务

``` bash
$ jsadebugd <pid>
Attaching to process ID <pid> and starting RMI services, please wait...
Debugger attached and RMI services started.
```
开启之后可以通过 `Ctrl + C` 关闭停止

# 远程连接

``` bash
$ jinfo localhost
  
Attaching to remote server localhost, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 24.75-b04
Java System Properties:
  
idea.version = =2016.3.3
java.runtime.name = Java(TM) SE Runtime Environment
java.vm.version = 24.75-b04
  
......

```
`jmap`、`jstack` 类似。


## 参考
> [jsadebugd](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/jsadebugd.html)官方文档

