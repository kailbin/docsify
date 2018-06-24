
# Dropwizard Metrics

## 简介


`Metrics` 能够帮助我们从各个角度度量已存在的java应用的成熟框架，简便地以jar包的方式集成进您的系统，
可以以http、ganglia、graphite、log4j等方式提供全栈式的监控视野。

在 Spring Boot 的领域内使用 Actuator 是最优的选择。
然而web应用五花八门，如一些应用直接使用Jetty启动的web应用、一些已存在的非springboot的应用、其他一些如已服务很久的基于netty的java应用, Spring Boot  Actuator 就不那么适用了。


## 度量 Java 应用的角度

|  度量术语  |  描述  |
|----------|--------|
|  Meter  |  随着时间推移，时间发生的频率；如 1、5、15的分钟的平局负载等  | 
|  Gauges  |  某个瞬时值；如磁盘空间等  | 
|  Counter  |  某个递增或者递减的值；如网站访问量等  | 
|  Histogram  |  直方图：以流的形式分析数据，除了最大、最小、平均以外，还有中位数、75th、90th、95th、98th、99th、99.9th 百分位（如果你的考试成绩在 90th百分位，说明大概有90%的人比你差）  | 
|  Timer  |  调用某段代码的频率、时间、以及所花时间的时长分布区域  | 
|  HealthChecks  |  集中式的检查应用的健康状态；如 database、mq、container 的  | 
|  Reporting  |  度量值可以通过 HTTP、slf4j、jmx、csv、graphite 等报告出来  | 



> 本文摘录自 [Dropwizard 之 application metrics](https://www.jianshu.com/p/070f615dfb57)
