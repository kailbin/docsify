
# Catalog HTTP API

`/catalog` 接口用来 `register`(`deregister`) `nodes`, `services`, 和`checks` 。`catalog`不应与`agent`混淆，虽然看起来很相似。


## Register Entity

这个接口是用于注册或更新catalog中的条目的底次机制，。
通常使用 [agent endpoints](api/agent/index.d)去操作更好，这样去执行 [anti-entropy](/docs/internals/anti-entropy.html)机制更简单。

| Method | Path                         | Produces                   |
| ------ | ---------------------------- | -------------------------- |
| `PUT`  | `/catalog/register`          | `application/json`         |

The table below shows this endpoint's support for [blocking queries](/api/index.html#blocking-queries), [consistency modes](/api/index.html#consistency-modes), and [required ACLs](/api/index.html#acls).

| Blocking Queries | Consistency Modes | ACL Required               |
| ---------------- | ----------------- | -------------------------- |
| `NO`             | `none`            | `node:write,service:write` |

### Parameters

- `ID` `(string: "")` - 为服务分配的 UUID . 如果没写会自动生成一个. 必须是 36 个字符的 UUID.

- `Node` `(string: <required>)` - 指定要注册的 Node ID.

- `Address` `(string: <required>)` - 指定注册地址.

- `Datacenter` `(string: "")` - 指定数据中心，如果没有指定使用默认的数据中心.

- `TaggedAddresses` `(map<string|string>: nil)` - 指定被标记的地址.

- `NodeMeta` `(map<string|string>: nil)` - 指定 KV 元数据对，用于过滤.

- `Service` `(Service: nil)` - 指定要注册的服务. 如果 `ID` 没有指定, 默认使用 `Service.Service` 属性值.
  一个服务与一个ID唯一对应， 提供给每个节点. 服务的 `Tags`, `Address`, 和 `Port` 是意义对应的.

- `Check` `(Check: nil)` - 指定注册的check. 
  该API会处理Catalog中运行的健康检车条目 , 但是不会设置 `script`, `TTL`, 或者 `HTTP` 去检查节点健康。 
  要真正的启用新注册的这个健康检查，该检查必须在 agent 中进行配置，或者通过 [agent endpoint](api/agent/index.md) 进行设置.
    
    - `CheckID` 可以省略，默认使用 `Name` 值。 和 `Service.ID` 一样 `CheckID` 必须唯一。 `Notes` 用了保存人可读的信息。
    如果 `ServiceID` 和节点的服务`ID` 是一样的, 则服务级别的健康检查会替换掉节点级别的健康检查. 
    `Status` 是 `passing`, `warning`, `critical` 其中之一。

    - `Definition`指定 TCP 或者 HTTP 的健康检查信息.  详情参见 [Health Checks](api/agent/checks.md) 页面.

    - 可以通过 `Checks` 替换 `Check` 来配置多种健康检查。.

- `SkipNodeUpdate` `(bool: false)` - 如果已经存在是否进行更新操作. 在需要更新节点上的健康检查或服务条目时有用。

需要注意的是，`Check`不一定要提供`Service`，反之亦然，catalog 配置的时候可以只有其中一个，也可以是两者都有。

### Sample Payload

``` json
{
  "Datacenter": "dc1",
  "ID": "40e4a748-2192-161a-0510-9bf59fe950b5",
  "Node": "foobar",
  "Address": "192.168.10.10",
  "TaggedAddresses": {
    "lan": "192.168.10.10",
    "wan": "10.0.10.10"
  },
  "NodeMeta": {
    "somekey": "somevalue"
  },
  "Service": {
    "ID": "redis1",
    "Service": "redis",
    "Tags": [
      "primary",
      "v1"
    ],
    "Address": "127.0.0.1",
    "Port": 8000
  },
  "Check": {
    "Node": "foobar",
    "CheckID": "service:redis1",
    "Name": "Redis health check",
    "Notes": "Script based health check",
    "Status": "passing",
    "ServiceID": "redis1",
    "Definition": {
      "TCP": "localhost:8888",
      "Interval": "5s",
      "Timeout": "1s",
      "DeregisterCriticalServiceAfter": "30s"
    }
  },
  "SkipNodeUpdate": false
}
```

### Sample Request

```text
$ curl \
    --request PUT \
    --data @payload.json \
    https://consul.rocks/v1/catalog/register
```

.  
.  
.  
.  
.  
.  
.  


## Deregister Entity

This endpoint is a low-level mechanism for directly removing entries from the Catalog. 
It is usually preferable to instead use the [agent endpoints](/api/agent.html) for deregistration as they are simpler and perform [anti-entropy](/docs/internals/anti-entropy.html).

| Method | Path                         | Produces                   |
| ------ | ---------------------------- | -------------------------- |
| `PUT`  | `/catalog/deregister`        | `application/json`         |

The table below shows this endpoint's support for [blocking queries](/api/index.html#blocking-queries), [consistency modes](/api/index.html#consistency-modes), and [required ACLs](/api/index.html#acls).

| Blocking Queries | Consistency Modes | ACL Required               |
| ---------------- | ----------------- | -------------------------- |
| `NO`             | `none`            | `node:write,service:write` |

### Parameters

The behavior of the endpoint depends on what keys are provided.

- `Node` `(string: <required>)` - Specifies the ID of the node. If no other values are provided, this node, all its services, and all its checks are removed.

- `Datacenter` `(string: "")` - Specifies the datacenter, which defaults to the agent's datacenter if not provided.

- `CheckID` `(string: "")` - Specifies the ID of the check to remove. 

- `ServiceID` `(string: "")` - Specifies the ID of the service to remove. The service and all associated checks will be removed.

### Sample Payloads

``` json
{
  "Datacenter": "dc1",
  "Node": "foobar"
}
```

``` json
{
  "Datacenter": "dc1",
  "Node": "foobar",
  "CheckID": "service:redis1"
}
```

``` json
{
  "Datacenter": "dc1",
  "Node": "foobar",
  "ServiceID": "redis1"
}
```

### Sample Request

```text
$ curl \
    --request PUT \
    --data @payload.json \
    https://consul.rocks/v1/catalog/deregister
```

.  
.  
.  
.  
.  
.  
.  


## List Datacenters

This endpoint returns the list of all known datacenters. 
The datacenters will be sorted in ascending order based on the estimated median round trip time from the server to the servers in that datacenter.

This endpoint does not require a cluster leader and will succeed even during an availability outage. 
Therefore, it can be used as a simple check to see if any Consul servers are routable.

| Method | Path                         | Produces                   |
| ------ | ---------------------------- | -------------------------- |
| `GET`  | `/catalog/datacenters`       | `application/json`         |

The table below shows this endpoint's support for [blocking queries](/api/index.html#blocking-queries), [consistency modes](/api/index.html#consistency-modes), and [required ACLs](/api/index.html#acls).

| Blocking Queries | Consistency Modes | ACL Required |
| ---------------- | ----------------- | ------------ |
| `NO`             | `none`            | `none`       |

### Sample Request

```text
$ curl \
    https://consul.rocks/v1/catalog/datacenters
```

### Sample Response

```json
["dc1", "dc2"]
```

.  
.  
.  
.  
.  
.  
.  


## List Nodes

This endpoint and returns the nodes registered in a given datacenter.

| Method | Path                         | Produces                   |
| ------ | ---------------------------- | -------------------------- |
| `GET`  | `/catalog/nodes`             | `application/json`         |

The table below shows this endpoint's support for [blocking queries](/api/index.html#blocking-queries), [consistency modes](/api/index.html#consistency-modes), and [required ACLs](/api/index.html#acls).

| Blocking Queries | Consistency Modes | ACL Required |
| ---------------- | ----------------- | ------------ |
| `YES`            | `all`             | `node:read`  |

### Parameters

- `dc` `(string: "")` - Specifies the datacenter to query. This will default to the datacenter of the agent being queried. This is specified as part of the URL as a query parameter.

- `near` `(string: "")` - Specifies a node name to sort the node list in ascending order based on the estimated round trip time from that node. Passing `?near=_agent` will use the agent's node for the sort. This is specified as part of the URL as a query parameter.

- `node-meta` `(string: "")` - Specifies a desired node metadata key/value pair of the form `key:value`. 
  This parameter can be specified multiple times, and will filter the results to nodes with the specified key/value pairs. 
  This is specified as part of the URL as a query parameter.

### Sample Request

```text
$ curl \
    https://consul.rocks/v1/catalog/nodes
```

### Sample Response

```json
[
  {
    "ID": "40e4a748-2192-161a-0510-9bf59fe950b5",
    "Node": "baz",
    "Address": "10.1.10.11",
    "Datacenter": "dc1",
    "TaggedAddresses": {
      "lan": "10.1.10.11",
      "wan": "10.1.10.11"
    },
    "Meta": {
      "instance_type": "t2.medium"
    }
  },
  {
    "ID": "8f246b77-f3e1-ff88-5b48-8ec93abf3e05",
    "Node": "foobar",
    "Address": "10.1.10.12",
    "Datacenter": "dc2",
    "TaggedAddresses": {
      "lan": "10.1.10.11",
      "wan": "10.1.10.12"
    },
    "Meta": {
      "instance_type": "t2.large"
    }
  }
]
```

.  
.  
.  
.  
.  
.  
.  


## List Services

This endpoint returns the services registered in a given datacenter.

| Method | Path                         | Produces                   |
| ------ | ---------------------------- | -------------------------- |
| `GET`  | `/catalog/services`          | `application/json`         |

The table below shows this endpoint's support for
[blocking queries](/api/index.html#blocking-queries),
[consistency modes](/api/index.html#consistency-modes), and
[required ACLs](/api/index.html#acls).

| Blocking Queries | Consistency Modes | ACL Required   |
| ---------------- | ----------------- | -------------- |
| `YES`            | `all`             | `service:read` |

### Parameters

- `dc` `(string: "")` - Specifies the datacenter to query. This will default to the datacenter of the agent being queried. This is specified as part of the URL as a query parameter.

- `node-meta` `(string: "")` - Specifies a desired node metadata key/value pair of the form `key:value`. This parameter can be specified multiple times, and will filter the results to nodes with the specified key/value pairs. This is specified as part of the URL as a query parameter.

### Sample Request

```text
$ curl \
    https://consul.rocks/v1/catalog/services
```

### Sample Response

```json
{
  "consul": [],
  "redis": [],
  "postgresql": [
    "primary",
    "secondary"
  ]
}
```

The keys are the service names, and the array values provide all known tags for
a given service.

.  
.  
.  
.  
.  
.  
.  


## List Nodes for Service

This endpoint returns the nodes providing a service in a given datacenter.

| Method | Path                         | Produces                   |
| ------ | ---------------------------- | -------------------------- |
| `GET`  | `/catalog/service/:service`  | `application/json`         |

The table below shows this endpoint's support for
[blocking queries](/api/index.html#blocking-queries),
[consistency modes](/api/index.html#consistency-modes), and
[required ACLs](/api/index.html#acls).

| Blocking Queries | Consistency Modes | ACL Required             |
| ---------------- | ----------------- | ------------------------ |
| `YES`            | `all`             | `node:read,service:read` |

### Parameters

- `service` `(string: <required>)` - Specifies the name of the service for which to list nodes. This is specified as part of the URL.

- `dc` `(string: "")` - Specifies the datacenter to query. This will default to the datacenter of the agent being queried. This is specified as part of the URL as a query parameter.

- `tag` `(string: "")` - Specifies the tag to filter on.

- `near` `(string: "")` - Specifies a node name to sort the node list in ascending order based on the estimated round trip time from that node. Passing `?near=_agent` will use the agent's node for the sort. This is specified as part of the URL as a query parameter.

- `node-meta` `(string: "")` - Specifies a desired node metadata key/value pair of the form `key:value`. This parameter can be specified multiple times, and will filter the results to nodes with the specified key/value pairs. This is specified as part of the URL as a query parameter.

### Sample Request

```text
$ curl  https://consul.rocks/v1/catalog/service/my-service
```

### Sample Response

```json
[
  {
    "ID": "40e4a748-2192-161a-0510-9bf59fe950b5",
    "Node": "foobar",
    "Address": "192.168.10.10",
    "Datacenter": "dc1",
    "TaggedAddresses": {
      "lan": "192.168.10.10",
      "wan": "10.0.10.10"
    },
    "NodeMeta": {
      "somekey": "somevalue"
    },
    "CreateIndex": 51,
    "ModifyIndex": 51,
    "ServiceAddress": "172.17.0.3",
    "ServiceEnableTagOverride": false,
    "ServiceID": "32a2a47f7992:nodea:5000",
    "ServiceName": "foobar",
    "ServicePort": 5000,
    "ServiceTags": [
      "tacos"
    ]
  }
]
```

- `Address` 注册服务所在节点的IP地址

- `Datacenter` 节点所在的数据中心

- `TaggedAddresses` agent 的 LAN 和 WAN 地址 

- `NodeMeta` is a list of user-defined metadata key/value pairs for the node

- `CreateIndex` is an internal index value representing when the service was created

- `ModifyIndex` is the last index that modified the service

- `Node` is the name of the Consul node on which the service is registered

- `ServiceAddress` is the IP address of the service host — if empty, node address should be used

- `ServiceEnableTagOverride` 指示在此服务上是否可以覆盖服务标记。

- `ServiceID` is a unique service instance identifier

- `ServiceName` is the name of the service

- `ServicePort` is the port number of the service

- `ServiceTags` is a list of tags for the service

.  
.  
.  
.  
.  
.  
.  


## List Services for Node

This endpoint returns the node's registered services.

| Method | Path                         | Produces                   |
| ------ | ---------------------------- | -------------------------- |
| `GET`  | `/catalog/node/:node`        | `application/json`         |

The table below shows this endpoint's support for blocking queries and
consistency modes.

The table below shows this endpoint's support for
[blocking queries](/api/index.html#blocking-queries),
[consistency modes](/api/index.html#consistency-modes), and
[required ACLs](/api/index.html#acls).

| Blocking Queries | Consistency Modes | ACL Required             |
| ---------------- | ----------------- | ------------------------ |
| `YES`            | `all`             | `node:read,service:read` |

### Parameters

- `node` `(string: <required>)` - Specifies the name of the node for which
  to list services. This is specified as part of the URL.

- `dc` `(string: "")` - Specifies the datacenter to query. This will default to
  the datacenter of the agent being queried. This is specified as part of the
  URL as a query parameter.

### Sample Request

```text
$ curl \
    https://consul.rocks/v1/catalog/node/my-node
```

### Sample Response

```json
{
  "Node": {
    "ID": "40e4a748-2192-161a-0510-9bf59fe950b5",
    "Node": "foobar",
    "Address": "10.1.10.12",
    "Datacenter": "dc1",
    "TaggedAddresses": {
      "lan": "10.1.10.12",
      "wan": "10.1.10.12"
    },
    "Meta": {
      "instance_type": "t2.medium"
    }
  },
  "Services": {
    "consul": {
      "ID": "consul",
      "Service": "consul",
      "Tags": null,
      "Port": 8300
    },
    "redis": {
      "ID": "redis",
      "Service": "redis",
      "Tags": [
        "v1"
      ],
      "Port": 8000
    }
  }
}
```