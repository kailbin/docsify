
# Check - Agent HTTP API

`/agent/check` 接口与 Consul 本地代理的检查相互作用。不应与catalog中的检查混淆。


## List Checks

该接口返回所有注册到本地 agent 中的 checks。
这些 checks 要么通过配置文件提供，要么使用HTTP API动态添加。

需要注意的是，agent 所知道的检查可能不同于 catalog 所报告的检查。
agent 会参与[anti-entropy](/docs/internals/anti-entropy.html),所以通常情况下，所有事情会在几秒内完成同步。

| Method | Path                         | Produces                   |
| ------ | ---------------------------- | -------------------------- |
| `GET`  | `/agent/checks`              | `application/json`         |

The table below shows this endpoint's support for [blocking queries](/api/index.html#blocking-queries), [consistency modes](/api/index.html#consistency-modes), and [required ACLs](/api/index.html#acls).

| Blocking Queries | Consistency Modes | ACL Required             |
| ---------------- | ----------------- | ------------------------ |
| `NO`             | `none`            | `node:read,service:read` |

### Sample Request

``` text
$ curl  https://consul.rocks/v1/agent/checks
```

### Sample Response

``` json
{
  "service:redis": {
    "Node": "foobar",
    "CheckID": "service:redis",
    "Name": "Service 'redis' check",
    "Status": "passing",
    "Notes": "",
    "Output": "",
    "ServiceID": "redis",
    "ServiceName": "redis",
    "ServiceTags": ["primary"]
  }
}
```

.  
.  
.  
.  
.  
.  
.  


## Register Check

该接口向本地代理添加一个新的检查。**Checks 可以是 `script`, `HTTP`, `TCP`, 或者 `TTL` 类型**。agent 负责管理检查的状态并保持`Catalog`同步。

| Method | Path                         | Produces                   |
| ------ | ---------------------------- | -------------------------- |
| `PUT`  | `/agent/check/register`      | `application/json`         |

The table below shows this endpoint's support for [blocking queries](/api/index.html#blocking-queries), [consistency modes](/api/index.html#consistency-modes), and [required ACLs](/api/index.html#acls).

| Blocking Queries | Consistency Modes | ACL Required               |
| ---------------- | ----------------- | -------------------------- |
| `NO`             | `none`            | `node:write,service:write` |

### Parameters

- `Name` `(string: <required>)` - 指定 check 的名称

- `ID` `(string: "")` - 节点上 check 的唯一 ID. 如果没指定 默认是 `Name`, 但是 提供唯一ID一定必要的.

- `Interval` `(string: "")` - 指定 check 执行的频率. 要求类型是 `HTTP` 和 `TCP` checks.

- `Notes` `(string: "")` - Consul 内部无用，是用来让人看的.

- `DeregisterCriticalServiceAfter` `(string: "")` - service 关联的这个检查应该在这个时间后被注销. 可以使用类似于 "10m" 的时间区间进行指定。如果 check 在 critical 状态超过了这个配置值, 与其关联的 service 会自动注销。 
  最小只能设为1分钟, 每30秒会统计一次critical状态的服务, 所以撤销注册可能比配置的时间稍微长一点. 

- `Args` `(array<string>)` - 指定check执行的的命令行参数。
  Prior to Consul 1.0, checks used a single `Script` field to define the command to run, and would always run in a shell. 
  In Consul 1.0, the `Args` array was added so that checks can be run without a shell. 
  The `Script` field is deprecated, and you should include the shell in the `Args` to run under a shell, eg. `"args": ["sh", "-c", "..."]`.

  -> **Note:** Consul 1.0 shipped with an issue where `Args` was erroneously named `ScriptArgs` in this API. 
    Please use `ScriptArgs` with Consul 1.0 (that will continue to be accepted in future versions of Consul), and `Args` in Consul 1.0.1 and later.

- `DockerContainerID` `(string: "")` - Specifies that the check is a Docker check, and Consul will evaluate the script every `Interval` in the given container using the specified `Shell`. 
  Note that `Shell` is currently only supported for Docker checks.


- `HTTP` `(string: "")` - Specifies an `HTTP` check to perform a `GET` request against the value of `HTTP` (expected to be a URL) every `Interval`. 
  If the response is any `2xx` code, the check is `passing`. If the response is `429 Too Many Requests`, the check is `warning`. Otherwise, the check is `critical`. 
  HTTP checks also support SSL. By default, a valid SSL certificate is expected. Certificate verification can be controlled using the `TLSSkipVerify`.

- `Method` `(string: "")` - Specifies a different HTTP method to be used for an `HTTP` check. When no value is specified, `GET` is used.

- `Header` `(map[string][]string: {})` - Specifies a set of headers that should be set for `HTTP` checks. Each header can have multiple values.

- `TLSSkipVerify` `(bool: false)` - Specifies if the certificate for an HTTPS check should not be verified.

- `TCP` `(string: "")` - Specifies a `TCP` to connect against the value of `TCP` (expected to be an IP or hostname plus port combination) every `Interval`. 
  If the connection attempt is successful, the check is `passing`. 
  If the connection attempt is unsuccessful, the check is `critical`. 
  In the case of a hostname that resolves to both IPv4 and IPv6 addresses, an attempt will be made to both addresses, and the first successful connection attempt will result in a successful check.

- `TTL` `(string: "")` - 指定这是一个TTL检查，使用TTL类型必须定期来更新检查的状态。

- `ServiceID` `(string: "")` - 指定服务的ID，以将注册的检查与agent提供的服务关联。

- `Status` `(string: "")` - 指定健康检查的初始化状态.

### Sample Payload

``` json
{
  "ID": "mem",
  "Name": "Memory utilization",
  "Notes": "Ensure we don't oversubscribe memory",
  "DeregisterCriticalServiceAfter": "90m",
  "Args": ["/usr/local/bin/check_mem.py"],
  "DockerContainerID": "f972c95ebf0e",
  "Shell": "/bin/bash",
  "HTTP": "https://example.com",
  "Method": "POST",
  "Header": {"x-foo":["bar", "baz"]},
  "TCP": "example.com:22",
  "Interval": "10s",
  "TTL": "15s",
  "TLSSkipVerify": true
}
```

### Sample Request

``` text
$ curl \
   --request PUT \
   --data @payload.json \
   https://consul.rocks/v1/agent/check/register
```

.  
.  
.  
.  
.  
.  
.  


## Deregister Check

This endpoint remove a check from the local agent. The agent will take care of
deregistering the check from the catalog. If the check with the provided ID does
not exist, no action is taken.

| Method | Path                                | Produces                   |
| ------ | ----------------------------------- | -------------------------- |
| `PUT`  | `/agent/check/deregister/:check_id` | `application/json`         |

The table below shows this endpoint's support for
[blocking queries](/api/index.html#blocking-queries),
[consistency modes](/api/index.html#consistency-modes), and
[required ACLs](/api/index.html#acls).

| Blocking Queries | Consistency Modes | ACL Required               |
| ---------------- | ----------------- | -------------------------- |
| `NO`             | `none`            | `node:write,service:write` |

### Parameters

- `check_id` `(string: "")` - Specifies the unique ID of the check to
  deregister. This is specified as part of the URL.

### Sample Request

```text
$ curl \
    --request PUT \
    https://consul.rocks/v1/agent/check/deregister/my-check-id
```

.  
.  
.  
.  
.  
.  
.  


## TTL Check Pass

TTL 类型的 check 用来设置 check 状态为`passing` 并重置 TTL 时钟.

| Method | Path                          | Produces                   |
| ------ | ----------------------------- | -------------------------- |
| `PUT`  | `/agent/check/pass/:check_id` | `application/json`         |

The table below shows this endpoint's support for [blocking queries](/api/index.html#blocking-queries), [consistency modes](/api/index.html#consistency-modes), and [required ACLs](/api/index.html#acls).

| Blocking Queries | Consistency Modes | ACL Required               |
| ---------------- | ----------------- | -------------------------- |
| `NO`             | `none`            | `node:write,service:write` |

### Parameters

- `check_id` `(string: "")` - Specifies the unique ID of the check to use. This is specified as part of the URL.

- `note` `(string: "")` - Specifies a human-readable message. This will be passed through to the check's `Output` field.

### Sample Request

```text
$ curl \
    https://consul.rocks/v1/agent/check/pass/my-check-id
```

.  
.  
.  
.  
.  
.  
.  


## TTL Check Warn

This endpoint is used with a TTL type check to set the status of the check to `warning` and to reset the TTL clock.

| Method | Path                          | Produces                   |
| ------ | ----------------------------- | -------------------------- |
| `PUT`  | `/agent/check/warn/:check_id` | `application/json`         |

The table below shows this endpoint's support for
[blocking queries](/api/index.html#blocking-queries),
[consistency modes](/api/index.html#consistency-modes), and
[required ACLs](/api/index.html#acls).

| Blocking Queries | Consistency Modes | ACL Required               |
| ---------------- | ----------------- | -------------------------- |
| `NO`             | `none`            | `node:write,service:write` |

### Parameters

- `check_id` `(string: "")` - Specifies the unique ID of the check to
  use. This is specified as part of the URL.

- `note` `(string: "")` - Specifies a human-readable message. This will be
  passed through to the check's `Output` field.

### Sample Request

```text
$ curl \
    https://consul.rocks/v1/agent/check/warn/my-check-id
```

.  
.  
.  
.  
.  
.  
.  


## TTL Check Fail

This endpoint is used with a TTL type check to set the status of the check to `critical` and to reset the TTL clock.

| Method | Path                          | Produces                   |
| ------ | ----------------------------- | -------------------------- |
| `PUT`  | `/agent/check/fail/:check_id` | `application/json`         |

The table below shows this endpoint's support for
[blocking queries](/api/index.html#blocking-queries),
[consistency modes](/api/index.html#consistency-modes), and
[required ACLs](/api/index.html#acls).

| Blocking Queries | Consistency Modes | ACL Required               |
| ---------------- | ----------------- | -------------------------- |
| `NO`             | `none`            | `node:write,service:write` |

### Parameters

- `check_id` `(string: "")` - Specifies the unique ID of the check to
  use. This is specified as part of the URL.

- `note` `(string: "")` - Specifies a human-readable message. This will be
  passed through to the check's `Output` field.

### Sample Request

```text
$ curl \
    https://consul.rocks/v1/agent/check/fail/my-check-id
```

.  
.  
.  
.  
.  
.  
.  


## TTL Check Update

This endpoint is used with a TTL type check to set the status of the check and to reset the TTL clock.

| Method | Path                            | Produces                   |
| ------ | ------------------------------- | -------------------------- |
| `PUT`  | `/agent/check/update/:check_id` | `application/json`         |

The table below shows this endpoint's support for
[blocking queries](/api/index.html#blocking-queries),
[consistency modes](/api/index.html#consistency-modes), and
[required ACLs](/api/index.html#acls).

| Blocking Queries | Consistency Modes | ACL Required               |
| ---------------- | ----------------- | -------------------------- |
| `NO`             | `none`            | `node:write,service:write` |

### Parameters

- `check_id` `(string: "")` - Specifies the unique ID of the check to
  use. This is specified as part of the URL.

- `Status` `(string: "")` - Specifies the status of the check. Valid values are
  `"passing"`, `"warning"`, and `"critical"`.

- `Output` `(string: "")` - Specifies a human-readable message. This will be
  passed through to the check's `Output` field.

### Sample Payload

```json
{
  "Status": "passing",
  "Output": "curl reported a failure:\n\n..."
}
```

### Sample Request

```text
$ curl \
    --request PUT \
    --data @payload.json \
    https://consul.rocks/v1/agent/check/update/my-check-id
```