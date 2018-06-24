# Install Consul

下载地址： https://www.consul.io/downloads.html

## 安装

下载后解压，把 `consul`（`consul.exe`）可执行文件，配置到环境变量即可。

> Consul 的安装很简单，**就 consul 这个一个可执行文件**

## 查看安装是否成功

``` bash
>consul.exe
Usage: consul [--version] [--help] <command> [<args>]

Available commands are:
    agent          Runs a Consul agent
    catalog        Interact with the catalog
    event          Fire a new event
    exec           Executes a command on Consul nodes
    force-leave    Forces a member of the cluster to enter the "left" state
    info           Provides debugging information for operators.
    join           Tell Consul agent to join cluster
    keygen         Generates a new encryption key
    keyring        Manages gossip layer encryption keys
    kv             Interact with the key-value store
    leave          Gracefully leaves the Consul cluster and shuts down
    lock           Execute a command holding a lock
    maint          Controls node or service maintenance mode
    members        Lists the members of a Consul cluster
    monitor        Stream logs from a Consul agent
    operator       Provides cluster-level tools for Consul operators
    reload         Triggers the agent to reload configuration files
    rtt            Estimates network round trip time between nodes
    snapshot       Saves, restores and inspects snapshots of Consul server state
    validate       Validate config files/directories
    version        Prints the Consul version
    watch          Watch for changes in Consul
```

# Run the Consul Agent

Consul 安装之后，代理（Agent）必须运行。代理可以在 `server` 或 `client` 模式下运行。每个数据中心（datacenter）都必须至少有一台 `server`，但推荐使用`3台或5台`。非常不鼓励只部署一个 `server`，因为在故障情况下数据会丢失。

所有其他`agent`以`client`模式运行。`client端`是一个非常轻量级的进程，它注册服务、运行健康检查、并将查询转发给服务器。`agent`必须在群集中的每个节点上运行。

创建数据中心的更多详细信息，请参阅 [本指南](https://www.consul.io/docs/guides/bootstrapping.html)。

## Starting the Agent
为了简单起见，我们现在将以开发模式启动Consul Agent。这种模式对于快速简单地启动单节点Consul环境非常有用。但是不要在生产中使用，因为它不会保存任何状态（It is not intended to be used in production as it does not persist any state）

``` bash
>consul agent -dev
==> Starting Consul agent...
2018/01/15 17:32:12 [WARN] Unable to get address for server id be6f0352-0fde-99ca-c5f9-77bb580060d1, using fallback address 127.0.0.1:8300: Could not find address for server id be6f0352-0fde-99ca-c5f9-77bb580060d1
==> Consul agent running!
           Version: 'v1.0.2'
           Node ID: 'be6f0352-0fde-99ca-c5f9-77bb580060d1'
         Node name: 'TTP-Kail'
        Datacenter: 'dc1' (Segment: '<all>')
            Server: true (Bootstrap: false)
       Client Addr: [127.0.0.1] (HTTP: 8500, HTTPS: -1, DNS: 8600)
      Cluster Addr: 127.0.0.1 (LAN: 8301, WAN: 8302)
           Encrypt: Gossip: false, TLS-Outgoing: false, TLS-Incoming: false

==> Log data will now stream in as it occurs:

    2018/01/15 17:32:11 [DEBUG] Using random ID "be6f0352-0fde-99ca-c5f9-77bb580060d1" as node ID
    2018/01/15 17:32:12 [INFO] raft: Initial configuration (index=1): [{Suffrage:Voter ID:be6f0352-0fde-99ca-c5f9-77bb580060d1 Address:127.0.0.1:8300}]
    2018/01/15 17:32:12 [INFO] raft: Node at 127.0.0.1:8300 [Follower] entering Follower state (Leader: "")
    2018/01/15 17:32:12 [WARN] raft: Heartbeat timeout from "" reached, starting election
    2018/01/15 17:32:12 [INFO] raft: Node at 127.0.0.1:8300 [Candidate] entering Candidate state in term 2
    2018/01/15 17:32:12 [DEBUG] raft: Votes needed: 1
    2018/01/15 17:32:12 [DEBUG] raft: Vote granted from be6f0352-0fde-99ca-c5f9-77bb580060d1 in term 2. Tally: 1
    2018/01/15 17:32:12 [INFO] raft: Election won. Tally: 1
    2018/01/15 17:32:12 [INFO] raft: Node at 127.0.0.1:8300 [Leader] entering Leader state
    2018/01/15 17:32:12 [INFO] serf: EventMemberJoin: TTP-Kail.dc1 127.0.0.1
    2018/01/15 17:32:12 [INFO] serf: EventMemberJoin: TTP-Kail 127.0.0.1
    2018/01/15 17:32:12 [INFO] consul: Adding LAN server TTP-Kail (Addr: tcp/127.0.0.1:8300) (DC: dc1)
    2018/01/15 17:32:12 [INFO] consul: Handled member-join event for server "TTP-Kail.dc1" in area "wan"
    2018/01/15 17:32:12 [INFO] consul: cluster leadership acquired
    2018/01/15 17:32:12 [INFO] consul: New leader elected: TTP-Kail
    2018/01/15 17:32:12 [INFO] agent: Started DNS server 127.0.0.1:8600 (udp)
    2018/01/15 17:32:12 [INFO] agent: Started DNS server 127.0.0.1:8600 (tcp)
    2018/01/15 17:32:12 [INFO] agent: Started HTTP server on 127.0.0.1:8500 (tcp)
    2018/01/15 17:32:12 [INFO] agent: started state syncer
    2018/01/15 17:32:12 [DEBUG] consul: Skipping self join check for "TTP-Kail" since the cluster is too small
    2018/01/15 17:32:12 [INFO] agent: Synced node info
    2018/01/15 17:32:12 [DEBUG] agent: Node info in sync
    2018/01/15 17:32:12 [INFO] consul: member 'TTP-Kail' joined, marking health alive
    2018/01/15 17:32:12 [DEBUG] consul: Skipping self join check for "TTP-Kail" since the cluster is too small
    2018/01/15 17:32:13 [DEBUG] Skipping remote check "serfHealth" since it is managed automatically
    2018/01/15 17:32:13 [DEBUG] agent: Node info in sync
```

如您所见，Consul Agent 已经启动并输出了一些日志数据。从日志数据中，您可以看到我们的代理正在Server模式下运行（`Server: true (Bootstrap: false)`），并声称对群集具有领导力(`Node at 127.0.0.1:8300 [Leader] entering Leader state`)。此外，当地成员已被标记为该群集的健康成员。

> Consul使用您的主机名作为默认节点名称。如果您的主机名包含句点（`.`），则对该节点的DNS查询将无法与Consul一起使用。为了避免这种情况，请使用 `-node` 显式设置您的节点的名称与ID。可以通过 `consul agent -?`命令查看清洗参数

## Cluster Members

如果您在另一个终端中运行 [`consul members`](https://www.consul.io/docs/commands/members.html)，则可以看到Consul群集的成员。由于刚刚本地就启动了一个Agent，没有其他成员加入，所以现在只应该看到一个成员（本机）：

``` bash
λ consul.exe members
Node      Address         Status  Type    Build  Protocol  DC   Segment
TTP-Kail  127.0.0.1:8301  alive   server  1.0.2  2         dc1  <all>
```

## Stopping the Agent


# Registering Services

## Defining a Service

# Consul Cluster
# Health Checks
# KV Data
# Consul Web UI

http://www.consul.la/docs/internals/coordinates