
# 伪分布式模式

## MAC ssh 服务

系统偏好设置 -> 共享 -> 勾选 `远程登陆`。

## ssh 免密码登陆
``` bash
$ ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
$ cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
$ chmod 0600 ~/.ssh/authorized_keys
```
测试登陆是否需要密码
``` bash
$ ssh localhost
```


## 配置

### `etc/hadoop/core-site.xml`

``` xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```


### `etc/hadoop/hdfs-site.xml`

``` xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>
```

## 启动 

1. 格式化文件系统:

``` bash
$ bin/hdfs namenode -format
```
  
  
2. 启动NameNode守护进程和DataNode守护进程:
``` bash
$ sbin/start-dfs.sh
```
日志会写在 `$HADOOP_LOG_DIR` 路径下 (默认是 `$HADOOP_HOME/logs`).


3. 浏览NameNode的Web界面:

- NameNode information - http://localhost:50070/
- DataNode information - http://localhost:50075/
- SecondaryNamenode information - http://localhost:50090/



4. 制作执行MapReduce作业所需的HDFS目录:

``` bash
$ bin/hdfs dfs -mkdir /user
$ bin/hdfs dfs -mkdir /user/<username>
```


5. 将 input 文件夹 复制到分布式文件系统中:
``` bash
$ bin/hdfs dfs -mkdir input
$ bin/hdfs dfs -put etc/hadoop/*.xml input
```


6. 运行提供的例子：
``` bash
$ bin/hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.5.jar grep input output 'dfs[a-z.]+'
```


7. 检查 output 文件: 把 output 文件夹 从分布式文件系统中 拷贝到 本地文件系统:
``` bash
$ bin/hdfs dfs -get output output
$ cat output/*
```

或者，直接在分布式文件系统中查看:
``` bash
$ bin/hdfs dfs -cat output/*
```


8. 当你完成后，停止守护进程:
``` bash
$ sbin/stop-dfs.sh
```




## Read More
- [Pseudo-Distributed Operation](http://hadoop.apache.org/docs/current/hadoop-project-dist/hadoop-common/SingleCluster.html#Pseudo-Distributed_Operation)
- [Hadoop 2.x常用端口及查看方法](http://www.cnblogs.com/jancco/p/4447756.html)
- [Pdsh使用方法](http://kumu-linux.github.io/blog/2013/06/19/pdsh/)