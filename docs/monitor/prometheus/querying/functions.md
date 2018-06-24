# 函数（functions）


[TOC]


一些函数具有默认参数，例如, `year(v=vector(time()) instant-vector)`。
它的意思是 参数`v`是一个瞬时矢量，如果没有提供，它将默认为表达式`vector(time())`的值。

## `abs()`
`abs(v instant-vector)` 将所有输入矢量的样本值 转换为其绝对值。

## `absent()`
`absent(v instant-vector) ` 判断是否缺席，缺席为1，否者为空

这对于在给定指标名称和标签组合不存在时间序列时，**进行警报很有用**。

```
absent(nonexistent{job="myjob"})
# => {job="myjob"}

absent(nonexistent{job="myjob",instance=~".*"})
# => {job="myjob"}

absent(sum(nonexistent{job="myjob"}))
# => {}
```

In the second example, absent() tries to be smart about deriving labels of the 1-element output vector from the input vector.
在第二个例子中，`absent（）`试图从输入矢量中导出元素输出矢量的标签。



## ceil()
ceil(v instant-vector) rounds the sample values of all elements in v up to the nearest integer.




## changes()
For each input time series, changes(v range-vector) returns the number of times its value has changed within the provided time range as an instant vector.



## `clamp_max()`
`clamp_max(v instant-vector, max scalar)` 将`v`中所有元素的样本值 进行 max的上限限制



## `clamp_min()`
`clamp_min(v instant-vector, min scalar)` 将`v`中所有元素的样本值 进行 min 的下限限制



## `day_of_month()`
`day_of_month(v=vector(time()) instant-vector)` 以UTC为单位返回每个给定时间的月份的一天,返回的值是从1到31。




## day_of_week()
`day_of_week(v=vector(time()) instant-vector)` returns the day of the week for each of the given times in UTC. Returned values are from 0 to 6, where 0 means Sunday etc.




## days_in_month()
`days_in_month(v=vector(time()) instant-vector)` returns number of days in the month for each of the given times in UTC. Returned values are from 28 to 31.





## `delta()`
`delta(v range-vector)`计算范围矢量`v`中每个时间系列元素的**第一个值和最后一个值之间的差值**，返回**给定增量和等效标签的瞬时矢量**。
`delta`是按照范围矢量选择器中指定的全部时间范围计算出来的，所以即使计数器只增加了整数增量，也可能得到一个非整数结果。

下面的示例表达式返回**现在和2小时之前的CPU温度差异**：
```
delta(cpu_temp_celsius{host="zeus"}[2h])
```
`delta`只能与`gauges`一块使用。





## `deriv()`
`deriv(v range-vector)` 使用简单的线性回归来计算范围矢量`v`中的时间序列的每秒导数。

`deriv`只能与`gauges`一块使用。




## exp()
exp(v instant-vector) calculates the exponential function for all elements in v. Special cases are:

Exp(+Inf) = +Inf
Exp(NaN) = NaN




## floor()
floor(v instant-vector) rounds the sample values of all elements in v down to the nearest integer.





## `histogram_quantile()` ???
`histogram_quantile(φ float, b instant-vector)` 从[直方图]()的桶`b`计算φ-分位数（0≤φ≤1）。
(关于`φ-分位数`的详细解释和一般直方图度量类型的用法，请[参阅直方图和摘要]()。)
`b`样本是每个桶中观察值的计数。
每个样本必须有一个标签，其中标签值表示桶的上限。
（没有这样的标签的样品被静默地忽略。）
直方图度量类型自动为`_bucket`后缀和相应的标签提供时间序列。

使用`rate()`函数来指定分位数计算的时间窗口。

示例：直方图度量标准称为`http_request_duration_seconds`。要计算最近10米的请求持续时间的第90百分位，请使用以下表达式：
```
histogram_quantile(0.9, rate(http_request_duration_seconds_bucket[10m]))
```
在`http_request_duration_seconds`中为每个标签组合计算分位数。
要进行聚合，请使用`sum()`聚合器包裹`rate()`函数。
由于`le`标签是`histogram_quantile()`所要求的，所以它必须包含在by子句中。
以下表达式按工作汇总第90百分位：
```
histogram_quantile(0.9, sum(rate(http_request_duration_seconds_bucket[10m])) by (job, le))
```
要汇总所有内容，只需指定标签：
```
histogram_quantile(0.9, sum(rate(http_request_duration_seconds_bucket[10m])) by (le))
```
`histogram_quantile()`函数通过假定桶内的线性分布来插值分位数值。
最高的桶必须具有`+Inf`的上限。 （否则，返回NaN。）
如果分位数位于最高分组中，则返回第二高分组的上限。
如果该桶的上限大于0，则最低桶的下限被假定为0。
在这种情况下，通常的线性内插应用在该桶内。
否则，返回位于最低桶中的分位数的最低桶的上限。

如果`b`包含少于两个桶，则返回`NaN`, 对于`φ<0`，返回`-Inf`, 对于`φ>1`，返回`+Inf`。



## `holt_winters()`
`holt_winters(v range-vector, sf scalar, tf scalar)`根据`v`中的范围产生时间序列的平滑值。
平滑因子`sf`越低，对旧数据的重视程度越高。趋势因子`tf`越高，数据的趋势就越多。`sf`和`tf`必须在0和1之间。

`holt_winters`只能与`gauges`一块使用。




## hour()
hour(v=vector(time()) instant-vector) returns the hour of the day for each of the given times in UTC. Returned values are from 0 to 23.





## `idelta()`

`idelta(v range-vector)`计算范围矢量`v`中最后两个样本之间的差异，**返回增量和等效标签的瞬时矢量**。

`idelta`只能与`gauges`一块使用。






## `increase()`

`increase(v range-vector)` 计算范围矢量中时间序列的增量。 中断后（例如由于目标重新启动而产生的计数器重置）会会自动调整。
这个增量值是按照范围矢量选择器中指定的全部时间范围计算出来的，所以即使计数器只增加了整数增量，也可能得到一个非整数结果。

以下示例表达式将返回范围矢量中每个时间序列在**过去5分钟内测得的HTTP请求数**：
```
increase(http_requests_total{job="api-server"}[5m])
```
`increase`只能与`counters`一块使用。这个是`rate(v)`乘以**指定时间范围窗口内秒数**的语法糖，主要用于人的可读性。应在`recording rules`中使用`rate`，以便每秒地跟踪一致的增量。





## `irate()`

`irate(v range-vector)` 计算范围矢量中时间序列的每秒**瞬时增长​​率**（**instant rate**）。
它基于最后两个数据点。中断后会自动调整（例如由于目标重新启动而产生的计数器重置）。

下面的示例表达式将返回距离矢量中每个时间序列的**最近两个数据点的最近5分钟的HTTP请求速率**：
```
irate(http_requests_total{job="api-server"}[5m])
```
只有在绘制易变、快速移动的计数器时才使用`irate`。
对**警报和移动缓慢的计数器**应使用`rate`，因为`rate`的简要变化会重置`FOR`子句，而**完全由少量的尖峰组成的图形**（`irate`）是很难读取的。

> **请注意**，当`irate()`与汇总运算符（例如`sum()`）或一个随时间聚合的函数（任何以`_over_time`结尾的函数）结合时， 一定要先使用`irate()`，然后再采用汇总。 否则，当目标重新启动时，`irate()` 不能检测到计数器重置。







## `label_join()`
对于每个时间序列 `v`, `label_join(v instant-vector, dst_label string, separator string, src_label_1 string, src_label_2 string, ...)`使用`separator`连接所有`src_labels`的值，并返回包含连接值的标签`dst_label`的时间序列。这个函数中可以有任意数量的`src_label`。

这个例子将返回一个有`foo`标签的时间序列，其值为`a，b，c`。
```
label_join(up{job="api-server",src1="a",src2="b",src3="c"}, "foo", ",", "src1", "src2", "src3")
```






## `label_replace()`

`label_replace(v instant-vector, dst_label string, replacement string, src_label string, regex string)`将正则表达式`regex `与标签`src_label`进行匹配, 如果匹配，则标签`src_label`将被`replacement`表达式抽取 成为标签`dst_label`, `$1`替换为第一个匹配的子组，`$2`替换第二个等。如果正则表达式不匹配，那么时间序列不变。

这个例子将返回一个矢量，每个时间序列都有一个`foo`标签，其值为`a`：
```
label_replace(up{job="api-server",service="a:c"}, "foo", "$1", "service", "(.*):.*")
```
> 该命令并不是标签替换，而是只抽取需要的标签内容






## ln()
ln(v instant-vector) calculates the natural logarithm for all elements in v. Special cases are:

ln(+Inf) = +Inf
ln(0) = -Inf
ln(x < 0) = NaN
ln(NaN) = NaN



## log2()
log2(v instant-vector) calculates the binary logarithm for all elements in v. The special cases are equivalent to those in ln.

## log10()
log10(v instant-vector) calculates the decimal logarithm for all elements in v. The special cases are equivalent to those in ln.



## minute()
minute(v=vector(time()) instant-vector) returns the minute of the hour for each of the given times in UTC. Returned values are from 0 to 59.



## month()
month(v=vector(time()) instant-vector) returns the month of the year for each of the given times in UTC. Returned values are from 1 to 12, where 1 means January etc.



## predict_linear()
predict_linear(v range-vector, t scalar) predicts the value of time series t seconds from now, based on the range vector v, using simple linear regression.

predict_linear should only be used with gauges.






## `rate()`

`rate(v range-vector)` 计算范围矢量中时间序列的**每秒平均增长率**。
中断后会自动调整（例如由于目标重新启动而产生的计数器重置）。
此外，计算推断到时间范围的末尾，从而**允许错误的采集**或**采集周期与范围的时间段的不完全对齐**。

> 这里应该是建议时间范围的选取最好与采集周期一致

以下示例表达式将返回范围矢量中,每个时间序列在**过去5分钟内的每秒HTTP请求速率**：
```
rate(http_requests_total{job="api-server"}[5m])
```
`rate`只能与计数器（counters）已使用。最适合警报，并绘制缓慢移动的图表。

> **请注意**，当`rate()`与汇总运算符（例如`sum()`）或一个随时间聚合的函数（任何以`_over_time`结尾的函数）结合时， 一定要先使用`rate()`，然后再采用汇总。 否则，当目标重新启动时，`rate()` 不能检测到计数器重置。






## `resets()`

对于每个输入时间序列，`resets(v range-vector)` 将所提供时**间范围内的计数器重置次数作为瞬时矢量**返回。两个连续采样之间值的任何减少，都被认为是计数器复位。

如下：查询 `kail_counter_requests` **最近一天的中断次数**（**中断往往是由于被监控服务重启导致的**） 
```
resets(kail_counter_requests[1d])
```

`resets`只能与计数器（counters）已使用。









## round()
round(v instant-vector, to_nearest=1 scalar) rounds the sample values of all elements in v to the nearest integer. 
Ties are resolved by rounding up. 
The optional to_nearest argument allows specifying the nearest multiple to which the sample values should be rounded. 
This multiple may also be a fraction.



## scalar()
Given a single-element input vector, scalar(v instant-vector) returns the sample value of that single element as a scalar. If the input vector does not have exactly one element, scalar will return NaN.



## `sort()`
`sort(v instant-vector)` 以升序返回按其样本值排序的矢量元素



## `sort_desc()`
与`sort`相同，但按降序排列。



## sqrt()
sqrt(v instant-vector) calculates the square root of all elements in v.



## time()
time() returns the number of seconds since January 1, 1970 UTC. Note that this does not actually return the current time, but the time at which the expression is to be evaluated.



## timestamp()
timestamp(v instant-vector) returns the timestamp of each of the samples of the given vector as the number of seconds since January 1, 1970 UTC.

This function was added in Prometheus 2.0




## `vector()`
vector(s scalar) returns the scalar s as a vector with no labels.
`vector(s scalar)`将标量`s`作为无标签的矢量返回




## `year()`
`year(v=vector(time()) instant-vector)` 返回每个给定时间的UTC年份





## `<aggregation>_over_time()`

以下函数允许汇总给定的范围矢量，并返回每个序列聚合后的结果，该结果是瞬时矢量：

- `avg_over_time(range-vector)`: 平均值
- `min_over_time(range-vector)`: 最小值
- `max_over_time(range-vector)`: 最大值
- `sum_over_time(range-vector)`: 求和
- `count_over_time(range-vector)`: 个数
- `quantile_over_time(scalar, range-vector)`:  φ-[分位数](https://baike.baidu.com/item/分位数/10064158) (0 ≤ φ ≤ 1) 
- `stddev_over_time(range-vector)`: 标准差.
- `stdvar_over_time(range-vector)`: 方差.

**请注意**，即使在整个时间间隔内值不是均匀分布，指定时间间隔内的所有值在聚合中的权重也相同。