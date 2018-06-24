
# DNS Caching


DNS是Consult提供的一个主要接口。使用DNS是一种不需要高度集成就可以将Consul集成到已有的基础设施中的简单方式。




默认情况下Consul服务的所有DNS查询数据不会长期保留（0 Time To Live）,这防止了所有缓存。
每个DNS查询都是重新获取的，优势是可以提供最及时的DNS信息。
但是，这样会增加延迟并且可能会降低整个集群查询的吞吐量。


为此，Consul提供了一些调优参数用来制定DNS查询。

## 读取陈旧数据（Stale Reads）


读取陈旧数据可以用来降低DNS查询时的延迟并提高吞吐量。
默认情况下，所有的查询都由[一个leader节点](internals/consensus-protocol.md)提供服务。
这样可以保持强一致性但受限于单一节点的吞吐量限制。
设置一个陈旧数据的读取可以允许任意的Consul server 提供查询，但不是leader节点提供的数据有可能是过期的数据。
通过允许数据稍微的陈旧，我们得到读取时的水平扩展。
所有的Consul服务端（Server）可以处理请求，所以我们可以通过增加集群中服务端的数量来增加吞吐量。


用来[配置](cli/agent/options.md)陈旧数据读取的配置是`dns_config.allow_stale`，这里需要设置为启用状态，并且`dns_config.max_stale`可以配置数据陈旧的最长时间。


从Consul的0.7版本开始，`allow_stale`默认设置为开启状态，并且`max_stale`的默认配置时间是5秒，
这意味着我们从任意的 Consul server 中读取到的数据都与leader中获取到的数据存在5秒的偏差。


## 拒绝响应缓存(Negative Response Caching)

DNS缓存有时候会拒绝响应，由于一个存在的服务没有处于健康的状态，Consul会返回了一个“not found”格式的响应。
这里意味着在执行过程中出现缓存拒绝响应，可能意味着服务出现了“未能正常运行”的时候，通过DNS进行服务发现时 他们实际获取的数据还没有及时更正。


一个简单的例子，操作系统默认会缓存拒绝的响应15分钟。DNS转发器也可能会缓存拒绝的响应造成同样的影响。
为了避免这个问题，为你的客户端操作系统和任何介于客户端与Consul之间的DNS转发器，应进行默认的拒绝响应缓存的检查，并设置适当的缓存时间。
在很多情况下简单做法就是关闭拒绝响应缓存，当服务再次变成可用时应用最好快速的恢复。


## TTL Values

TTL的值可以用来设置Consul用来缓存DNS结果的时间。
较大的TTL值可以以越来越多的陈旧数据为代价缩短通过Consul服务的查找次数和客户端的查找速度。
默认情况下，所有的TTL都被设置为0，防止进行缓存。


开启节点查找的缓存（如“foo.node.consul”），我们可以设置`dns_config.node_ttl`的值，这里可以设置10s作为一个例子，
这样所有的节点查找将会提供一个10秒TTL结果 。

服务的TTL可以更细粒度的使用，你可以为每个服务设置多个TTL，使用通配符设置一个默认的TTL.这里使用`dns_config.service_ttl`匹配。

例如，一个`dns_config`，这里为服务提供了**通配符TTL**和**指定具体服务TTL**格式如下：

``` javascript
{
  "dns_config": {
    "service_ttl": {
      "*": "5s",
      "web": "30s"
    }
  }
}
```


这个设置对所有查找`web.service.consul`应用30秒的TTL,同时对`db.service.consul` 或者 `api.service.consul`将会通过通配符应用5秒的TTL。

[Prepared Queries](api/prepared-query.md)提供另外一种方式来管理TTL。
它允许通过一个查询来定义TTL,并且通过更新操作快速修改。
如果一个TTL没有为prepared query配置，它会退回到在Consul agent中如上文描述的指定服务的配置方式，
如果在Consul agent中没有TTL为服务指定,TTL会最终设置为0。

# Read More

> 来自 [Consul中文翻译计划 - DNS缓存](http://consul.la/docs/guides/dns-cache)