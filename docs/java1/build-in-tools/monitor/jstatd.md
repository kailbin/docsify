
jstatd 会开启一个 RMI 服务，供其他机器进行远程监控

## 参数

``` bash
-nr     如果RMI注册中心没有找到，不会创建一个内部的RMI注册中心
-p      RMI端口，默认为1099
-n      RMI名称， 默认是 JStatRemoteHost
-J      传递JVM参数
```

## 例子

**文件 `jstatd.all.policy`内容如下，给jstatd授予所有权限**
``` 
grant codebase "file:${java.home}/../lib/tools.jar" {   
    permission java.security.AllPermission;
};
```

**开启jstatd 守护进程**，开启后会有一个光标一直在闪，`Ctrl + C` 退出
``` bash
jstatd -J-Djava.security.policy=jstatd.all.policy
```

**绑定端口和设置名称**
``` bash
jstatd -J-Djava.security.policy=jstatd.all.policy -p 6789 -n rmiJstatsName
```

**使用jps远程监控**
假设 jstatd 的所在机器是 `192.168.4.35`
``` bash
jps rmi://192.168.4.35:6789/rmiJstatsName
```
输出结果

    11452 RemoteMavenServer
    4440 Jstatd
    10016 Launcher

**使用 jstat 远程监控**
``` bash
jstat -gcutil 11452@192.168.4.35:6789/rmiJstatsName 500 10
```

**打开日志输出**
``` bash
jstatd -J-Djava.security.policy=jstatd.all.policy -J-Djava.rmi.server.logCalls=true
```

## 详请查看以下文档

> 官网 [jstatd](http://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstatd.html)
>
> [jstatd 命令(Java Statistics Monitoring Daemon) ](http://blog.csdn.net/fenglibing/article/details/17323515)
>
> [Default Policy Implementation and Policy File Syntax](http://docs.oracle.com/javase/8/docs/technotes/guides/security/PolicyFiles.html)



