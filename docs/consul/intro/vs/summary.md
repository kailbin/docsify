
Consul 主要用来进行服务发现，还附加其他功能。

- **服务发现** 
    1. 使用起来不如Eureka简单，对Java应用来说，都是使用 HTTP Api, SDK 社区 感觉不如 Eureka 活跃，Consul 官方没有没提供针对 HTTP Api 的客户端，客户端全部来自社区
    2. Zookeeper 作为服务发现来说，貌似需要自主开发，或者依赖 Spring Cloud 对 ZooKeeper 的支持
    3. 对于运维做应用程序级别的服务发现来说，非常方便，Eureka和Zookeeper不太适合运维进行集成

- 健康检查
    1. Eureka 通过超时续约来做，Consul 貌似也是
    2. Zookeeper 如果客户端关掉的话，通过创创建的临时 Key 会自动删除，实时性比 Eureka和Consul要高
- 锁
    1. Eureka 无此功能
    2. Zookeeper 和 Consul 差不多
- Key/Value存储
    1. 不如Redis
- 多数据中心
- 事件系统
    1. Eureka 无此功能
    2. Consul Http 长连接
    3. Socket 长连接
- ACL
    1. Eureka 无此功能
    2. Consul 和 Zookeeper 均支持
    
    
