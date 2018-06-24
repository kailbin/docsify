
# Geo Failover

Within a datacenter, Consul provides automatic failover for services by omitting failed service instances from DNS lookups, and by providing service health information in APIs. 
在数据中心内，领事通过从DNS查询中省略失败的服务实例，以及在AP中提供服务健康信息，为服务提供自动故障转移。
When there are no more instances of a service available in the local datacenter, it can be challenging to implement failover policies to other datacenters because typically that logic would need to be written into each application.
当本地数据中心中没有可用的服务实例时，将故障转移策略应用到其他数据中心是很困难的，因为通常需要将逻辑写到每个应用程序中。



Fortunately, Consul has a [prepared query](/api/query.html) capability that lets users define failover policies in a centralized way. 
幸运的是，领事有一个[准备好的查询](/ api /查询)。允许用户集中定义故障转移策略的功能。
It's easy to expose these to applications using Consul's DNS interface and it's also available to applications that consume Consul's APIs.
使用领事的DNS接口将它们暴露在应用程序中是很容易的，而且它也可以应用于使用领事的AP的应用程序。
These policies range from fully static lists of alternate datacenters to fully dynamic policies that make use of Consul's [network coordinate](/docs/internals/coordinates.html) subsystem to automatically determine the next best datacenter to fail over to based on network round trip time.
这些策略包括从完全静态的备选数据中心列表到利用领事的[网络坐标](/ docs / internals /坐标)的完全动态策略。基于网络往返时间，自动确定下一个最佳数据中心故障的子系统。
Prepared queries can be made with policies specific to certain services and prepared query templates allow one policy to apply to many, or even all services, with just a small number of templates.
准备好的查询可以使用特定于某些服务的策略进行，并且准备好的查询模板允许一个策略应用于许多甚至所有的服务，只有一小部分模板。


This guide shows how to build geo failover policies using prepared queries through a set of examples.
本指南展示了如何通过一组示例来构建geo故障转移策略。


## Prepared Queries

Prepared queries are objects that are defined at the datacenter level, similar to the values in Consul's KV store. 
准备好的查询是在数据中心级别定义的对象，类似于领事的KV存储中的值。
They are created once and then invoked by applications to perform the query and get the latest results.
它们被创建一次，然后由应用程序调用来执行查询并获得最新的结果。

Here's an example request to create a prepared query:
下面是创建准备好的查询的示例请求:

```
$ curl -X POST -d \
'{
  "Name": "api",
  "Service": {
    "Service": "api",
    "Tags": ["v1.2.3"]
  }
}' http://127.0.0.1:8500/v1/query

{"ID":"fe3b8d40-0ee0-8783-6cc2-ab1aa9bb16c1"}
```

This creates a prepared query called "api" that does a lookup for all instances of the "api" service with the tag "v1.2.3". 
这就创建了一个名为“api”的准备查询，该查询为“api”服务的所有实例进行查找，标签为“v1.2.3”。

This policy could be used to control which version of a "api" applications should be using in a centralized way. 
该策略可用于控制哪些版本的“api”应用程序应该以集中的方式使用。
By [updating this prepared query](/api/query.html#update-prepared-query) to look for the tag "v1.2.4" applications could start to find the newer version of the service without having to reconfigure anything.
通过[更新这个准备好的查询](/ api /查询)。html #更新-准备-查询)寻找标签“v1.2.4”应用程序可以开始查找新版本的服务，而无需重新配置任何东西。


Applications can make use of this query in two ways. 
应用程序可以通过两种方式使用这个查询。
Since we gave the prepared query a name, they can simply do a DNS lookup for "api.query.consul" instead of "api.service.consul". 
由于我们给准备好的查询起了一个名字，他们可以简单地为“api”做一个DNS查找。查询。领事"而不是" api "。服务。高”。
Now with the prepared query, there's the additional filter policy working behind the scenes that the application doesn't have to know about.
 现在，在已准备好的查询中，在应用程序不需要知道的场景后面有额外的筛选策略。
Queries can also be executed using the [prepared query execute API](/api/query.html#execute-prepared-query) for applications that integrate with Consul's APIs directly.
还可以使用[编写的查询执行API](/ API /查询)执行查询。与领事的AP相结合的应用程序的html #执行-准备-查询。

## Failover Policies

Using the techniques in this section we will develop prepared queries with failover policies where simply changing application configurations to look up "api.query.consul" instead of "api.service.consul" via DNS will result in automatic geo failover to the next closest federated Consul datacenters, in order of increasing network round trip time.

Failover is just another policy choice for a prepared query, it works in the same manner as the previous example and is similarly transparent to applications. The failover policy is configured using the `Failover` structure, which contains two fields, both of which are optional, and determine what happens if no healthy nodes are available in the local datacenter when the query is executed.

- `NearestN` `(int: 0)` - Specifies that the query will be forwarded to up to `NearestN` other datacenters based on their estimated network round trip time using [network coordinates](/docs/internals/coordinates.html).

- `Datacenters` `(array<string>: nil)` - Specifies a fixed list of remote datacenters to forward the query to if there are no healthy nodes in the local datacenter. Datacenters are queried in the order given in the list.

The following sections show examples using these fields to implement different geo failover policies.

### Static Policy

A static failover policy includes a fixes list of datacenters to contact once there are no healthy instances in the local datacenter.

Here's the example from the introduction, expanded with a static failover policy:

```
$ curl \
    --request POST \
    --data \
'{
  "Name": "api",
  "Service": {
    "Service": "api",
    "Tags": ["v1.2.3"],
    "Failover": {
      "Datacenters": ["dc1", "dc2"]
    }
  }
}' http://127.0.0.1:8500/v1/query

{"ID":"fe3b8d40-0ee0-8783-6cc2-ab1aa9bb16c1"}
```

When this query is executed, such as with a DNS lookup to "api.query.consul", the following actions will occur:

1. Consul servers in the local datacenter will attempt to find healthy instances of the "api" service with the required tag.
2. If none are available locally, the Consul servers will make an RPC request to the Consul servers in "dc1" to perform the query there.
3. If none are available in "dc1", then an RPC will be made to the Consul servers in "dc2" to perform the query there.
4. Finally an error will be returned if none of these datacenters had any instances available.

### Dynamic Policy

In a complex federated environment with many Consul datacenters, it can be cumbersome to set static failover policies, so Consul offers a dynamic option based on Consul's [network coordinate](/docs/internals/coordinates.html) subsystem. Consul continuously maintains an estimate of the network round trip time from the local datacenter to the servers other datacenters it is federated with. Each server uses the median round trip time from itself to the servers in the remote datacenter. This means that failover can simply try other remote datacenters in order of increasing network round trip time, and if datacenters come and go, or experience network issues, this order will adjust automatically.

Here's the example from the introduction, expanded with a dynamic failover policy:

```
$ curl \
    --request POST \
    --data \
'{
  "Name": "api",
  "Service": {
    "Service": "api",
    "Tags": ["v1.2.3"],
    "Failover": {
      "NearestN": 2
    }
  }
}' http://127.0.0.1:8500/v1/query

{"ID":"fe3b8d40-0ee0-8783-6cc2-ab1aa9bb16c1"}
```

This query is resolved in a similar fashion to the previous example, except the choice of "dc1" or "dc2", or possibly some other datacenter, is made automatically.

### Hybrid Policy

It is possible to combine `Datacenters` and `NearestN` in the same policy. The `NearestN` queries will be performed first, followed by the list given by `Datacenters`. A given datacenter will only be queried one time during a failover, even if it is selected by both `NearestN` and is listed in `Datacenters`. This is useful for allowing a limited number of round trip-based attempts, followed by a static configuration for some known datacenter to failover to.

## Templates

For datacenters with many services, it can be cumbersome to define a prepared query to apply a geo failover policy for each service. Consul provides a [prepared query template](/api/query.html#prepared-query-templates) capability to allow one prepared query to apply to many, and even all, services.

Here's an example request to create a prepared query template that applies a dynamic geo failover policy to all services:

```
$ curl \
    --request POST \
    --data \
'{
  "Name": "",
  "Template": {
    "Type": "name_prefix_match"
  },
  "Service": {
    "Service": "${name.full}",
    "Failover": {
      "NearestN": 2
    }
  }
}' http://127.0.0.1:8500/v1/query

{"ID":"fe3b8d40-0ee0-8783-6cc2-ab1aa9bb16c1"}
```

Templates can match on prefixes or use full regular expressions to determine which services they match. In this case, we've chosen the `name_prefix_match` type and given it an empty name, which means that it will match any service. If multiple queries are registered, the most specific one will be selected, so it's possible to have a template like this as a catch-all, and then apply more specific policies to certain services.

With this one prepared query template in place, simply changing application configurations to look up "api.query.consul" instead of "api.service.consul" via DNS will result in automatic geo failover to the next closest federated Consul datacenters, in order of increasing network round trip time.