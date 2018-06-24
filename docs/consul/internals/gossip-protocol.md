
# Gossip 协议

Consul使用`Gossip`协议来管理集群成员和广播消息，这些功能通过使用[Serf库](https://www.serf.io/)来提供。
Serf中的`Gossip`协议基于“SWIM:可伸缩的弱一致性的传染式进程组成员协议”([SWIM: Scalable Weakly-consistent Infection-style Process Group Membership Protocol"](http://www.cs.cornell.edu/~asdas/research/dsn02-swim.pdf))，并进行了一些小小的改良。
关于Serf的协议[这里](https://www.serf.io/docs/internals/gossip.html)有更多细节。


> Gossip算法如其名，灵感来自办公室八卦，只要一个人八卦一下，在有限的时间内所有的人都会知道该八卦的信息。  
> Gossip的特点：在一个有界网络中，每个节点都随机地与其他节点通信，经过一番杂乱无章的通信，最终所有节点的状态都会达成一致  
> Gossip是一个带冗余的容错算法，更进一步，Gossip是一个**最终一致性算法**  
> 因为Gossip不要求节点知道所有其他节点，因此又具有**去中心化**的特点，节点之间完全对等，不需要任何的中心节点
> 
> Gossip的缺点也很明显，冗余通信会对网路带宽、CPU资源造成很大的负载，而这些负载又受限于通信频率，该频率又影响着算法收敛的速度
>
> [Gossip算法](http://blog.csdn.net/chen77716/article/details/6275762)  
> [分布环境下的Gossip算法综述](http://www.doc88.com/p-919959141193.html)

> [!]本文将介绍Consul内部的技术细节，对于实际操作和使用Consul，你不需要知道这些细节。这些细节记录在这里为那些希望了解它们的人，小伙伴们不必再去探索源代码。

## Consul中的`Gossip`

Consul使用两个不同的`Gossip`池，分别是`LAN池`和`WAN池`。

Conusl运行的`每个数据中心有一个包含所有数据中心成员的`LAN池，包括所有的`Client`和`Server`。LAN池用于以下目的：
- 成员信息使Client可以自动发现Server，减少需要的配置数量；
- 分布式的故障检测使故障检测工作分享到整个集群，代替集中在某些Server之上；
- 最后，`Gossip`池提供可靠和快速的事件广播，比如leader选举事件。

WAN池是全局唯一的，不管是哪个数据中心，所有的Server都参与其中。WAN池提供的成员信息使Server可移植性跨数据中心的请求。集成的故障检测使Consul可以优雅的处理整个数据中心的失连，或者只是远程数据中心的某个Server。

所有这些功能都利用`Serf`来提供。它作为一个嵌入式的库来提供这些功能。从用户的视角来看，这并不重要，因为这些概念已经被Consul掩盖了。
然而作为一个开发者，理解这个库如何被利用的是有用的。

## Lifeguard 增强

SWIM假设本地节点是健康的，因为软实时数据包处理（soft real-time processing of packets）是可能会出现的的。
然而，本地节点在经历CPU或网络资源耗尽的情况下，这个假设就会被打破。
结果是，serfHealth检查状态会出现偶尔的波动，产生误报警，增加遥测噪音，很容易导致整个集群浪费CPU和网络资源去诊断一个可能根本不存在的故障。


Lifeguard 通过对SWIM进行新的增强，完全的解决了这个问题。请看Serf的`Gossip`协议指南中关于Lifeguard的部分了解更多细节。

- [Making Gossip More Robust with Lifeguard ](https://www.hashicorp.com/blog/making-gossip-more-robust-with-lifeguard/?_ga=2.233414878.1125839144.1516002498-323234591.1516002498)
- [Lifeguard : SWIM-ing with Situational Awareness](https://arxiv.org/abs/1707.00788)
- [Serf gossip protocol guide](https://www.serf.io/docs/internals/gossip.html#lifeguard)

