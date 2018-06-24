
# DNS 转发（Forwarding DNS）


默认情况下，DNS通过**53端口**提供服务。在多数操作系统中，这都需要很高的系统权限。
与其使用管理或根帐户运行Consul，还可以将适当的查询转发给Consul，在一个非特权端口上运行，从另一个DNS服务器或端口重定向。



在这篇指南中，我们将通过
[BIND](https://www.isc.org/downloads/bind/)、
[dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html)、
[Unbound](https://www.unbound.net/)和[iptables](http://www.netfilter.org/)进行示范。
出于简单的目的，在这个例子中BIND和Consul运行在同一台机器上。
对于iptables的规则必须设置在与Consul实例统一的宿主上并且依赖的域名解析文件不能是同一个宿主不然跳转会拦截通信。
对于iptables，规则必须设置在与Consul实例相同的主机上，而中继主机不应位于同一主机上，否则跳转会被拦截。



值得一提的是，默认情况下，Consul不提供不包含`.consul.`域名的DNS记录的解析,除非在配置文件中已经配置了[recursors](cli/agent/options.md)。
作为一个例子说明这个配置是怎么改变Consul的行为的，假设一个Consul的DNS可以对一个没有指向`.consul`的顶级域名的的权威名称（`CNAME`）进行应答。
通常这个DNS只会应答包含权威名称（`CNAME`）的记录。作为比较，当`recursors`被设置并且 `upstream`解析器运行正常的话，
Consul会尝试解析所有的权威名称（`CNAME`）和包含别名的记录 (说明： IPV4主机地址（`A`）,IPV6主机地址（`AAAA`）,解析IP的指针（`PTR`）)用于它的DNS应答中。


### 绑定设置（BIND Setup）

首先，你必须禁用`DNSSEC`，这样Consul和BIND才何以进行通信，这里是一个对应配置的例子

``` text
options {
  listen-on port 53 { 127.0.0.1; };
  listen-on-v6 port 53 { ::1; };
  directory       "/var/named";
  dump-file       "/var/named/data/cache_dump.db";
  statistics-file "/var/named/data/named_stats.txt";
  memstatistics-file "/var/named/data/named_mem_stats.txt";
  allow-query     { localhost; };
  recursion yes;

  dnssec-enable no;
  dnssec-validation no;

  /* Path to ISC DLV key */
  bindkeys-file "/etc/named.iscdlv.key";

  managed-keys-directory "/var/named/dynamic";
};

include "/etc/named/consul.conf";
```

### Zone File

接下来我们在`consul.conf`文件中为我们的Consul建立一个区域来管理记录

``` text
zone "consul" IN {
  type forward;
  forward only;
  forwarders { 127.0.0.1 port 8600; };
};
```

这里我们假设Consul正在以默认配置运行并且正在8600端口端口提供DNS服务。

### Dnsmasq Setup

Dnsmasq通常通过`dnsmasq.conf`文件或者在`/etc/dnsmasq.d`下的一些文件进行配置，
在Dnsmasq的配置文件中添加以下内容（比如 `/etc/dnsmasq.d/10-consul`）

``` text
# Enable forward lookup of the 'consul' domain:
server=/consul/127.0.0.1#8600

# Uncomment and modify as appropriate to enable reverse DNS lookups for
# common netblocks found in RFC 1918, 5735, and 6598:
#rev-server=0.0.0.0/8,127.0.0.1#8600
#rev-server=10.0.0.0/8,127.0.0.1#8600
#rev-server=100.64.0.0/10,127.0.0.1#8600
#rev-server=127.0.0.1/8,127.0.0.1#8600
#rev-server=169.254.0.0/16,127.0.0.1#8600
#rev-server=172.16.0.0/12,127.0.0.1#8600
#rev-server=192.168.0.0/16,127.0.0.1#8600
#rev-server=224.0.0.0/4,127.0.0.1#8600
#rev-server=240.0.0.0/4,127.0.0.1#8600
```


配置文件创建好了，重启`dnsmasq`服务。
`dnsmasq`的其他有用的配置细节可以通过[`dnsmasq(8)`](http://www.thekelleys.org.uk/dnsmasq/docs/dnsmasq-man.html)查看。

```
# Accept DNS queries only from hosts whose address is on a local subnet.
#local-service

# Don't poll /etc/resolv.conf for changes.
#no-poll

# Don't read /etc/resolv.conf. Get upstream servers only from the command
# line or the dnsmasq configuration file (see the "server" directive below).
#no-resolv

# Specify IP address(es) of other DNS servers for queries not handled
# directly by consul. There is normally one 'server' entry set for every
# 'nameserver' parameter found in '/etc/resolv.conf'. See dnsmasq(8)'s
# 'server' configuration option for details.
#server=1.2.3.4
#server=208.67.222.222
#server=8.8.8.8

# Set the size of dnsmasq's cache. The default is 150 names. Setting the
# cache size to zero disables caching.
#cache-size=65536
```

### Unbound Setup


Unbound通常通过`unbound.conf`文件或者在`/etc/unbound/unbound.conf.d`下的一些文件进行配置，
在Unbound的配置文件中添加以下内容（比如 `/etc/unbound/unbound.conf.d/consul.conf`）

``` text
#Allow insecure queries to local resolvers
server:
  do-not-query-localhost: no
  domain-insecure: "consul"

#Add consul as a stub-zone
stub-zone:
  name: "consul"
  stub-addr: 127.0.0.1@8600
```

你可能需要在你的配置文件`/etc/unbound/unbound.conf` 的最下面添加一个配置，把最新的配置包含进去:

``` text
include: "/etc/unbound/unbound.conf.d/*.conf"
```

### iptables Setup




在Linux操作系统中，接收进来的请求并请求到本地主机可以用`iptables`转发到同一台机器上的其他端口而不再需要其他的服务。
由于 Consul 默认情况下只提供`.consul`的顶级域名，如果你希望`iptables`配置可以解析其他的域名，配置`recursors`尤为重要。
`recursors`不会包含本地主机当做跳转，而只会拦截这些请求。


iptables的方式适用于一个外部的DNS服务刚好已经运行在你的基础设施中，并且被用作一个`recursor`的环境中，
或者如果你想用一个已经存在的DNS服务当做你的查询终端并为Consul服务提供Consul域名的请求。
在这两种情况中，你可能想要查询consul服务但不需要在consul宿主上拆分服务。

```
[root@localhost ~]# iptables -t nat -A PREROUTING -p udp -m udp --dport 53 -j REDIRECT --to-ports 8600
[root@localhost ~]# iptables -t nat -A PREROUTING -p tcp -m tcp --dport 53 -j REDIRECT --to-ports 8600
[root@localhost ~]# iptables -t nat -A OUTPUT -d localhost -p udp -m udp --dport 53 -j REDIRECT --to-ports 8600
[root@localhost ~]# iptables -t nat -A OUTPUT -d localhost -p tcp -m tcp --dport 53 -j REDIRECT --to-ports 8600
```

### macOS Setup

在 MacOS 下，你可以使用MacOS的解析器解决 `.consul` 请求等问题。
只需在 `/etc/resolver/`中添加一个解析条目。
文档可以通过 `man resolver`获得。
创建一个新文件`/etc/resolver/consul` (需要root权限) 写入一下内容:

```
nameserver 127.0.0.1
port 8600
```

This is telling the macOS resolver daemon for all .consul TLD requests, ask 127.0.0.1 on port 8600.

### Testing

First, perform a DNS query against Consul directly to be sure that the record exists:
首先，对Consul地址DNS进行查询已确保该条记录存在
``` text
[root@localhost ~]# dig @localhost -p 8600 primary.redis.service.dc-1.consul. A

; <<>> DiG 9.8.2rc1-RedHat-9.8.2-0.23.rc1.32.amzn1 <<>> @localhost primary.redis.service.dc-1.consul. A
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 11536
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;primary.redis.service.dc-1.consul. IN A

;; ANSWER SECTION:
primary.redis.service.dc-1.consul. 0 IN A 172.31.3.234

;; Query time: 4 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Wed Apr  9 17:36:12 2014
;; MSG SIZE  rcvd: 76
```

Then run the same query against your BIND instance and make sure you get a
valid result:
然后对BIND实例运行相同的查询，并确保得到一个有效的结果:

``` text
[root@localhost ~]# dig @localhost -p 53 primary.redis.service.dc-1.consul. A

; <<>> DiG 9.8.2rc1-RedHat-9.8.2-0.23.rc1.32.amzn1 <<>> @localhost primary.redis.service.dc-1.consul. A
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 11536
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;primary.redis.service.dc-1.consul. IN A

;; ANSWER SECTION:
primary.redis.service.dc-1.consul. 0 IN A 172.31.3.234

;; Query time: 4 msec
;; SERVER: 127.0.0.1#53(127.0.0.1)
;; WHEN: Wed Apr  9 17:36:12 2014
;; MSG SIZE  rcvd: 76
```

如果和预料的一样，用相同的方法论证反向代理的DNS

``` text
[root@localhost ~]# dig @127.0.0.1 -p 8600 133.139.16.172.in-addr.arpa. PTR

; <<>> DiG 9.10.3-P3 <<>> @127.0.0.1 -p 8600 133.139.16.172.in-addr.arpa. PTR
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 3713
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 0
;; WARNING: recursion requested but not available

;; QUESTION SECTION:
;133.139.16.172.in-addr.arpa.   IN  PTR

;; ANSWER SECTION:
133.139.16.172.in-addr.arpa. 0  IN  PTR consul1.node.dc1.consul.

;; Query time: 3 msec
;; SERVER: 127.0.0.1#8600(127.0.0.1)
;; WHEN: Sun Jan 31 04:25:39 UTC 2016
;; MSG SIZE  rcvd: 109
[root@localhost ~]# dig @127.0.0.1 +short -x 172.16.139.133
consul1.node.dc1.consul.
```

### 故障排查（Troubleshooting）

如果你没有通过你的DNS服务(e.g. BIND, Dnsmasq)获取一个结果但是你通过Consul获取倒了，你最好确保打开你的DNS服务的日志查询一下发生了什么。

``` text
[root@localhost ~]# rndc querylog
[root@localhost ~]# tail -f /var/log/messages
```

日志展示的异常可能是这样的：
``` text
error (no valid RRSIG) resolving
error (no valid DS) resolving
```

这表明DNSSEC没有正确的禁用。

如果你看到有关网略链接的异常，核实防火墙关闭或者Consul与运行的BIND服务没有路由问题。

对于Dnsmasq，查看`log-queries`的配置选项和`USR1`信号。


