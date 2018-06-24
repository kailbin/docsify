# HTTP API

Prometheus服务器上的`/api/v1`下可以访问当前稳定的HTTP API。

## 格概览

API响应格式是`JSON`。每个成功的API请求都会返回一个`2xx`状态码。

Invalid requests that reach the API handlers return a JSON error object and one of the following HTTP response codes:
API无法处理的请求，会返回JSON错误对象 和 以下HTTP响应代码：

- `400` 错误的请求，当参数丢失或不正确的时候出现
- `422` 不可处理的实体，当表达式不能被执行时（[RFC4918](https://tools.ietf.org/html/rfc4918#page-78)）。
- `503` 服务不可用，查询超时或者终止的时候出现.

出现其他非`2xx`代码，可能是请求在还没走到指定的API。

JSON响应信封格式如下：
```
{
  "status": "success" | "error",
  "data": <data>,

  // Only set if status is "error". The data field may still hold additional data.
  "errorType": "<string>",
  "error": "<string>"
}
```
Input timestamps may be provided either in RFC3339 format or as a Unix timestamp in seconds, with optional decimal places for sub-second precision. Output timestamps are always represented as Unix timestamps in seconds.
输入时间戳可以以RFC3339格式提供，也可以以秒为单位提供Unix时间戳，可选的小数位数为亚秒级精度。, 输出时间戳总是以秒表示为Unix时间戳。

Names of query parameters that may be repeated end with [].
查询参数的名称可以重复以[]结尾。

<series_selector> placeholders refer to Prometheus time series selectors like http_requests_total or http_requests_total{method=~"(GET|POST)"} and need to be URL-encoded.
<series_selector>占位符引用普罗米修斯时间序列选择器，如http_requests_total或http_requests_total {method =〜“（GET | POST）”}，并且需要进行URL编码。

<duration> placeholders refer to Prometheus duration strings of the form [0-9]+[smhdwy]. For example, 5m refers to a duration of 5 minutes.
<duration>占位符指形式为[0-9] + [smhdwy]的普罗米修斯持续时间字符串。, 例如，5米是指5分钟的持续时间。






## Expression queries
Query language expressions may be evaluated at a single instant or over a range of time. The sections below describe the API endpoints for each type of expression query.
查询语言表达式可以在一个瞬间或在一定时间范围内进行评估。, 以下部分描述了每种类型的表达式查询的API端点。

### Instant queries

The following endpoint evaluates an instant query at a single point in time:
以下端点在单个时间点评估即时查询：

```
GET /api/v1/query
```
URL query parameters:

- query=<string>: Prometheus expression query string.
- time=<rfc3339 | unix_timestamp>: Evaluation timestamp. Optional.
- timeout=<duration>: Evaluation timeout. Optional. Defaults to and is capped by the value of the -query.timeout flag.

The current server time is used if the time parameter is omitted.
如果省略时间参数，则使用当前服务器时间。

The data section of the query result has the following format:
查询结果的数据部分具有以下格式：
```
{
  "resultType": "matrix" | "vector" | "scalar" | "string",
  "result": <value>
}
```
<value> refers to the query result data, which has varying formats depending on the resultType. See the expression query result formats.
<value>是指查询结果数据，根据resultType具有不同的格式。, 查看表达式查询结果格式。

The following example evaluates the expression up at the time 2015-07-01T20:10:51.781Z:
以下示例在2015-07-01T20：10：51.781Z时评估表达式：

```
$ curl 'http://localhost:9090/api/v1/query?query=up&time=2015-07-01T20:10:51.781Z'
{
   "status" : "success",
   "data" : {
      "resultType" : "vector",
      "result" : [
         {
            "metric" : {
               "__name__" : "up",
               "job" : "prometheus",
               "instance" : "localhost:9090"
            },
            "value": [ 1435781451.781, "1" ]
         },
         {
            "metric" : {
               "__name__" : "up",
               "job" : "node",
               "instance" : "localhost:9100"
            },
            "value" : [ 1435781451.781, "0" ]
         }
      ]
   }
}
```



### Range queries

The following endpoint evaluates an expression query over a range of time:
```
GET /api/v1/query_range
```

URL query parameters:

- query=<string>: Prometheus expression query string.
- start=<rfc3339 | unix_timestamp>: Start timestamp.
- end=<rfc3339 | unix_timestamp>: End timestamp.
- step=<duration>: Query resolution step width.
- timeout=<duration>: Evaluation timeout. Optional. Defaults to and is capped by the value of the -query.timeout flag.

The data section of the query result has the following format:

```
{
  "resultType": "matrix",
  "result": <value>
}
```
For the format of the <value> placeholder, see the range-vector result format.

The following example evaluates the expression up over a 30-second range with a query resolution of 15 seconds.

```
$ curl 'http://localhost:9090/api/v1/query_range?query=up&start=2015-07-01T20:10:30.781Z&end=2015-07-01T20:11:00.781Z&step=15s'
{
   "status" : "success",
   "data" : {
      "resultType" : "matrix",
      "result" : [
         {
            "metric" : {
               "__name__" : "up",
               "job" : "prometheus",
               "instance" : "localhost:9090"
            },
            "values" : [
               [ 1435781430.781, "1" ],
               [ 1435781445.781, "1" ],
               [ 1435781460.781, "1" ]
            ]
         },
         {
            "metric" : {
               "__name__" : "up",
               "job" : "node",
               "instance" : "localhost:9091"
            },
            "values" : [
               [ 1435781430.781, "0" ],
               [ 1435781445.781, "0" ],
               [ 1435781460.781, "1" ]
            ]
         }
      ]
   }
}
```









## Querying metadata

### Finding series by label matchers

The following endpoint returns the list of time series that match a certain label set.

```
GET /api/v1/series
```


URL query parameters:

- match[]=<series_selector>: Repeated series selector argument that selects the series to return. At least one match[] argument must be provided.
- start=<rfc3339 | unix_timestamp>: Start timestamp.
- end=<rfc3339 | unix_timestamp>: End timestamp.

The data section of the query result consists of a list of objects that contain the label name/value pairs which identify each series.

The following example returns all series that match either of the selectors up or process_start_time_seconds{job="prometheus"}:
以下示例返回与选择器up或process_start_time_seconds {job =“prometheus”}匹配的所有系列：

```
$ curl -g 'http://localhost:9090/api/v1/series?match[]=up&match[]=process_start_time_seconds{job="prometheus"}'
{
   "status" : "success",
   "data" : [
      {
         "__name__" : "up",
         "job" : "prometheus",
         "instance" : "localhost:9090"
      },
      {
         "__name__" : "up",
         "job" : "node",
         "instance" : "localhost:9091"
      },
      {
         "__name__" : "process_start_time_seconds",
         "job" : "prometheus",
         "instance" : "localhost:9090"
      }
   ]
}
```


### Querying label values

The following endpoint returns a list of label values for a provided label name:

```
GET /api/v1/label/<label_name>/values
```
The data section of the JSON response is a list of string label names.

This example queries for all label values for the job label:
此示例查询作业标签的所有标签值：

```
$ curl http://localhost:9090/api/v1/label/job/values
{
   "status" : "success",
   "data" : [
      "node",
      "prometheus"
   ]
}
```


## Expression query result formats

Expression queries may return the following response values in the result property of the data section. <sample_value> placeholders are numeric sample values. JSON does not support special float values such as NaN, Inf, and -Inf, so sample values are transferred as quoted JSON strings rather than raw numbers.
表达式查询可能会在数据部分的结果属性中返回以下响应值。 , <sample_value>占位符是数字样本值。 , JSON不支持特殊的浮点值，如NaN，Inf和-Inf，因此样本值将作为带引号的JSON字符串而不是原始数据传输。

### Range vectors

```
Range vectors are returned as result type matrix. The corresponding result property has the following format:

[
  {
    "metric": { "<label_name>": "<label_value>", ... },
    "values": [ [ <unix_time>, "<sample_value>" ], ... ]
  },
  ...
]
```

### Instant vectors

Instant vectors are returned as result type vector. The corresponding result property has the following format:

```
[
  {
    "metric": { "<label_name>": "<label_value>", ... },
    "value": [ <unix_time>, "<sample_value>" ]
  },
  ...
]
```

### Scalars

Scalar results are returned as result type scalar. The corresponding result property has the following format:
```
[ <unix_time>, "<scalar_value>" ]
```

### Strings

String results are returned as result type string. The corresponding result property has the following format:
```
[ <unix_time>, "<string_value>" ]
```

## Targets
This API is experimental as it is intended to be extended with targets dropped due to relabelling in the future.
The following endpoint returns an overview of the current state of the Prometheus target discovery:
```
GET /api/v1/targets
```
Currently only the active targets are part of the response.
```
$ curl http://localhost:9090/api/v1/targets
{
  "status": "success",                                                                                                                                [3/11]
  "data": {
    "activeTargets": [
      {
        "discoveredLabels": {
          "__address__": "127.0.0.1:9090",
          "__metrics_path__": "/metrics",
          "__scheme__": "http",
          "job": "prometheus"
        },
        "labels": {
          "instance": "127.0.0.1:9090",
          "job": "prometheus"
        },
        "scrapeUrl": "http://127.0.0.1:9090/metrics",
        "lastError": "",
        "lastScrape": "2017-01-17T15:07:44.723715405+01:00",
        "health": "up"
      }
    ]
  }
}
```

## Alertmanagers
This API is experimental as it is intended to be extended with Alertmanagers dropped due to relabelling in the future.
The following endpoint returns an overview of the current state of the Prometheus alertmanager discovery:
```
GET /api/v1/alertmanagers
```
Currently only the active Alertmanagers are part of the response.
```
$ curl http://localhost:9090/api/v1/alertmanagers
{
  "status": "success",
  "data": {
    "activeAlertmanagers": [
      {
        "url": "http://127.0.0.1:9090/api/v1/alerts"
      }
    ]
  }
}
```


## TSDB Admin APIs
These are APIs that expose database functionalities for the advanced user. These APIs are not enabled unless the --web.enable-admin-api is set.
这些是为高级用户公开数据库功能的API。, 除非设置了--web.enable-admin-api，否则不会启用这些API。

We also expose a gRPC API whose definition can be found here. This is experimental and might change in the future.
我们还公开了一个gRPC API，其定义可以在这里找到。, 这是实验性的，未来可能会改变。

### Snapshot

Snapshot creates a snapshot of all current data into snapshots/<datetime>-<rand> under the TSDB's data directory and returns the directory as response.
快照将所有当前数据的快照创建到TSDB数据目录下的快照/ <datetime>  -  <rand>中，并将该目录作为响应返回。
```
POST /api/v1/admin/tsdb/snapshot
```

```
$ curl -XPOST http://localhost:9090/api/v1/admin/tsdb/snapshot
{
  "status": "success",
  "data": {
    "name": "20171210T211224Z-2be650b6d019eb54"
  }
}
```
The snapshot now exists at <data-dir>/snapshots/20171210T211224Z-2be650b6d019eb54

>New in v2.1

### Delete Series

DeleteSeries deletes data for a selection of series in a time range. The actual data still exists on disk and is cleaned up in future compactions or can be explicitly cleaned up by hitting the Clean Tombstones endpoint.
DeleteSeries删除时间范围内的一系列选择的数据。, 实际的数据仍然存在于磁盘上，并在未来的压缩中清除，或者通过点击Clean Tombstones端点来清除。
If successful, a 204 is returned.
```
POST /api/v1/admin/tsdb/delete_series
```
URL query parameters:

- match[]=<series_selector>: Repeated label matcher argument that selects the series to delete. At least one match[] argument must be provided.
- start=<rfc3339 | unix_timestamp>: Start timestamp. Optional and defaults to minimum possible time.
- end=<rfc3339 | unix_timestamp>: End timestamp. Optional and defaults to maximum possible time.

Not mentioning both start and end times would clear all the data for the matched series in the database.

Example:
```
$ curl -XPOST \                                                              
  -g 'http://localhost:9090/api/v1/admin/tsdb/delete_series?match[]=up&match[]=process_start_time_seconds{job="prometheus"}'
  ```
> New in v2.1



### Clean Tombstones

CleanTombstones removes the deleted data from disk and cleans up the existing tombstones. This can be used after deleting series to free up space.
CleanTombstones从磁盘中删除已删除的数据并清理现有的墓碑。, 这可以在删除系列以释放空间后使用。

If successful, a 204 is returned.
```
POST /api/v1/admin/tsdb/clean_tombstones
```
This takes no parameters or body.
这不需要参数或正文。

```
$ curl -XPOST http://localhost:9090/api/v1/admin/tsdb/clean_tombstones
```

> New in v2.1