# What is Consul?

Consul有多个组件，但总体而言，它在基础设施中 充当**服务发现**和**服务配置**的角色。其提供了几个关键功能：

- **服务发现(Service Discovery)**：客户端可以使用 `Consul` 来发现给定服务的提供者。使用`DNS`或`HTTP`，应用程序可以轻松找到他们所依赖的服务。
- **健康检查(Health Checking)**：`Consul` 客户端可以提供任意数量的健康检查，或者跟给定的服务，或与本地节点相关联。运维人员可以通过此信息来监控群集运行状况，并通过服务发现组件将流量从不健康的主机中引导出去。
- **键值对存储(KV Store)**：应用程序可以使用`Consul`的 键/值存储来实现 动态配置、协调、选举  等等。通过简单的HTTP API即可使用。
- **多数据中心(Multi Datacenter)**：`Consul` 支持多个数据中心。这意味着`Consul`的用户不必担心多个区域扩展。

> - **Service Discovery**: Clients of Consul can provide a service, such as api or mysql, and other clients can use Consul to discover providers of a given service. Using either DNS or HTTP, applications can easily find the services they depend upon.
> - **Health Checking**: Consul clients can provide any number of health checks, either associated with a given service ("is the webserver returning 200 OK"), or with the local node ("is memory utilization below 90%"). This information can be used by an operator to monitor cluster health, and it is used by the service discovery components to route traffic away from unhealthy hosts.
> - **KV Store**: Applications can make use of Consul's hierarchical key/value store for any number of purposes, including dynamic configuration, feature flagging, coordination, leader election, and more. The simple HTTP API makes it easy to use.
> - **Multi Datacenter**: Consul supports multiple datacenters out of the box. This means users of Consul do not have to worry about building additional layers of abstraction to grow to multiple regions.



# Consul 基础架构（Basic Architecture of Consul）

Consul是一个分布式，高度可用的系统。本节将介绍基础知识，故意省略一些不必要的细节，以便您快速了解Consul的工作原理。有关更多详细信息，请参阅 [ in-depth architecture overview](https://www.consul.io/docs/internals/architecture.html)。

向Consul提供服务的每个节点都运行一个 Consul agent。`发现其他服务`或 通过`getting/setting`操作`key/value` 数据不需要运行代理。agent 负责健康检查节点上的服务以及节点本身。

Agents 与一个或多个 Consul Server 进行通讯。 Consul Server 是数据存储和复制的地方。服务器会自己选出 一个 leader。虽然Consul可以在一台服务器上运行，但推荐使用`3到5台`来避免故障导致数据丢失的情况。每个数据中心都建议使用一组Consul服务器。

需要发现其他服务或节点的基础架构组件可以查询任何Consul服务器或任何Consul代理。代理自动将查询转发到 Consul Server 端。

每个数据中心都运行 Consul Server 集群。当发生跨数据中心服务发现或配置请求时，本地 Consul Server 将请求转发给远程数据中心并返回结果。

