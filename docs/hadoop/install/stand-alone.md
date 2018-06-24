
# 独立模式

## 推荐的 Java 版本 

- Hadoop 2.7+ 要求 Java 7 
- 早些的 2.6 及其以下版本 支持 Java 6 。

> [HadoopJavaVersions](https://wiki.apache.org/hadoop/HadoopJavaVersions)



## 安装

``` bash
# 下载 后 解压
wget http://mirrors.hust.edu.cn/apache/hadoop/common/hadoop-2.7.5/hadoop-2.7.5.tar.gz

# 配置 Java 环境变量
vim hadoop-2.7.5/etc/hadoop/hadoop-env.sh
# 修改 JAVA_HOME
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_131.jdk/Contents/Home/

# 使用 hadoop 命令
# 
./hadoop-2.7.5/bin/hadoop

```

独立（本地）模式 无需运行任何守护进程，所有程序都在同一个 JVM 上运行，该模式下测试和调试 MapReduce 程序很方便，比较适合开发阶段；
`./bin/hadoop fs` 操作的都是本地文件，而非 HDFS

``` bash
$ cd hadoop-2.7.5
$ mkdir input
$ cp etc/hadoop/*.xml input
$ ./bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.5.jar grep input output 'dfs[a-z.]+'
$ cat output/*
```


## Read More

- [Standalone Operation](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/SingleCluster.html#Standalone_Operation)
