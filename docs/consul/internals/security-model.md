
# Security Model

Consul 依赖于轻量级的`gossip`机制和RPC系统来提供各种功能。这两种系统都有不同的安全机制，这些机制来自于它们自己的设计。
然而，领事的安全机制有一个共同的目标:提供[机密、完整和鉴权（confidentiality, integrity, and authentication）](https://en.wikipedia.org/wiki/Information_security)安全系统。

 [gossip protocol](/docs/internals/gossip.html) 由[Serf](https://www.serf.io/)提供,它使用是对称加密或共享密钥的加密系统。

关于安全的更多细节请查看 [Serf](https://www.serf.io/docs/internals/security.html).
有关如何启用Serf在Consul中的Gossip加密的细节，请参阅[encryption 文档](cli/agent/encryption.md)。

The RPC system supports using end-to-end TLS with optional client authentication.
RPC系统支持使用 端到端（end-to-end） TLS与可选的客户端身份验证。
[TLS](https://en.wikipedia.org/wiki/Transport_Layer_Security) 是一种广泛运用的非对称密码系统，是网络安全的基础。

这意味着Consul通信可以防止窃听、篡改和欺骗。
这使得在不受信任的网络(如 EC2和其他共享主机提供商)上运行Consul成为可能

> [!]本文将介绍Consul内部的技术细节，对于实际操作和使用Consul，你不需要知道这些细节。这些细节记录在这里为那些希望了解它们的人，小伙伴们不必再去探索源代码。


## 威胁模型（Threat Model）

以下是威胁模型的各个部分：

* 非成员访问数据。
* 由于恶意消息而导致的集群状态操纵
* 由于恶意消息而产生的虚假数据
* 篡改导致的状态环混乱
* 拒绝服务 保护节点

此外，我们认识到攻击者可以在很长一段时间内观察网络流量，从而推断集群成员。
Consul使用的gossip机制依赖于向随机成员发送消息，因此攻击者可以记录所有目的地并确定集群的所有成员。

当设计一个系统的安全性时，你应该设计它来符合威胁模型。
我们的目标不是保护顶级机密数据，而是提供一个“合理（reasonable）”的安全级别，使得攻击者投入的大量资源付之东流。

## Network Ports

配置网络规则以支持Consul请查看[Ports Used](cli/agent/options.md),
文档中列出了 Consul 使用的端口 和 他们的详细用途
