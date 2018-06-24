
本节为 **常见使用场景** 提供了各种指南。如下:

- **ACLs** - This guide covers `Consul`'s `Access Control List` (`ACL`) capability, which can be used to control access to Consul resources.

- **Adding/Removing Servers** - This guide covers how to safely add and remove Consul servers from the cluster. This should be done carefully to avoid availability outages.

- **Autopilot** - This guide covers Autopilot, which provides automatic operator-friendly management of Consul servers.

- **Bootstrapping** - This guide covers bootstrapping a new datacenter. This covers safely adding the initial Consul servers.

- **Consul with Containers** - This guide describes critical aspects of operating a Consul cluster that's run inside containers.

- **[DNS Caching](guides/dns-cache.md)** - 为DNS查询缓存启用TTL

- **[DNS Forwarding](guides/dns-forwarding.md)** - 绑定 Consul 转发 DNS 请求

- **External Services** - This guide covers registering an external service. This allows using 3rd party services within the Consul framework.

- _Federation (Basic and Advanced)(**ENTERPRISE**)_ - Configuring Consul to support multiple datacenters.

- **Geo Failover** - This guide covers using prepared queries to implement geo failover for services.

- **Leader Election** - The goal of this guide is to cover how to build client-side leader election using Consul.

- _Network Segments(**ENTERPRISE**)_ - Configuring Consul to support partial LAN connectivity using Network Segments.

- **Outage Recovery** - This guide covers recovering a cluster that has become unavailable due to server failures.

- **Semaphore** - This guide covers using the KV store to implement a semaphore.

- _Sentinel(**ENTERPRISE**)_ - This guide covers using Sentinel for policy enforcement in Consul.

- **Server Performance** - This guide covers minimum requirements for Consul servers as well as guidelines for running Consul servers in production.