
> Use these commands to troubleshoot Java applications and the Java Virtual Machine (JVM). Most of these commands are unsupported and experimental and might not be available in future JDK releases.

可以使用下面这个工具类来排查 Java 应用和JVM的问题。这些命令大部分是不受支持的，有可能在以后的JDK版本中被移除

> - jcmd: Sends diagnostic command requests to a running JVM.
> - jinfo: Experimental. Generates configuration information.
> - jhat: Experimental. Analyzes the Java heap.
> - jmap: Experimental. Prints shared object memory maps or heap memory details for a process, core file, or remote debug server.
> - jsadebugd: Experimental. Attaches to a Java process or core file and acts as a debug server.
> - jstack: Experimental. Prints Java thread stack traces for a Java process, core file, or remote debug server.

- `jcmd`: 向运行中的 JVM 发送诊断命令
- `jinfo`: [**试验性的**]获得 JVM 配置信息
- `jhat`:  [**试验性的**]方便析 Java 堆 的工具
- `jmap`:  [**试验性的**]主要用于打印Java进程的 堆 信息
- `jsadebugd`: [**试验性的**]依附到一个Java进程或核心文件，担当一个调试服务器的作用
- `jstack`:  [**试验性的**]主要用于打印Java进程的 栈 信息





# Read More
> [Troubleshooting](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/s11-troubleshooting_tools.html)