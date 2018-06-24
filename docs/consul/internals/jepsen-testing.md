# Jepsen Testing

[Jepsen](http://aphyr.com/posts/281-call-me-maybe-carly-rae-jepsen-and-the-perils-of-network-partitions) 是 Kyle Kingsbury 写的一个工具。
它可以用来**测试分布式系统的中的分区容忍度**。
创建网络分区，同时对系统进行随机操作。
分析结果会发现系统是否违反了它声称拥有的一致性属性。

作为Consul测试的一部分，我们运行了一个Jepsen测试来确定是否可以发现一致性问题。

在我们的测试中，Consul优雅地从分区中恢复，而没有出现任何一致性问题。


测试结果详见官方文档 ： https://www.consul.io/docs/internals/jepsen.html