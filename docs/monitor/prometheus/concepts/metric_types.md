
# METRIC TYPES

Prometheus客户端库提供了四种核心度量类型。
目前这些功能仅在客户端库（以启用适合特定类型使用的API）以及有线协议中有所不同。
Prometheus服务器尚未使用类型信息，它将所有数据变为无类型的时间序列。这在将来可能会改变。


## Counter

计数器是一个累计度量，它代表了一个只能上升的数值。
计数器通常用于统计请求的数量，完成的任务，发生的错误等。
不应该使用计数器来显示可以减少的指标，例如： 当前正在运行的进程数量, 这个用例应该使用 gauges。

Counter 的客户端库使用情况
- Go
- Java
- Python
- Ruby


## Gauge

Gauge是一个度量标准，代表一个可以任意加减的单个数值。

Gauges 通常用于测量值，如温度或当前的内存使用情况，但也可以上下“计数”，如正在运行的进程的数量。

Gauge 的客户端库使用情况

- Go
- Java
- Python
- Ruby


## Histogram

直方图对观察结果进行采样（通常是请求持续时间或响应大小），并将其计入可配置的桶中。它也提供了所有观测值的总和。

A histogram with a base metric name of <basename> exposes multiple time series during a scrape:
基本度量标准名称为<basename>的直方图在刮取期间公开多个时间序列：

- cumulative counters for the observation buckets, exposed as <basename>_bucket{le="<upper inclusive bound>"}
- the total sum of all observed values, exposed as <basename>_sum
- the count of events that have been observed, exposed as <basename>_count (identical to <basename>_bucket{le="+Inf"} above)

Use the histogram_quantile() function to calculate quantiles from histograms or even aggregations of histograms. 
A histogram is also suitable to calculate an Apdex score. 
When operating on buckets, remember that the histogram is cumulative. 
See histograms and summaries for details of histogram usage and differences to summaries.

使用histogram_quantile（）函数根据直方图或甚至直方图的聚合来计算分位数。
直方图也适用于计算Apdex分数。
在桶上操作时，请记住直方图是累积的。
有关直方图使用情况和摘要差异的详细信息，请参阅直方图和摘要。


Histogram 的客户端库使用情况
- Go
- Java
- Python
- Ruby


## Summary

Similar to a histogram, a summary samples observations (usually things like request durations and response sizes). While it also provides a total count of observations and a sum of all observed values, it calculates configurable quantiles over a sliding time window.

A summary with a base metric name of <basename> exposes multiple time series during a scrape:

streaming φ-quantiles (0 ≤ φ ≤ 1) of observed events, exposed as <basename>{quantile="<φ>"}
the total sum of all observed values, exposed as <basename>_sum
the count of events that have been observed, exposed as <basename>_count
See histograms and summaries for detailed explanations of φ-quantiles, summary usage, and differences to histograms.

Client library usage documentation for summaries:

Go
Java
Python
Ruby