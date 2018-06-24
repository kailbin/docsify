
# Consul vs. Serf

[Serf](https://www.serf.io)是一个节点发现和协调工具，它是基于最终一致性的 Gossip协议 模型构建的，不需要集中式部署服务器。
它提供一系列的功能，包括`组成员`、`故障检测`、`事件广播`，以及一个`查询`机制。
然而，Serf不提供任何高级功能，比如服务发现、健康检查或者键值存储。Consul是一个提供所有这个功能的完整系统。


实际上Consul内部使用的[Gossip协议](internals/gossip-protocol.md)是由Serf类库支持的：Consul在其`成员管理`和`故障检测`功能之上来构建服务发现。
相比**Serf的发现功能在节点级别**，Consul提供**服务和节点级别的发现**功能。



Serf提供的健康检查非常低级，仅标识Agent是否活着。
Consul在此基础上提供了一个丰富的健康检查系统，可以处理任意主机和服务级别的活跃度检查；健康检查有一个集成面板，运维人员可以容易的查询，并获取对集群的洞察力。


Serf提供的成员管理在节点级别；Consul聚焦在服务级别，单个节点映射到多个服务。
Serf可以使用标签进行模仿，但是它有很多限制，并且不能提供有用的查询接口。
Consul还使用一个强一致性的Catalog，而Serf仅是最终一致性。


除了服务级别抽象和健康检查的提升，Consul还提供一个键值存储，并支持多数据中心。
Serf可以在WAN环境下运行，但是会降低性能。
Consul使用[多流言池](internals/consul-architecture.md)，所以使用它连接WAN环境的多个数据中心的情况下，Serf在LAN环境的性能可以保持。


Consul用法相对固定，而Serf是一个更加灵活和通用的工具。
在[CAP](https://en.wikipedia.org/wiki/CAP_theorem)理论中，**Consul使用CP架构，强调一致性超过可用性**。
**Serf是一个AP系统，牺牲一致性保证可用性**。
这意味着几乎所有的情况下如果中央服务器不能达到法定数目，Consul将不能运行，而Serf将继续运作。
