
# COMPARISON TO ALTERNATIVES



[TOC]




## Prometheus vs. Graphite

### Scope

Graphite专注于被动时间**序列数据库**，具有**查询语言**和**绘图功能**。任何其他问题都由外部组件解决。

Prometheus 是一个完整的**监控和趋势(走向)系统**，包括内置的**主动收集**、**存储**、**查询**、**绘图**和**警报** 的基于时间序列数据的。它具有大局观（哪些端点应该存在，什么时间序列模式意味着麻烦等等），并积极地找出错误。

### Data model

Graphite存储指定时间序列的数字样本，就像普Prometheus一样。
但是，Prometheus 的元数据模型更加丰富：
Graphite指标名称由点分离组件组成，Prometheus将维度显式编码为附加到度量标准名称的键值对（称为标签）。
这允许通过这些标签和查询语言进行简单的过滤，分组和匹配。

此外，特别是当Graphite与StatsD结合使用时，通常只将聚合的数据存储在所有受监视的实例上，而不是将实例保留为一个维度，深入到单个有问题的实例中。

例如，将响应代码为500的API服务器的HTTP请求数量和POST跟踪端点的方法的HTTP请求数量通常在 `Graphite/StatsD` 中编码如下：
```
stats.api-server.tracks.post.500 -> 93
```

在Prometheus ，相同的数据可以这样编码（假设三个api服务器实例）：
```
api_server_http_requests_total{method="POST",handler="/tracks",status="500",instance="<sample1>"} -> 34
api_server_http_requests_total{method="POST",handler="/tracks",status="500",instance="<sample2>"} -> 28
api_server_http_requests_total{method="POST",handler="/tracks",status="500",instance="<sample3>"} -> 31
```

### Storage

Graphite以Whisper格式存储本地磁盘上的时间序列数据，这是一种RRD样式的数据库，需要样本定期到达。
每个时间序列都存储在一个单独的文件中，新的样本在一段时间后覆盖旧的样本。

Prometheus也会在每个时间序列中创建一个本地文件，而且允许在发生刮擦或规则评估时以任意间隔存储样本
由于新样本是简单的附加，旧的数据可能会被长时间保持。
Prometheus 也适用于许多短暂的，频繁变化的时间序列。

### Summary

Prometheus 提供了​​更丰富的数据模型和查询语言，而且更容易运行和集成到您的环境（支持HA双写，不支持集群）。
如果您想要一个可以长期保存历史数据的集群解决方案，那么Graphite可能是更好的选择。









## Prometheus vs. InfluxDB
InfluxDB是一个开源的时间序列数据库，商业版支持集群扩展（开源版不支持）。
InfluxDB项目在Prometheus开发后近一年后才发布，所以当时我们无法将其视为一种替代方案。
尽管如此，Prometheus和InfluxDB之间还是有很大的不同，两种系统各自针对不同的用例。

### Scope

为了公平的比较，我们还必须`Kapacitor`和`InfluxDB`一块考虑，因为它们组合在一起处理与`Prometheus`和`Alertmanager`才具有相同功能。


InfluxDB本身也适用于与Graphite相同的场景。
另外InfluxDB提供连续的查询，相当于Prometheus的 记录规则（recording rules）。

Kapacitor的范围是Prometheus记录规则（recording rules）、警报规则、Alertmanager的通知功能 的组合。
Prometheus 提供了​​一个更强大的图形和警报查询语言。
Prometheus Alertmanager还提供分组，重复数据消除和沉默功能。

### Data model / storage

像Prometheus 一样，InfluxDB数据模型也有键值对作为标签。
另外，InfluxDB还有一个叫做“字段”的标签，它的使用更为有限。
InfluxDB支持最高达纳秒级别的时间戳，以及float64，int64，bool和string数据类型。
相比之下，Prometheus只支持float64数据类型，对字符串和毫秒级别的时间戳 支持有限。

InfluxDB使用日志结构合并树（log-structured merge tree）的变体来存储，并提前写入日志，并按时间分割。
作为事件日志记录更适合，Prometheus的每个时间序列仅附加文件。

> Logs and Metrics and Graphs, Oh My! describes the differences between event logging and metrics recording.
日志、度量、图形，我介绍了事件记录和度量记录之间的区别。


### Architecture

Prometheus 服务器彼此独立运行，只依靠其本地存储来实现其核心功能：抓取，规则处理和警报。与InfluxDB的开源版本类似。


商业InfluxDB产品在设计上是一个分布式存储集群，存储和查询一次被许多节点处理。

这意味着商业InfluxDB将更容易横向扩展，但这也意味着您必须从一开始就管理分布式存储系统的复杂性。
Prometheus运行起来会比较简单，但是在某些时候，您需要明确地沿着可扩展性边界（如产品，服务，数据中心或类似方面）对服务器进行分片。
独立的服务器（可以并行运行）也可以提供更好的可靠性和故障隔离。


Kapacitor目前没有内建的 规则、警报或通知 的分布式/冗余选项。
相比之下，Prometheus和Alertmanager通过运行Prometheus的冗余副本并使用Alertmanager的高可用性模式提供了冗余选项。
另外，Kapacitor可以通过用户的手动分片进行伸缩，类似于Prometheus 。

### Summary

这两个系统之间有许多相似之处。,两者都有labels（在InfluxDB中称为tags）来有效地支持多维指标。
两者使用基本相同的数据压缩算法，都有广泛的继承，包括彼此之间的集成。
两者都有钩子，允许您进一步扩展它们，例如分析统计工具中的数据或执行自动操作。

#### InfluxDB 的优势:

- 如果你正在做事件记录。
- 商业选项为InfluxDB提供了集群，对于长期数据存储来说也更好。
- 节点之间复制数据的最终一致视图。


#### Prometheus 的优势:

- 如果你主要做指标
- 更强大的查询语言，警报和通知功能。
- 更高的可用性和正常运行时间的图形和警报。

InfluxDB由一个商业公司维护，遵循开放核心模式（open-core），提供诸如闭源集群，托管和支持等高级功能。
Prometheus 是一个完全开源和独立的项目，由许多公司和个人维护，其中一些还提供商业服务和支持。









## Prometheus vs. OpenTSDB

OpenTSDB是基于Hadoop和HBase的分布式时间序列数据库。
### Scope

与Graphite相同的范差异适用于此。

### Data model

OpenTSDB的数据模型与普罗米修斯的数据模型几乎相同：时间序列由一组任意的键值对（OpenTSDB是tags，普罗米修斯是labels）标识。
指标的所有数据都一起存储，限制了度量的基数。
虽然有一些细微的差别：Prometheus允许标签值中的任意字符，而OpenTSDB更具有限制性。
OpenTSDB也缺乏完整的查询语言，只允许通过API进行简单的聚合和数学运算。

### Storage

OpenTSDB的存储是在Hadoop和HBase之上实现的。
这意味着OpenTSDB水平扩展很容易，但是您必须从一开始就接受运行Hadoop / HBase集群的整体复杂性。

普罗米修斯最初运行起来会比较简单，但是一旦超过单个节点的容量，将需要明确的分片。

### Summary

普罗米修斯提供了​​更丰富的查询语言，可以处理更高的基数度量标准，并构成完整监控系统的一部分。
如果您已经在运行Hadoop并重视长期存储，那么OpenTSDB是一个不错的选择。









## Prometheus vs. Nagios
Nagios是一个起源于20世纪90年代的NetSaint监控系统。


### Scope

Nagios主要是基于脚本的退出代码(exit codes)进行报警。这些被称为`checks`。 有单独的警报沉默，但没有分组，路由或重复数据删除。


Nagios有各种各样的插件。,例如，只有几千字节的perfData插件传输数据到时间序列数据库（如Graphite）或使用`NRPE`在远程计算机上运行检查。

### Data model

Nagios是基于主机的。每个主机可以有一个或多个服务，每个服务可以执行一个检查。没有标签或查询语言的概念。

### Storage

Nagios本身没有存储，超出当前的检查状态。 有插件可以存储数据和可视化。

### Architecture

Nagios服务器是独立的。 所有配置的检查都是通过文件。


### Summary

Nagios适用于小型和/或静态系统的基本监控。

如果你想做白盒监测，或者有一个动态的或基于云的环境，那么普罗米修斯是一个不错的选择。






## Prometheus vs. Sensu

Sensu从广义上讲是更现代化的Nagios。


### Scope

与Nagios相同的一般范围差异适一样。

主要区别在于Sensu客户端自行注册，并可以确定从中央或本地配置运行的检查。 Sensu对perfData的数量没有限制。

还有一个客户端套接字允许任意检查结果被推入Sensu。


### Data model

与 Nagios 一样

### Storage

Sensu在Redis存储称为stashes。,这些主要用于存储沉默。它还存储所有已经注册的客户端。

### Architecture

Sensu有一些组件。它使用RabbitMQ作为传输，Redis作为当前状态，并使用单独的服务器进行处理。
RabbitMQ和Redis都可以集群。可以运行服务器的多个副本进行扩展和冗余。


### Summary

如果您有一个现有的Nagios设置，您希望按原样进行扩展，或者想要利用Sensu的注册功能，那么Sensu就是一个不错的选择。

如果你想做白盒监控，或者有一个非常动态或基于云的环境，那么普罗米修斯是一个不错的选择。