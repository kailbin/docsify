
# 网络坐标(Network Coordinates)

Consul 使用 [network tomography(断层摄影)](https://en.wikipedia.org/wiki/Network_tomography) 系统为集群中的节点计算网络坐标。
有了这些坐标，就可以使用非常简单的方式，**估算在任意两个网络节点之间往返时间**。
它可以应用于很多场景，例如查找要请求节点最近的服务节点，或者故障转移到最近的数据中心。


该功能都基于[Serf库](https://www.serf.io/)进行开发。
Serf的网络断层技术是基于 ["Vivaldi: A Decentralized Network Coordinate System"](http://www.cs.ucsb.edu/~ravenben/classes/276/papers/vivaldi-sigcomm04.pdf),
进行的改进. 详情查看 [Serf's network coordinates here](https://www.serf.io/docs/internals/coordinates.html).

> [!]本文将介绍Consul内部的技术细节，对于实际操作和使用Consul，你不需要知道这些细节。这些细节记录在这里为那些希望了解它们的人，小伙伴们不必再去探索源代码。


## Network Coordinates in Consul

在 Consul 内部 网络坐标（network coordinates）体现在一下几方面：

* [`consul rtt`](cli/rtt.md) 命令可用于查询任意两个节点之间的网络往返时间.
* [Catalog API](api/catalog.md) 和 [Health API](api/health.html) 可以根据给定节点的网络往返时间，使用`?near=`参数来对查询结果进行排序.
* [Prepared queries API](/api/query.html) 可以根据网络往返时间，自动将服务故障转移到其他Consul数据中心.  示例 ：[Geo Failover](guides/geo-failover.md) .
* [Coordinate API](api/coordinate.html) 可以查询原始网络坐标以便在其他应用程序中使用.

`Consul`使用`Serf`管理两个不同的`Gossip`池，一个是针对数据中心内部成员的LAN，另一个是由所有数据中心的 Consul Server 组成的WAN。
需要注意的是，**网络坐标在这两个池之间是不兼容的**。
LAN坐标仅在与其他LAN坐标的对比才有意义，而WAN坐标仅与其他WAN坐标对比才有意义。

## Working with Coordinates

一旦有了坐标，计算任意两个节点之间的网络往返时间就很简单了
这是一个坐标样本，通过[Coordinate API](api/coordinate.md)获取.

```
    "Coord": {
        "Adjustment": 0.1,
        "Error": 1.5,
        "Height": 0.02,
        "Vec": [0.34,0.68,0.003,0.01,0.05,0.1,0.34,0.06]
    }
```

所有的值都是以秒为单位的浮点数，除了不是用于距离计算的`Error`项。

这里有一个完整的例子，说明了如何计算两个坐标之间的距离:

```
import (
    "math"
    "time"

    "github.com/hashicorp/serf/coordinate"
)

func dist(a *coordinate.Coordinate, b *coordinate.Coordinate) time.Duration {
    // Coordinates will always have the same dimensionality, so this is
    // just a sanity check.
    if len(a.Vec) != len(b.Vec) {
        panic("dimensions aren't compatible")
    }

    // Calculate the Euclidean distance plus the heights.
    sumsq := 0.0
    for i := 0; i < len(a.Vec); i++ {
        diff := a.Vec[i] - b.Vec[i]
        sumsq += diff * diff
    }
    rtt := math.Sqrt(sumsq) + a.Height + b.Height

    // Apply the adjustment components, guarding against negatives.
    adjusted := rtt + a.Adjustment + b.Adjustment
    if adjusted > 0.0 {
        rtt = adjusted
    }

    // Go's times are natively nanoseconds, so we convert from seconds.
    const secondsToNanoseconds = 1.0e9
    return time.Duration(rtt * secondsToNanoseconds)
}
```