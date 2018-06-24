
# GLOSSARY

## Alert(告警、警报)

警报是 Prometheus 主动触发警报规则的结果。 警报从Prometheus发送到 Alertmanager。


## Alertmanager

[Alertmanager](prometheus/alerting/overview.md)接收警报，将它们**分组聚合**、**删除重复**、**应用沉默**、**节流**，然后发送通知到**电子邮件**、**Pagerduty**、**Slack**等。


## Bridge

Bridge 是一个从客户端库中提取样本并将其展示给 非Prometheus 监控系统的组件。例如：Python、Go和Java客户端可以将度量标准导出到 Graphite。



## Client library

A client library is a library in some language (e.g. Go, Java, Python, Ruby) that makes it easy to directly instrument your code, write custom collectors to pull metrics from other systems and expose the metrics to Prometheus.



## Collector

`Collector` 是 `Exporter` 的一部分，代表一组指标。 如果它是`Direct Instrumentation`的一部分，则可能是单个指标，如果是从其他系统提取指标，则可能是多个指标。


## Direct instrumentation

`Direct instrumentation`是作为程序源代码的一部分内联添加的。


## Endpoint

可以进行搜罗的主表来源，通常对应于一个单独的进程。



## Exporter

Exporter 通常将**非Prometheus格式的指标**转换**为Prometheus支持的格式**来公开Prometheus指标。



## Instance

Instance 是唯一标识作业中的目标的标签。



## Job

具有相同目的的一组目标（例如，监视一组为了可伸缩性或可靠性而复制的类似进程）被称为 Job。



## Notification

Notification 代表一组警报，由Alertmanager发送给电子邮件、Pagerduty、Slack等。



## Promdash

Promdash是Prometheus的本地仪表板生成器。它已被弃用，取而代之的是Grafana。



## Prometheus

Prometheus 通常指 Prometheus系统的核心可执行文件。 它也可能是指整个 Prometheus监测系统。



## PromQL

PromQL是Prometheus查询语言。它支持丰富的操作，包括**聚合**、**切片和切割**、**预测**和**连接**。



## Pushgateway

Pushgateway 持久化最近从批处理作业中推过来的指标。这使得Prometheus在终止之后也可以收集他们的指标。



## Remote Read

远程读取是一个Prometheus功能，允许从其他系统（如长期存储）作为查询的一部分透明读取时间序列。



## Remote Read Adapter

并非所有系统都直接支持远程读取。 Prometheus和另一个系统之间有一个远程读取适配器，用于转换它们之间的时间序列请求和响应。



## Remote Read Endpoint

远程读取端点是普罗米修斯进行远程读取时所说的内容。



## Remote Write

远程写入是Prometheus功能，可以将摄入的样本随时发送到其他系统，如长期存储。


## Remote Write Adapter

Not all systems directly support remote write. A remote write adapter sits between Prometheus and another system, converting the samples in the remote write into a format the other system can understand.


## Remote Write Endpoint

A remote write endpoint is what Prometheus talks to when doing a remote write.


## Silence

A silence in the Alertmanager prevents alerts, with labels matching the silence, from being included in notifications.

## Target

A target is the definition of an object to scrape. For example, what labels to apply, any authentication required to connect, or other information that defines how the scrape will occur.