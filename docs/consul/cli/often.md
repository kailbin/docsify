
# agent
``` bash
# -server ： 服务模式会持久化数据
# -ui ： 图形界面可以远程访问，否则只能本机访问
consul agent -server -data-dir=/tmp/consul -bind=0.0.0.0 -ui -client=0.0.0.0 -bootstrap

#  -dev：开发模式，每次启动数据都是空的
consul agent -dev
```

# join（将agent加入到consul集群）

``` bash
consul join -wan dc1-server1
consul join dc1-server1
```

# members（列出consul cluster集群中的成员）

``` bash
consul member
consul member -wan
```

# leave（将节点移除所在集群）


# Consul Docker
``` bash
# 单机启动
docker run -d -p 8500:8500 --name=consul-test consul agent -server -data-dir=/tmp/consul -bind=0.0.0.0 -ui -client=0.0.0.0 -bootstrap

# 进入 consul docker 容器
docker exec -it consul-test sh
```

> https://hub.docker.com/_/consul/