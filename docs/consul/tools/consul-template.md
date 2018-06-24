
## Consul-Template简介
Consul-Template是基于Consul的**自动替换配置文件**的应用。在Consul-Template没出现之前，大家构建服务发现系统大多采用的是Zookeeper、Etcd+Confd这样类似的系统。

Consul官方推出了自己的模板系统Consul-Template后，动态的配置系统可以分化为**Etcd+Confd**和**Consul+Consul-Template**两大阵营。Consul-Template的定位和Confd差不多，Confd的后端可以是Etcd或者Consul。

Consul-Template提供了一个便捷的方式从Consul中获取存储的值，**Consul-Template守护进程会查询Consul实例来更新系统上指定的任何模板。当更新完成后，模板还可以选择运行一些任意的命令。**

## Consul-Template的使用场景

Consul-Template可以查询Consul中的**服务目录**、**Key**、**Key-values**等。这种强大的抽象功能和查询语言模板可以使Consul-Template**特别适合动态的创建配置文件**。
例如：创建Apache/Nginx Proxy Balancers、Haproxy Backends、Varnish Servers、Application Configurations等。


# Read More
- [GitHub 官方教程](https://github.com/hashicorp/consul-template)
- 中文文档 [通过Consul-Template实现动态配置服务](https://www.hi-linux.com/posts/36431.html) 
