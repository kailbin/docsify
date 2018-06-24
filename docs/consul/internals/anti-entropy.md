
# Anti-Entropy

Consul 使用先进的方法 维护服务和健康信息。
本页面详细介绍了**如何注册服务和检查**、**如何填充目录(catalog)** 以及**如何在更新健康状态信息**。


> [!] 本文将介绍Consul内部的技术细节，对于实际操作和使用Consul，你不需要知道这些细节。这些细节记录在这里为那些希望了解它们的人，小伙伴们不必再去探索源代码。


### Components

首先要了解服务和健康检查中涉及的部件: [agent](#agent) 和 [catalog](#catalog)。
 



<a name="agent"></a>
#### Agent

每个Consul agent 维护自己的一套服务和健康检查信息。
agents 负责执行他们自己的健康检查和更新他们的本地状态

在agent上下文中提供的服务和检查具有丰富的配置选项。
这是因为`agent`负责通过使用[health checks](cli/agent/checks.md)生成有关其服务和健康的信息。



<a name="catalog"></a>
#### Catalog

**Consul 的服务发现由`catalog`支持**。
catalog是由 agent 提交的聚合信息形成的。
`catalog` 维护集群的高级视图，包括**哪些服务可用**，**哪些节点运行这些服务**、**健康信息**等等。
`catalog` 通过 Consul 提供的各种接口(包括`DNS`和`HTTP`)公开这些信息。


与 agent 相比，在`catalog` 上下文中的服务和检查信息具有更有限的字段集。
这是因为`catalog`只负责记录和返回有关服务、节点和健康的信息。

`catalog`仅由服务器节点维护。
这是因为`catalog`是通过[Raft log](internals/consensus-protocol.md)复制的。为集群提供一个统一的、一致的视图。




<a name="anti-entropy"></a>
### 反熵（Anti-Entropy）

熵是系统越来越无序的趋势。
领事的反熵（anti-entropy）机制是为了对抗这种倾向，即使是一些组件的故障，也保持集群的状态的一致。


正如上面所讨论的，Consul 在全局服务目录和 agent的本地状态之间有明确的区分。
反熵机制协调了这两种世界观: 反熵是本地 agent 状态和`catalog`的同步。


例如，当用户通过 `agent` 注册了一个新服务或检查时， `agent` 将向`catalog`通知该信息。
类似地，当从`agent`中删除一个服务时，它也会被从`catalog`中删除。


反熵也用于更新可用性信息。
当`agent`运行他们的健康检查时，他们的状态可能会改变，在这种情况下，他们的新状态被同步到`catalog`中。
通过使用这些信息，`catalog`可以根据其可用性对其节点和服务的查询进行智能响应。


在此同步过程中，`catalog`还会检查是否正确。
`catalog`会自动删除`agent`中没有的服务，以使`catalog`适当的反映该出`agent`的服务集和健康信息。
Consul 视`agent`的状态为权威信息;如果`agent`和`catalog`视图之间存在任何差异，则始终以`agent`本地视图为准。


### 同步周期（Periodic Synchronization）

除了在`agent`发生更改时进行运作之外，反熵也是一个长期运行的过程，它会周期性地唤醒同步服务并检查`catalog`的状态。
这将确保`catalog`与`agent`的状态一致。
这也使Consul在完全数据丢失的情况下可以重新填充服务`catalog`。


为了避免饱和，周期性的反熵运行的时间会根据进去的大小而变化。
下表定义了集群大小和同步间隔之间的关系:

| Cluster Size（集群大小）  | Periodic Sync Interval（定期同步的间隔）  |
| :------: | :------: |
|  1 - 128   |   1 minute  |
|   129 - 256  |  2 minutes   |
|   257 - 512  |  3 minutes   |
|   513 - 1024  |  4 minutes   |
|   ...  |   ...  |


上面的间隔是近似值。
每个 Consul `agent`将在间隔窗口中选择一个随机的开始时间，以避免羊群效应。


### 最大努力同步（Best-effort sync）

反熵在许多情况下可能会失败，包括`agent`的错误配置或其操作环境， I/O问题(磁盘满了、文件系统权限等)、网络问题(`agent`无法与服务器通信)，以及其他问题。
因此，`agent`试图以最大努力的方式进行同步。


如果在反熵运行期间遇到错误，将记录错误并继续运行。
反熵机制是周期性运行的，可以自动从这些瞬时故障中恢复。


### 启用标签覆盖（EnableTagOverride）

服务注册的同步可以被部分修改，`agent`可以更改服务的便签（tag）。
在外部监视服务就是标签信息的真实来源的情况下，这一点非常有用。
例如，Redis数据库及其监视服务Redis Sentinel有这种关系。
Redis实例对它们的大部分配置负责，但是哨兵决定Redis实例是主还是次要的。
使用Consul服务配置项[EnableTagOverride](cli/agent/services.md)您可以指示执行Redis数据库的领事`agent`在反熵同步期间不更新标记。
更多信息见[Services](cli/agent/services.md)页面。