

# 内存区域设置
|    |    |
| --- | --- |
|  **-Xms4096M** |  初始化堆大小  |
|  **-Xmx4096M**  |  最大堆大小  |
|  -Xss1024k（-XX:ThreadStackSize=） |  每个线程的堆栈大小(建议不要超过1M)  |
|    |    |
|  *–Xmn1024M*  |  年轻代大小(官方推荐配置为整个堆的3/8)  |
|  *-XX:NewSize=1024M*  |  年轻代初始化大小 |
|  *-XX:MaxNewSize=1024M*  |  年轻代再打化大小 最大
|    |    |
|  **-XX:NewRatio=2**  |  年轻代:年老代=1:2，年轻代占整个堆的1/3 。**设置该值的情况会忽略掉 -Xmn** |
|  **-XX:SurvivorRatio=3**  |  `S0:S1:Eden=1:1:3`，整个Survivor占年轻代的 2/5  |
|  -XX:MaxTenuringThreshold=15  |  对象年龄到15岁的时候晋升到老年代，**默认是15**  |
|  -XX:TargetSurvivorRatio=90  |  表示MinorGC结束了Survivor区域中占用空间的期望比例。期望未达到时，重新调整TenuringThreshold值，让对象尽早的去old区，**默认是50** |
|    |    |
|  -XX:PermSize=256M  |  持久代初始化大小  |
|  -XX:MaxPermSize=256M  |  持久代最大大小  |
|  -XX:MetaspaceSize=256M  |  JDK8 元数据空间初始大小  |
|  -XX:MaxMetaspaceSize=256M  |  JDK8 元数据空间最大值  |
|    |    |




# GC 设置
|    |    |
| --- | --- |
|  -server  |  x64下只有server模式；新生代默认`Parallel Scavenge`，老年代默认 `Serial Old` |
|    |    |
|  **-XX:+UseConcMarkSweepGC**  |  ParNew + CMS + Serial Old  |
|  **-XX:ConcGCThreads=2**  |  并发收集线程数，如果没有设置则为(ParallelGCThreads + 3)/4  |
|  -XX:CMSFullGCsBeforeCompaction=0  |  运行多少次FullGC以后对内存空间进行压缩、整理, 默认是0 |
|  -XX:+UseCMSCompactAtFullCollection  |  可以对年老代的压缩，默认是打开的  |
|  -XX:CMSInitiatingOccupancyFraction=75  |  设置执行GC的阈值，如75，则占用空间到达75%的时候开始GC  |
|  [JVM实用参数（七）CMS收集器](http://ifeve.com/useful-jvm-flags-part-7-cms-collector/)   |   |
|    |    |
|    |    |
|  -XX:+UseG1GC  |  使用G1GC  |
|  -XX:G1HeapRegionSize=n   |  值是 2 的幂，范围是 1 MB 到 32 MB 之间。目标是根据最小的 Java 堆大小划分出约 2048 个区域  |
|  -XX:MaxGCPauseMillis=200   |  GC可以暂停多长时间。 这只是个建议值，G1GC 暂停的时间会尽可能比这个值短。默认值是 200 毫秒  |
|  -XX:G1HeapWastePercent=10   |  设置您愿意浪费的堆百分比。 垃圾越大， GC 在新区域分配对象时越快，而不是试图将这些空间浪费掉  |
|    |    |
|    |    |
|  **-XX:UseParallelOldGC**   |  Parallel Scavenge + Parallel Old  |
|  -XX:ParallelGCThreads=4  |  并行收集线程数，默认是CPU个数  |
|  -XX:MaxGCPauseMillis=n  |  并行收集最大暂停时间  |
|  -XX:GCTimeRatio=n  |  GC时间占程序时间的百分比,公式为1/(1+n)  |
|  -XX:+UseAdaptiveSizePolicy  |  自动选择年轻代区大小和相应的Survivor区比例，以达到目标系统规定的最低相应时间或者收集频率等;设置该选项之后，就不要再设置 -Xmn、-XX:SurvivorRatio、-XX:MaxTenuringThreshold 等选项了，但是可以设置 -Xms、-Xmx规定一下整个堆的大小  |
|    |    |
|    |    |
|  *-XX:UseParallelGC*   |  Parallel Scavenge + Serial Old  |
|  *-XX:UseParNewGC*  |  ParNew + Serial Old  |
|  -XX:UseSerialGC  |  Serial + Serial Old  |
|    |    |
|    |    |
|  -Xnoclassgc  |  禁用垃圾回收（慎用，会导致内存溢出）  |
|    |    |
|    |    |
|  -XX:+DisableExplicitGC  |  禁用`System.gc()`  |
| -XX:+ExplicitGCInvokesConcurrent	| 当调用`System.gc()`的时候，执行并行gc。默认是不开启的，只有使用`-XX:+UseConcMarkSweepG`C选项的时候才能开启这个选项。|
|-XX:+ExplicitGCInvokesConcurrentAndUnloadsClasses	| 当调用`System.gc()`的时候， 执行并行gc。并在垃圾回收的周期内卸载类。 只有使用`-XX:+UseConcMarkSweepGC`选项的时候才能开启这个选项。|
|    |    |

> [Java GC 总结](https://github.com/kailbin/blog/issues/7)






# 故障排查设置

|    |    |
| --- | --- |
| -**Xloggc:/opt/logs/resin/GC.log**			|  把GC信息输出到文件中，和verbose:gc的内容是一样的。如果这两个命令一起使用的话，Xloggc会覆盖verbose命令  |
| **-XX:+PrintGC**												|  打印GC信息  |
| **-XX:+PrintGCDetails**									|  打印GC详细信息  |
| **-XX:+PrintGCApplicationConcurrentTime** |  打印自从上次GC停顿到现在过去了多少时间  |
| **-XX:+PrintGCApplicationStoppedTime**	|  打印GC一共停顿了多长时间  |
| **-XX:+PrintHeapAtGC**								|  在GC的时候打印堆信息  |
| -XX:+PrintGCDateStamps  |   打印GC 日期  |
| **-XX:+PrintGCTimeStamps**  |   打印GC时间戳  |
| -XX:-PrintTenuringDistribution		|  打印各代信息| 
|     |     |
|     |     |
| **-XX:ErrorFile=/opt/logs/hs_err_pid.log** | 	如果JVM crashed，将错误日志输出到指定文件路径。| 
|     |     |
|     |     |
| **-XX:+HeapDumpOnOutOfMemoryError**	| 在OOM时，输出一个dump.core文件，记录当时的堆内存快照。| 
| -**XX:HeapDumpPath=/opt/logs/java_pid.hprof**	|  堆内存快照的存储文件路径。| 
|     |     |
|     |     |
|  -verbose:class	|  展示出每一个类被加载的信息  |
|  -verbose:gc	   |   展示出每一次垃圾回收事件  |
|  -verbose:jni   |   展示本地方法的使用，以及本地接口的活动  |
|     |     |
| -Xverify:mode | 	设置字节码验证模式。字节码验证可以帮助我们找到一些问题。mode的参数如下：`none`不进行验证,节省启动时间，同时也减少了java提供的保护；默认`remote`验证那些不是被引导类加载器加载的类；`all`验证所有的类| 
|     |     |
|     |     |
| -XX:-PrintTenuringDistribution	|  跟踪每次新生代GC后，survivor区中对象的年龄分布  |
| -XX:-PrintGCTaskTimeStamps		|  为每个独立的gc线程打印时间戳| 
| -XX:-PrintStringDeduplicationStatistics	|  打印字符串去重统计信息| 
| -XX:-TraceClassLoading		|  跟踪类的加载信息,当类加载的时候输入该类，默认关闭| 
| -XX:-TraceClassLoadingPreorder		|  按照引用顺序跟踪类加载。默认关闭| 
| -XX:-TraceClassResolution		|  跟踪常量池，默认关闭| 
| -XX:-TraceClassUnloading		|  跟踪类的卸载信息，默认关闭| 
| -XX:-TraceLoaderConstraints	|  	跟踪类加载器约束的相关信息，默认关闭| 
| -XX:-PrintCommandLineFlags		|  输出JVM设置的选项和值，比如：堆大小、垃圾回收器等。默认这个选项是关闭的。| 
| -XX:-G1PrintHeapRegions	|  	打印G1收集器收集的区域。默认这个选项是关闭的。| 
|     |     |
|     |     |
| -XX:-CITime		|  打印消耗在JIT编译的时间| 
| -XX:-PrintCompilation		|  打印方法被JIT编译时的信息。| 
|     |     |
|     |     |




# 性能相关

|    |    |
| --- | --- |
|  **-XX:InitialCodeCacheSize=50M**  |    |
|  **-XX:ReservedCodeCacheSize=256M**  |  JIT编译代码的最大缓存，和`-Xmaxjitcodesize` 类似  |
| -Xmaxjitcodesize=240m | 指定JIT编译代码的最大缓存，默认的最大缓存是240M | 
|  **-XX:+UseCodeCacheFlushing**  |  缓存被填满时让 JVM 放弃一些编译代码，避免切换到解释模式  |
| -Xmixed  | 	使用混合模式运行代码：解释模式和编译模式| 
|    |    |
| **-XX:+UseCompressedOops**  |  （x64默认开启）减少内存的使用  |
|    |    |
|  -Xint  |  设置jvm以解释模式运行，所有的字节码将被直接执行，而不会编译成本地码  |
|    |    |
|    |    |

# 系统属性设置
|    |    |
| --- | --- |
|  -D  |  设置系统属性  |
|    |    |
|  **-Dsun.net.inetaddr.ttl=60**  |  dns解析**成功**的缓存超时的设置 (0:禁止缓存，**默认-1:永远有效**)  |
|  **-Dsun.net.inetaddr.negative.ttl=10**  | dns解析**失败**的缓存超时的设置 (0:禁止缓存，-1:永远有效，**默认10**)  |
|  *-Dnetworkaddress.cache.ttl=60*  | (非容器环境) dns解析**成功**的缓存超时的设置 (0:禁止缓存，-1:永远有效)  |
|  *-Dnetworkaddress.cache.negative.ttl=10*  | (非容器环境) dns解析**失败**的缓存超时的设置 (0:禁止缓存，-1:永远有效)  |
| -Dsun.net.client.defaultConnectTimeout=1000   |  链接超时时间  |
| -Dsun.net.client.defaultReadTimeout=4000   |  读取超时时间  |
|    |    |
|    |    |
|    |    |
|    |    |
|  -Dcom.sun.management.jmxremote=true               |  开启 JMX 监控   |
|  -Dcom.sun.management.jmxremote.port=<port>      |     监控端口    |
|  -Dcom.sun.management.jmxremote.ssl=false                  |  非 ssl  |
|  -Dcom.sun.management.jmxremote.authenticate=false  |  关掉认证  |
|    |    |
|    |    |
|  -Duser.timezone=Asia/Shanghai  |  设置时区  |
|  -Dfile.encoding=UTF-8  |    |
|    |    |
|    |    |
|    |    |
|    |    |
|    |    |
|  -Djava.awt.headless=true  |  避免在winodws开发环境下 图片显示的好好的可是在linux/unix下却显示不出来的问题 |
|    |    |
|    |    |

> [Networking Properties](https://docs.oracle.com/javase/8/docs/technotes/guides/net/properties.html)
> 

# 其他
|    |    |
| --- | --- |
|  -javaagent  |  链接class文件对字节码进行修改  |
|  -Xrunjdwp:transport=dt_socket,address=8000,server=y,suspend=n  | 开启JPDA远程调试。 `suspend`:y表示启动的JVM会暂停等待，直到调试器连接上，n表示不暂停等待  |
|    |    |
|    |    |


> [ JVM命令参数大全](http://blog.csdn.net/zero__007/article/details/52848040)


# 如何查看JVM默认参数是什么？

## 方法1 -XX:+PrintFlagsFinal

”=”表示参数的默认值，而”:=” 表明了参数被用户或者JVM赋值了。

```
$ java -XX:+PrintFlagsFinal  -version | grep ":="
......
     bool UseCompressedClassPointers               := true                                {lp64_product}
     bool UseCompressedOops                        := true                                {lp64_product}
     bool UseParallelGC                            := true                                {product}
......
```

## 方法2 -XX:+PrintCommandLineFlags
```
$ java -XX:+PrintCommandLineFlags -version
-XX:InitialHeapSize=268435456 -XX:MaxHeapSize=4294967296 -XX:+PrintCommandLineFlags -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseParallelGC 
java version "1.8.0_131"
Java(TM) SE Runtime Environment (build 1.8.0_131-b11)
Java HotSpot(TM) 64-Bit Server VM (build 25.131-b11, mixed mode)
```


# Read More
> [Java 8: 从永久代（PermGen）到元空间（Metaspace）](http://blog.csdn.net/zhushuai1221/article/details/52122880)  
> [Java HotSpot VM Command-Line Options](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/clopts001.html)  
> [HotSpot VM Command Line Options](http://www.oracle.com/technetwork/java/javase/tech/index-jsp-136373.html)  
> [Java SE Technologies](http://www.oracle.com/technetwork/java/javase/tech/index.html)  
