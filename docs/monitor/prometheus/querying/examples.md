
# QUERY EXAMPLES

## 简单的时间序列查询

返回指标`http_requests_total`的所有时间序列 :
```
http_requests_total
```


使用 `job` 和 `handler` 标签 筛选指标`http_requests_total`的所有时间序列:
```
http_requests_total{job="apiserver", handler="/api/comments"}
```


Return a whole range of time (in this case 5 minutes) for the same vector, making it a range vector:
返回一个范围时间（在这种情况下为5分钟）内的指标情况
> 返回的数据条数取决于收集数据的周期频率(`scrape_interval`)，如果是 5s 收集一次，则返回数据为 5*60/5=60 条
```
http_requests_total{job="apiserver", handler="/api/comments"}[5m]
```
**请注意**，范围矢量的表达式不能直接绘制，但可以在表达式浏览器的表格（"Console"）视图中查看。


Using regular expressions, you could select time series only for jobs whose name match a certain pattern, in this case, all jobs that end with server. 
使用正则表达式，您可以选择时间序列只适用于名称匹配特定模式的作业；下面这个查询语句 返回以 server结尾的 job。
**请注意，这是一个子字符串匹配，而不是一个完整的字符串匹配**：
```
http_requests_total{job=~".*server"}
```
Prometheus的所有正则表达式都使用RE2语法



要选择除`4xx`之外的所有`HTTP`状态码，可以运行
```
http_requests_total{status!~"4.."}
```


## 使用 函数、操作符 等

使用`http_requests_total`指标名称返回 过去5分钟所测量的时间序列的**每秒速率**，如：
```
rate(http_requests_total[5m])
```


假设`http_requests_total`时间序列都有标签 `job`和 `instance`，我们可能想要统计所有 instance 的速率，这时我们就可以得到较少的时间序列，但仍然保留 job 维度：
```
sum(rate(http_requests_total[5m])) by (job)
```


如果有**相同维度标签**的两个**不同度量标准**，我们可以对它们应用二元运算符，并且具有相同标签集合的两侧元素将被匹配并传播到输出。例如，这个表达式在每个实例返回未使用的内存（单位MiB）：
```
(instance_memory_limit_bytes - instance_memory_usage_bytes) / 1024 / 1024
```


同样的，应用的统计，可以这样写
```
sum(
  instance_memory_limit_bytes - instance_memory_usage_bytes
) by (app, proc) / 1024 / 1024
```


如果相同的集群调度程序公开了每个实例的如下所示的CPU使用率度量：
```
instance_cpu_time_ns{app="lion", proc="web", rev="34d0f99", env="prod", job="cluster-manager"}
instance_cpu_time_ns{app="elephant", proc="worker", rev="34d0f99", env="prod", job="cluster-manager"}
instance_cpu_time_ns{app="turtle", proc="api", rev="4d3a513", env="prod", job="cluster-manager"}
instance_cpu_time_ns{app="fox", proc="widget", rev="4d3a513", env="prod", job="cluster-manager"}
...
```
我们可以像这样获得按应用程序（app）和进程类型（proc）分组的**前3名CPU用户**
```
topk(3, sum(rate(instance_cpu_time_ns[5m])) by (app, proc))
```

假设此度量标准包含每个正在运行的实例的一个时间序列，则可以**计算每个应用程序正在运行的实例的数量**，
```
count(instance_cpu_time_ns) by (app)
```