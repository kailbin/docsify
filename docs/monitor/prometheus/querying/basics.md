
# 查询 Prometheus

Prometheus提供了一种表达式语言，可让用户实时选择和汇总时间序列数据。
表达式的结果可以显示为图形，可以在Prometheus的表达式浏览器中以表格形式查看，或者通过[HTTP API](prometheus/querying/http-api.md)供外部系统调用。

## Examples
本文件仅供参考。为了快速学习，从[这几个例子](prometheus/querying/examples.md)开始可能会更容易一些。

## 表达式语言数据类型(Expression language data types)

在 Prometheus 的表达语言中，表达式或子表达式可以以下为四种类型：

- **Instant vector(瞬时矢量)** - 一组时间序列，每个时间序列包含单个样本，全部共享相同的时间戳
- **Range vector(范围矢量)** - 一组时间序列，每个时间序列包含一系列随时间变化的数据点
- **Scalar(纯量)** - 一个简单的数字浮点值
- **String(字符串)** - 一个简单的字符串值;, 目前尚未使用

根据用例（例如，在绘制图形和显示表达式输出时），这些类型中只有一些类型才能作为用户指定表达式的合法结果。
例如，返回**瞬时矢量的表达式是唯一可以直接绘制的类型**。

## Literals

### String literals

字符串可以用单引号（`single quotes`），双引号（`double quotes`）或反引号（`backticks`）指定。

PromQL遵循与Go相同的[转义规则](https://golang.org/ref/spec#String_literals)。
在单引号或双引号中，反斜杠开始一个转义序列，后面可以跟着`a`，`b`，`f`，`n`，`r`，`t`，`v`或`\`。
可以使用八进制（`\nnn`）或十六进制（`\xnn`，`\unnnn`和`\Unnnnnnnn`）来提供特定字符。

No escaping is processed inside backticks. Unlike Go, Prometheus does not discard newlines inside backticks.
反引号内没有处理反引号。 不像Go，Prometheus不会在反引号内放弃换行符。

Example:

```
"this is a string"
'these are unescaped: \n \\ \t'
`these are not unescaped: \n ' " \t`
```


### Float literals

标量浮点值可以从字面上写成`[-](digits)[.(digits)]`的形式。

```
-2.43
```




## Time series Selectors

### Instant vector selectors

Instant vector selectors allow the selection of a set of time series and a single sample value for each at a given timestamp (instant): in the simplest form, only a metric name is specified. 
即时矢量选择器允许在给定的时间戳（即时）中为每个时间序列和每个样本值选择一组时间序列：以最简单的形式，只指定度量名称。
This results in an instant vector containing elements for all time series that have this metric name.
这将导致包含具有此度量标准名称的所有时间序列元素的即时矢量。

This example selects all time series that have the `http_requests_total` metric name:
本示例选择具有“http_requests_total”度量标准名称的所有时间序列：

```
http_requests_total
```
It is possible to filter these time series further by appending a set of labels to match in curly braces ({}).
可以通过在花括号（{}）中添加一组要匹配的标签来进一步过滤这些时间序列。

This example selects only those time series with the http_requests_total metric name that also have the job label set to prometheus and their group label set to canary:
本示例仅选择具有http_requests_total度量标准名称的作业标签设置为prometheus且其组标签设置为canary的时间序列：
```
http_requests_total{job="prometheus",group="canary"}
```
It is also possible to negatively match a label value, or to match label values against regular expressions. The following label matching operators exist:
也可能负面匹配标签值，或将标签值与正则表达式匹配。, 存在以下标签匹配运算符：

- =: Select labels that are exactly equal to the provided string.
- !=: Select labels that are not equal to the provided string.
- =~: Select labels that regex-match the provided string (or substring).
- !~: Select labels that do not regex-match the provided string (or substring).

For example, this selects all http_requests_total time series for staging, testing, and development environments and HTTP methods other than GET.
例如，这将选择所有http_requests_total时间序列用于分段，测试和开发环境以及GET以外的HTTP方法。

```
http_requests_total{environment=~"staging|testing|development",method!="GET"}
```
Label matchers that match empty label values also select all time series that do not have the specific label set at all. Regex-matches are fully anchored.
匹配空标签值的标签匹配器也会选择所有没有设置特定标签的时间序列。, 正则表达式匹配完全锚定。

Vector selectors must either specify a name or at least one label matcher that does not match the empty string. The following expression is illegal:
矢量选择器必须指定一个名称或至少一个与空字符串不匹配的标签匹配器。, 以下表达式是非法的：
```
{job=~".*"} # Bad!
```
In contrast, these expressions are valid as they both have a selector that does not match empty label values.
相反，这些表达式是有效的，因为它们都有一个不匹配空标签值的选择器。
```
{job=~".+"}              # Good!
{job=~".*",method="get"} # Good!
```

Label matchers can also be applied to metric names by matching against the internal __name__ label. For example, the expression http_requests_total is equivalent to {__name__="http_requests_total"}. Matchers other than = (!=, =~, !~) may also be used. The following expression selects all metrics that have a name starting with job::
标签匹配器也可以通过匹配内部的__name__标签来应用到度量标准名称。, 例如，表达式http_requests_total相当于{__name __ =“http_requests_total”}。, 除了=（！=，=〜，！〜）之外的匹配符也可以被使用。, 以下表达式选择名称以job ::开头的所有度量标准
```
{__name__=~"job:.*"}
```
All regular expressions in Prometheus use RE2 syntax.
普罗米修斯的所有正则表达式都使用RE2语法。

### Range Vector Selectors

Range vector literals work like instant vector literals, except that they select a range of samples back from the current instant. Syntactically, a range duration is appended in square brackets ([]) at the end of a vector selector to specify how far back in time values should be fetched for each resulting range vector element.
范围矢量文字就像即时矢量文字一样工作，只不过它们从当前时刻开始选择一个范围的样本。, 在句法上，范围持续时间被附加在矢量选择器末尾的方括号（[]）中，以指定每个得到的范围矢量元素应该取回多久的时间值。
Time durations are specified as a number, followed immediately by one of the following units:
持续时间被指定为一个数字，紧接着是以下单位之一：

- s - seconds
- m - minutes
- h - hours
- d - days
- w - weeks
- y - years

In this example, we select all the values we have recorded within the last 5 minutes for all time series that have the metric name http_requests_total and a job label set to prometheus:
在本例中，我们选择了所有时间序列中过去5分钟内记录的所有具有度量标准名称http_requests_total和作业标签设置为prometheus的值：
```
http_requests_total{job="prometheus"}[5m]
```

### Offset modifier

The offset modifier allows changing the time offset for individual instant and range vectors in a query.
偏移修改器允许更改查询中单个瞬时和范围矢量的时间偏移。

For example, the following expression returns the value of http_requests_total 5 minutes in the past relative to the current query evaluation time:
例如，以下表达式相对于当前查询评估时间返回过去5分钟的http_requests_total值：
```
http_requests_total offset 5m
```

Note that the offset modifier always needs to follow the selector immediately, i.e. the following would be correct:
请注意，偏移量修改器总是需要立即跟随选择器，即以下内容是正确的：
```
sum(http_requests_total{method="GET"} offset 5m) // GOOD.
```
While the following would be incorrect:
虽然以下将是不正确的：
```
sum(http_requests_total{method="GET"}) offset 5m // INVALID.
```
The same works for range vectors. This returns the 5-minutes rate that http_requests_total had a week ago:
范围矢量相同的作品。, 这将返回http_requests_total一周前的5分钟速率：
```
rate(http_requests_total[5m] offset 1w)
```

## Operators
Prometheus supports many binary and aggregation operators. These are described in detail in the expression language operators page.
普罗米修斯支持许多二进制和聚合运算符。, 这些在表达式语言运算符页面中详细描述。


## Functions
Prometheus supports several functions to operate on data. These are described in detail in the expression language functions page.
Prometheus支持多种功能来操作数据。, 这些在表达式语言功能页面中详细描述。

## Gotchas

### Staleness

When queries are run, timestamps at which to sample data are selected independently of the actual present time series data. This is mainly to support cases like aggregation (sum, avg, and so on), where multiple aggregated time series do not exactly align in time. Because of their independence, Prometheus needs to assign a value at those timestamps for each relevant time series. It does so by simply taking the newest sample before this timestamp.
当运行查询时，独立于当前实际时间序列数据选择要对数据进行采样的时间戳。, 这主要是为了支持像聚合（总和，平均等），其中多个聚合时间序列不及时准确对齐的情况。, 由于其独立性，普罗米修斯需要为每个相关时间序列在这些时间戳上分配一个值。, 它通过简单地在这个时间戳之前采取最新的样本。

If a target scrape or rule evaluation no longer returns a sample for a time series that was previously present, that time series will be marked as stale. If a target is removed, its previously returned time series will be marked as stale soon afterwards.
如果目标刮擦或规则评估不再返回先前存在的时间序列的样本，则该时间序列将被标记为陈旧。, 如果一个目标被删除，之前返回的时间序列将很快被标记为陈旧。

If a query is evaluated at a sampling timestamp after a time series is marked stale, then no value is returned for that time series. If new samples are subsequently ingested for that time series, they will be returned as normal.
如果在时间序列被标记为陈旧之后，在采样时间戳处评估查询，则该时间序列不会返回任何值。, 如果新的样本随后被摄取，那么它们将被正常返回。

If no sample is found (by default) 5 minutes before a sampling timestamp, no value is returned for that time series at this point in time. This effectively means that time series "disappear" from graphs at times where their latest collected sample is older than 5 minutes or after they are marked stale.
如果在采样时间戳之前5分钟没有找到样本（默认情况下），则此时不会为该时间序列返回任何值。, 这实际上意味着时间序列从最近收集的样本超过5分钟或标记为陈旧之后的图中“消失”。

Staleness will not be marked for time series that have timestamps included in their scrapes. Only the 5 minute threshold will be applied in that case.
过时不会被标记为在时间戳中包含时间戳的时间序列。, 在这种情况下，只有5分钟的门槛。

### Avoiding slow queries and overloads

If a query needs to operate on a very large amount of data, graphing it might time out or overload the server or browser. Thus, when constructing queries over unknown data, always start building the query in the tabular view of Prometheus's expression browser until the result set seems reasonable (hundreds, not thousands, of time series at most). Only when you have filtered or aggregated your data sufficiently, switch to graph mode. If the expression still takes too long to graph ad-hoc, pre-record it via a recording rule.
如果查询需要处理大量的数据，则可能会超时或超载服务器或浏览器。, 因此，在构建对未知数据的查询时，始终在Prometheus的表达式浏览器的表格视图中开始构建查询，直到结果集似乎合理（最多有数百个，而不是数千个时间序列）。, 只有当您已经过滤或聚合足够的数据时，切换到图形模式。, 如果表达式仍然需要太长的时间才能图形化，请通过录制规则预先录制。

This is especially relevant for Prometheus's query language, where a bare metric name selector like api_http_requests_total could expand to thousands of time series with different labels. Also keep in mind that expressions which aggregate over many time series will generate load on the server even if the output is only a small number of time series. This is similar to how it would be slow to sum all values of a column in a relational database, even if the output value is only a single number.
这对于普罗米修斯的查询语言来说尤为重要，在这种语言中，像api_http_requests_total这样的公制名称选择器可以扩展到数千个具有不同标签的时间序列。, 还要记住，即使输出只是少数几个时间序列，在许多时间序列上聚合的表达式也会在服务器上产生负载。, 这与在关系数据库中总结列的所有值的速度相似，即使输出值只是单个数字。
