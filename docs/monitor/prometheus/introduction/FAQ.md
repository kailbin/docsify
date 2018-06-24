
# FREQUENTLY ASKED QUESTIONS


[TOC]

## General


### 什么是 Prometheus?

Prometheus是一个开源监控系统和警报工具包，拥有一个活跃的生态系统。 请参阅[概述](https://prometheus.io/docs/introduction/overview/)。



### Prometheus 如何与其他监测系统进行比较?

请查看 [与其它选择比较](prometheus/introduction/vs.md) 页面.



### Prometheus 有什么依赖?

Prometheus服务器独立运行，没有外部依赖。



### Prometheus 有高可用方案吗?

是的，在两台或多台独立的机器上运行相同的Prometheus服务器。相同的警报将由[Alertmanager](https://github.com/prometheus/alertmanager)进行重复数据删除。

为了提高Alertmanager的可用性，您可以在Mesh群集中运行多个实例，并配置Prometheus服务器向每个服务器发送通知。

> [Can Prometheus be made highly available?](https://prometheus.io/docs/introduction/faq/#can-prometheus-be-made-highly-available)




### 我被告知 Prometheus “没有伸缩性(doesn't scale)”？

There are in fact various ways to scale and federate Prometheus. Read Scaling and Federating Prometheus on the Robust Perception blog to get started.



### Prometheus 用什么语言写的?

大多数Prometheus组件都是用Go编写的。有些也用Java，Python和Ruby编写。




### Prometheus功能，存储格式和API的稳定性如何？?

All repositories in the Prometheus GitHub organization that have reached version 1.0.0 broadly follow semantic versioning. Breaking changes are indicated by increments of the major version. Exceptions are possible for experimental components, which are clearly marked as such in announcements.

Even repositories that have not yet reached version 1.0.0 are, in general, quite stable. We aim for a proper release process and an eventual 1.0.0 release for each repository. In any case, breaking changes will be pointed out in release notes (marked by [CHANGE]) or communicated clearly for components that do not have formal releases yet.




### 为什么是 pull 而不是 push?

通过HTTP pull 提供了许多好处：

You can run your monitoring on your laptop when developing changes.
开发变更时，您可以在笔记本电脑上运行您的监控。
You can more easily tell if a target is down.
你可以更容易地判断一个目标是否失败。
You can manually go to a target and inspect its health with a web browser.
您可以手动转到目标并使用Web浏览器检查其健康状况。

Overall, we believe that pulling is slightly better than pushing, but it should not be considered a major point when considering a monitoring system.
总体而言，我们认为拉动略好于推动，但在考虑监测系统时不应该被视为重点。

For cases where you must push, we offer the Pushgateway.
对于你必须推的情况，我们提供Pushgateway。




### How to feed logs into Prometheus?

Short answer: Don't! Use something like the ELK stack instead.

Longer answer: Prometheus is a system to collect and process metrics, not an event logging system. The Raintank blog post Logs and Metrics and Graphs, Oh My! provides more details about the differences between logs and metrics.

If you want to extract Prometheus metrics from application logs, Google's mtail might be helpful.

### Prometheus 是谁写的?

Prometheus was initially started privately by Matt T. Proud and Julius Volz. The majority of its initial development was sponsored by SoundCloud.

It's now maintained and extended by a wide range of companies and individuals.

### Prometheus 的发布 license 是什么?

普罗米修斯是在**Apache 2.0许可**下发布的。

### Prometheus 的复数单词是什么?

经过广泛的研究，已经确定`Prometheus`的正确复数是`Prometheis`。

### 如何重新加载 Prometheus 的配置?

Yes, sending SIGHUP to the Prometheus process or an HTTP POST request to the /-/reload endpoint will reload and apply the configuration file. The various components attempt to handle failing changes gracefully.

### Can I send alerts?

Yes, with the Alertmanager.

Currently, the following external systems are supported:

- Email
- Generic Webhooks
- HipChat
- OpsGenie
- PagerDuty
- Pushover
- Slack



### Can I create dashboards?

是的，我们推荐在生产环境使用 Grafana。还有控制台模板（`Console templates`）。


### 我可以改变时区吗? 为什么都是 UTC?

To avoid any kind of timezone confusion, especially when the so-called daylight saving time is involved, we decided to exclusively use Unix time internally and UTC for display purposes in all components of Prometheus. 
为了避免任何类型的时区混淆，特别是当涉及到所谓的夏令时时，我们决定在内部使用Unix时间和UTC作为普罗米修斯所有组件的显示目的。
A carefully done timezone selection could be introduced into the UI. Contributions are welcome. 
可以在UI中引入精心完成的时区选择。, 贡献是受欢迎的。
See issue #500 for the current state of this effort.
请参阅问题＃500了解此项工作的当前状态。




## Instrumentation


### Which languages have instrumentation libraries?

There are a number of client libraries for instrumenting your services with Prometheus metrics. See the client libraries documentation for details.

If you are interested in contributing a client library for a new language, see the exposition formats.

### Can I monitor machines?

Yes, the Node Exporter exposes an extensive set of machine-level metrics on Linux and other Unix systems such as CPU usage, memory, disk utilization, filesystem fullness, and network bandwidth.

### Can I monitor network devices?

Yes, the SNMP Exporter allows monitoring of devices that support SNMP.

### Can I monitor batch jobs?

Yes, using the Pushgateway. See also the best practices for monitoring batch jobs.

### What applications can Prometheus monitor out of the box?

See the list of exporters and integrations.

### Can I monitor JVM applications via JMX?

Yes, for applications that you cannot instrument directly with the Java client, you can use the JMX Exporter either standalone or as a Java Agent.

### What is the performance impact of instrumentation?

Performance across client libraries and languages may vary. For Java, benchmarks indicate that incrementing a counter/gauge with the Java client will take 12-17ns, depending on contention. This is negligible for all but the most latency-critical code.




## Troubleshooting


### My Prometheus server takes a long time to start up and spams the log with copious information about crash recovery.

You are suffering from an unclean shutdown. Prometheus has to shut down cleanly after a SIGTERM, which might take a while for heavily used servers. If the server crashes or is killed hard (e.g. OOM kill by the kernel or your runlevel system got impatient while waiting for Prometheus to shutdown), a crash recovery has to be performed, which should take less than a minute under normal circumstances, but can take quite long under certain circumstances. See crash recovery for details.



### My Prometheus server runs out of memory.

See the section about memory usage to configure Prometheus for the amount of memory you have available.



### My Prometheus server reports to be in “rushed mode” or that “storage needs throttling”.

Your storage is under heavy load. Read the section about configuring the local storage to find out how you can tweak settings for better performance.



## Implementation


### Why are all sample values 64-bit floats? I want integers.

We restrained ourselves to 64-bit floats to simplify the design. The IEEE 754 double-precision binary floating-point format supports integer precision for values up to 253. Supporting native 64 bit integers would (only) help if you need integer precision above 253 but below 263. In principle, support for different sample value types (including some kind of big integer, supporting even more than 64 bit) could be implemented, but it is not a priority right now. A counter, even if incremented one million times per second, will only run into precision issues after over 285 years.

### Why don't the Prometheus server components support TLS or authentication? Can I add those?

While TLS and authentication are frequently requested features, we have intentionally not implemented them in any of Prometheus's server-side components. There are so many different options and parameters for both (10+ options for TLS alone) that we have decided to focus on building the best monitoring system possible rather than supporting fully generic TLS and authentication solutions in every server component.

### If you need TLS or authentication, we recommend putting a reverse proxy in front of Prometheus. See, for example Adding Basic Auth to Prometheus with Nginx.

This applies only to inbound connections. Prometheus does support scraping TLS- and auth-enabled targets, and other Prometheus components that create outbound connections have similar support.




# 修改配置文件如何 reload
```
kill -HUP <pid>
```
或
```
curl -X POST http://localhost:9090/-/reload
```
> [How to reload Prometheus's configuration? ](https://github.com/prometheus/prometheus/issues/1572)