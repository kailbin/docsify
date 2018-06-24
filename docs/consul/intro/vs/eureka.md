


`Eureka`是一个服务发现工具。架构主要是 client/server 模式，每个数据中心有一套Eureka服务器，通常每个可用区域一个。
通常`Eureka`的客户使用嵌入式SDK来注册和发现服务。对于非本地集成的客户端，使用像Ribbon这样的sidecar可以通过Eureka透明地发现服务。


`Eureka` 提供了一种弱一致的服务视图，通过尽可能的相互复制数据来实现。
当客户端向服务器注册时，该服务器将尝试复制到其他服务器，但不提供保证。
服务注册有一个短暂的生存时间（TTL），要求客户端与服务器保持心跳。
不健康的服务或节点将停止心跳，导致它们超时并从注册表中删除。
这个简化的模型允许简单的群集管理和高扩展性。



Consul 提供了一套超级功能，包括更丰富的健康检查，key/value 存储，和多数据中心。 
Consul 每个数据中心都需要一套服务器与每个客户的代理，类似于使用 Ribbon。 
**Consul agent 允许大多数应用程序忽略Consul的存在，通过配置文件进行注册（需要`Consul agent`命令），通过 DNS 均衡器 进行服务发现。**

> 感觉应该可以这样做： Consul 随 Docker 启动，启动的时候把整个Docker的服务地址注册到 Consul，服务间调用的时候，通过名字服务名（域名）进行调用
> 对于 JVM 应用来说们就需要解决 DNS 缓存、超时时间等问题



Consul 使用 [Raft 协议](internals/consensus-protocol.md) 进行服务间的状态复制，提供了强大的一致性保证。 
Consul 支持一组丰富的健康检查组件，包括TCP、HTTP、兼容Nagios/Sensu脚本，或者类似于Eureka的TTL。
Client节点参与 [基于gossip的健康检查](internals/gossip-protocol.md)。它承担了健康检查的工作，不像集中的心跳会成为了可伸缩性的挑战。
注册的服务被路由到所选的 Consul Leader，这使得他们在默认情况下具有很强的一致性。
如果客户端允许读取过期数据，那么所有服务器都可以处理请求，从而允许像Eureka这样的线性可伸缩性。

> 基于gossip的健康检查???


Consul 的强烈一致性意味着它可以被用作**领导选举**和**集群协调的锁定服务**。
Eureka并没有提供类似的保证，而且通常需要管理员为需要进行协调或有更强的一致性需求的服务运行管理。


Consul提供了支持面向服务的体系架构所需的工具包。
这包括**服务发现**，还包括丰富的**健康检查**，**锁**，**Key/Value存储**，**多数据中心**，**事件系统**和**ACL**。
Consul和consul-template和envconsul等工具的生态系统都尽量减少集成所需的应用程序更改，以避免需要通过SDK进行本地集成。
Eureka 是一个更大的Netflix OSS套件的一部分，该套件需要和应用程序紧密集成。
因此，Eureka只解决了一小部分问题，需要ZooKeeper等其他工具配合一起使用。


> Eureka2.0 改进 [Eureka 2.0 Motivations](https://github.com/Netflix/eureka/wiki/Eureka-2.0-Motivations)