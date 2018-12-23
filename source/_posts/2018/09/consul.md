---
title: consul笔记
date: 2018-09-24 01:53:10
categories: "微服务"
tags:
  - 微服务
  - 分布式系统
  - 服务发现
---

## Consul简介

### consul介绍

`consul` 是Google开源的，使用go语言开发的一个_服务发现_、_配置管理服务_的一个工具。内置了 **服务注册与发现框架**、**分布一致性协议实现**、**健康检查**、**Key/Value存储**、**多数据中心方案**，不再需要依赖其他工具（比如`ZooKeeper`等）。服务部署简单，只有一个可运行的二进制的包。

几个关键的功能介绍(作用):
- **服务发现**: `Consul`的某些客户端可以提供一个服务，例如`api`或者`mysql`，其它客户端可以使用Consul去发现这个服务的提供者。使用DNS或者HTTP，应用可以很容易的找到他们所依赖的服务。
- **健康检查**: Consul客户端可以提供一些健康检查，这些健康检查可以关联到一个指定的服务（服务是否返回200 OK），也可以关联到本地节点（内存使用率是否在90%以下）。这些信息可以被一个操作员用来监控集群的健康状态，被服务发现组件路由时用来远离不健康的主机。
- **健值(Key/Value)存储**: 应用可以使用Consul提供的分层键值存储用于一些目的，包括 **动态配置** 、**特征标记**、**协作**、**leader选举**等等。通过一个简单的HTTP API可以很容易的使用这个组件。
- **多数据中心**: Consul对多数据中心有非常好的支持，这意味着Consul用户不必担心由于创建更多抽象层而产生的多个区域。

### 基础架构

每个节点都需要运行`agent`，他有两种运行模式`server`和`client`。每个数据中心官方建议需要3或5个`server`节点以保证数据安全，同时保证`server-leader`的选举能够正确的进行。

#### @client
CLIENT表示consul的`client模式`，就是客户端模式。是consul节点的一种模式，这种模式下，所有注册到当前节点的服务会被转发到`SERVER`，本身是不持久化这些信息。

#### @server
SERVER表示consul的server模式，表明这个consul是个`server`，这种模式下，功能和`CLIENT`都一样，唯一不同的是，它会把所有的信息持久化的本地，这样遇到故障，信息是可以被保留的

#### @server-leader
中间那个`SERVER`下面有`LEADER`的字眼，表明这个`SERVER`是它们的老大，它和其它`SERVER`不一样的一点是，**它需要负责同步注册的信息给其它的SERVER，同时也要负责各个节点的健康监测。**

#### @raft
server节点之间的数据一致性保证，一致性协议使用的是`raft`协议。


## 安装

### 1.通过`choco`工具安装
```shell
choco install -y consul
```

### 2. 官方安装
从[官方下载](https://www.consul.io/downloads.html) 对应系统的`zip`包,解压到安装目录，并将安装目录的路径添加到环境变量`PATH`中。 


## 启动

完成`consul`安装之后必须运行`agent`,`agent` 可以运行`server`或`client`模式。每个数据中心至少必须拥有一台`server`， 建议在一个集群中有3或者5个`server`.部署单一的`server`,在出现失败时会不可避免的造成数据丢失。

### 运行Consul Server
```shell
consul agent -server -bootstrap-expect 3 -data-dir /tmp/consul -node=s1 -bind 192.168.31.115 -ui-dir ./consul_ui/ -rejoin -config-dir=/etc/consul.d/ -client 0.0.0.0
```
- `-server1`： 定义agent运行在server模式
- `-bootstrap-expect` ：在一个datacenter中期望提供的server节点数目，当该值提供的时候，consul一直等到达到指定sever数目的时候才会引导整个集群，该标记不能和bootstrap共用
- `-bind`：该地址用来在集群内部的通讯，集群内的所有节点到地址都必须是可达的，默认是0.0.0.0
- `-node`：节点在集群中的名称，在一个集群中必须是唯一的，默认是该节点的主机名
- `-ui-dir`： 提供存放web ui资源的路径，该目录必须是可读的
- `-rejoin`：使consul忽略先前的离开，在再次启动后仍旧尝试加入集群中。
- `-config-dir`：：配置文件目录，里面所有以.json结尾的文件都会被加载
- `-client`：consul服务侦听地址，这个地址提供HTTP、DNS、RPC等服务，默认是127.0.0.1所以不对外提供服务，如果你要对外提供服务改成0.0.0.0
- `-ui`： 使用默认的ui管理界面

### 运行Consul Client
```shell
consul agent -data-dir /tmp/consul -node=c1 -bind=192.168.31.23 -config-dir=/etc/consul.d/ -join 192.168.31.115
```
### 查看集群成员
```shell
consul members
```

### 加入集群
```shell
consul join 192.168.31.115
```

### 停止agent
你可以使用`Ctrl-C` 优雅的关闭`Agent`. 中断`Agent`之后你可以看到他离开了集群并关闭.

### 开发模式
开发过程中只需要简单的运行一个consul `Agent`即可,可通过如下命令实现:
```shell
consul agent --dev
```

## 服务注册与发现

### 服务注册
consul支持两种方式实现服务注册，一种是通过`consul`的服务注册http API，由服务自己调用API实现注册，另一种方式是通过json个是的配置文件实现注册，将需要注册的服务以json格式的配置文件给出。`consul`官方建议使用第二种方式。

#### 通过Http API注册
```shell
/v1/agent/service/register -PUT
```

#### 通过配置文件注册文件
1. 将注册的服务写到 `consul`配置目录下
```shell
echo '{"service": {"name": "web", "tags": ["rails"], "port": 80}}' >/etc/consul.d/web.json
```
2. 启动consul服务时,指定配置目录
```shell
 -config-dir=/etc/consul.d/ 
```




### 服务发现
consul支持两种方式实现服务发现，一种是通过`http API`来查询有哪些服务，另外一种是通过`consul agent` 自带的`DNS`（8600端口），域名是以`NAME.service.consul`的形式给出，`NAME`即在定义的服务配置文件中服务的名称。`DNS方式`可以通过`check`的方式检查服务。

例如: 已经向consul集群中注册了一个服务名为`Web`的服务，那么他的域名是`web.service.consul`：

```sehll
dig @127.0.0.1 -p 8600 web.service.consul
```


#### Http API
1. 查询制定服务
```shell
curl 127.0.0.1:8500/v1/catalog/service/web
```
2. 通过目录API给出所有节点提供的服务
```shell
curl 127.0.0.1:8500/v1/catalog/service/web?passing
```


#### DNS API
DNS API中,服务的DNS名字是 `NAME.service.consul`. 虽然是可配置的,但默认的所有DNS名字会都在consul命名空间下.这个子域告诉Consul,我们在查询服务,`NAME`则是服务的名称。

## 健康检查
健康检查是服务发现的关键组件.预防使用到不健康的服务

### 自定义检查
使用和检查定义来注册检查，和服务类似,因为这是建立检查最常用的方式

通过配置服务下的`check`节点自定义检查:
```json
{"service": {
    "name": "Faceid",
    "tags": ["extract", "verify", "compare", "idcard"],
    "address": "10.201.102.198",
    "port": 9000,
    "check": {
        "name": "ping",
        "script": "curl -s localhost:9000",
        "interval": "3s"
        }
    }
}
```

## K/V
Consul提供了一个易用的键/值存储,这可以用来 **保持动态配置** ,**协助服务协调**,**领袖选举**,做开发者可以想到的任何事情
可以通过Http API `v1/kv/?{key}`来获取设置获取KEY/VALUE的存储与设置


## 常用命令
### 概述
consul只有一个命令行应用，就是`consul`命令，consul命令可以包含`agent`、`members`等参数进行使用,`consul -h`即可看到consul cli所支持的参数，而每个参数里面又支持其他参数。

```shell
[root@consul ~]# consul
usage: consul [--version] [--help] <command> [<args>]
Available commands are:
    agent          Runs a Consul agent  运行一个consul agent
    configtest     Validate config file
    event          Fire a new event
    exec           Executes a command on Consul nodes  在consul节点上执行一个命令
    force-leave    Forces a member of the cluster to enter the "left" state   强制节点成员在集群中的状态转换到left状态
    info           Provides debugging information for operators  提供操作的debug级别的信息
    join           Tell Consul agent to join cluster   加入consul节点到集群中
    keygen         Generates a new encryption key  生成一个新的加密key
    keyring        Manages gossip layer encryption keys
    kv             Interact with the key-value store
    leave          Gracefully leaves the Consul cluster and shuts down
    lock           Execute a command holding a lock
    maint          Controls node or service maintenance mode
    members        Lists the members of a Consul cluster    列出集群中成员
    monitor        Stream logs from a Consul agent  打印consul节点的日志信息
    operator       Provides cluster-level tools for Consul operators
    reload         Triggers the agent to reload configuration files   触发节点重新加载配置文件
    rtt            Estimates network round trip time between nodes
    snapshot       Saves, restores and inspects snapshots of Consul server state
    version        Prints the Consul version    打印consul的版本信息
    watch          Watch for changes in Consul   监控consul的改变

```
### Agent

`agent`指令是consul的核心，它运行`agent`来维护成员的重要信息、运行检查、服务宣布、查询处理等等

### event

`event`命令提供了一种机制，用来`fire`自定义的用户事件，这些事件对consul来说是不透明的，但它们可以用来构建自动部署、重启服务或者其他行动的脚本。

```shell
-http-addr：http服务的地址，agent可以链接上来发送命令，如果没有设置，则默认是127.0.0.1:8500。
-datacenter：数据中心。
-name：事件的名称
-node：一个正则表达式，用来过滤节点
-service：一个正则表达式，用来过滤节点上匹配的服务
-tag：一个正则表达式，用来过滤节点上符合tag的服务，必须和-service一起使用。
```
### exec
`exec`指令提供了一种远程执行机制，比如你要在所有的机器上执行`uptime`命令，远程执行的工作通过`job`来指定，存储在KV中，`agent`使用`event`系统可以快速的知道有新的job产生，消息是通过gossip协议来传递的，因此消息传递是最佳的，但是并不保证命令的执行。事件通过gossip来驱动，远程执行依赖KV存储系统(就像消息代理一样)。

```shell
-http-addr：http服务的地址，agent可以链接上来发送命令，如果没有设置，则默认是127.0.0.1:8500。
-datacenter：数据中心。
-prefix：key在KV系统中的前缀，用来存储请求数据，默认是_rexec
-node：一个正则表达式，用来过滤节点，评估事件
-service：一个正则表达式，用来过滤节点上匹配的服务
-tag：一个正则表达式，用来过滤节点上符合tag的服务，必须和-service一起使用。
-wait：在节点多长时间没有响应后，认为job已经完成。
-wait-repl：
-verbose：输出更多信息
```

### join
`join`指令告诉`consul agent`加入一个已经存在的集群中，一个新的`consul agent`必须加入一个已经有至少一个成员的集群中，这样它才能加入已经存在的集群中，如果你不加入一个已经存在的集群，则agent是它自身集群的一部分，其他`agent`则可以加入进来。`agents`可以加入其他`agent`多次。`consul join [options] address`。如果你想加入多个集群，则可以写多个地址，consul会加入所有的地址。

### leave
`leave`指令触发一个优雅的离开动作并关闭`agent`，节点离开后不会尝试重新加入集群中。运行在`server`状态的节点，节点会被优雅的删除，这是很严重的，**在某些情况下一个不优雅的离开会影响到集群的可用性**。

### members
`members`指令输出`consul agent`目前所知道的所有的成员以及它们的状态，节点的状态只有`alive`、`left`、`failed`三种状态。

### monitor
`monitor`指令用来链接运行的`agent`，并显示日志。`monitor`会显示最近的日志，并持续的显示日志流，不会自动退出，除非你手动或者远程`agent`自己退出。

### reload
`reload`指令可以重新加载`agent`的配置文件。SIGHUP指令在重新加载配置文件时使用，任何重新加载的错误都会写在`agent`的`log`文件中，并不会打印到屏幕。

### watch
`watch`指令提供了一个机制，用来监视实际数据视图的改变(节点列表、成员服务、KV)，如果没有指定进程，当前值会被dump出来

## Consul 配置
`agent`有各种各样的配置项可以在命令行或者配置文件进行定义，所有的配置项都是可选择的，当加载配置文件的时候，consul从配置文件或者配置目录加载配置。后面定义的配置会合并前面定义的配置，但是大多数情况下，合并的意思是后面定义的配置会覆盖前面定义的配置。

### 详细的配置文件参数

```shell
acl_datacenter：只用于server，指定的datacenter的权威ACL信息，所有的servers和datacenter必须同意ACL datacenter
acl_default_policy：默认是allow
acl_down_policy：
acl_master_token：
acl_token：agent会使用这个token和consul server进行请求
acl_ttl：控制TTL的cache，默认是30s
addresses：一个嵌套对象，可以设置以下key：dns、http、rpc
advertise_addr：等同于-advertise
bootstrap：等同于-bootstrap
bootstrap_expect：等同于-bootstrap-expect
bind_addr：等同于-bind
ca_file：提供CA文件路径，用来检查客户端或者服务端的链接
cert_file：必须和key_file一起
check_update_interval：
client_addr：等同于-client
datacenter：等同于-dc
data_dir：等同于-data-dir
disable_anonymous_signature：在进行更新检查时禁止匿名签名
disable_remote_exec：禁止支持远程执行，设置为true，agent会忽视所有进入的远程执行请求
disable_update_check：禁止自动检查安全公告和新版本信息
dns_config：是一个嵌套对象，可以设置以下参数：allow_stale、max_stale、node_ttl 、service_ttl、enable_truncate
domain：默认情况下consul在进行DNS查询时，查询的是consul域，可以通过该参数进行修改
enable_debug：开启debug模式
enable_syslog：等同于-syslog
encrypt：等同于-encrypt
key_file：提供私钥的路径
leave_on_terminate：默认是false，如果为true，当agent收到一个TERM信号的时候，它会发送leave信息到集群中的其他节点上。
log_level：等同于-log-level
node_name:等同于-node
ports：这是一个嵌套对象，可以设置以下key：dns(dns地址：8600)、http(http api地址：8500)、rpc(rpc:8400)、serf_lan(lan port:8301)、serf_wan(wan port:8302)、server(server rpc:8300)
protocol：等同于-protocol
recursor：
rejoin_after_leave：等同于-rejoin
retry_join：等同于-retry-join
retry_interval：等同于-retry-interval
server：等同于-server
server_name：会覆盖TLS CA的node_name，可以用来确认CA name和hostname相匹配
skip_leave_on_interrupt：和leave_on_terminate比较类似，不过只影响当前句柄
start_join：一个字符数组提供的节点地址会在启动时被加入
statsd_addr：
statsite_addr：
syslog_facility：当enable_syslog被提供后，该参数控制哪个级别的信息被发送，默认Local0
ui_dir：等同于-ui-dir
verify_incoming：默认false，如果为true，则所有进入链接都需要使用TLS，需要客户端使用ca_file提供ca文件，只用于consul server端，因为client从来没有进入的链接
verify_outgoing：默认false，如果为true，则所有出去链接都需要使用TLS，需要服务端使用ca_file提供ca文件，consul server和client都需要使用，因为两者都有出去的链接
watches：watch一个详细名单

```


## 常用API
consul的主要接口是RESTful HTTP API，该API可以用来增删查改nodes、services、checks、configguration。所有的endpoints主要分为以下类别:
```shell
kv - Key/Value存储
agent - Agent控制
catalog - 管理nodes和services
health - 管理健康监测
session - Session操作
acl - ACL创建和管理
event - 用户Events
status - Consul系统状态
```

### agent
agent endpoints用来和本地agent进行交互，一般用来服务注册和检查注册，支持以下接口

```shell
/v1/agent/checks : 返回本地agent注册的所有检查(包括配置文件和HTTP接口)
/v1/agent/services : 返回本地agent注册的所有 服务
/v1/agent/members : 返回agent在集群的gossip pool中看到的成员
/v1/agent/self : 返回本地agent的配置和成员信息
/v1/agent/join/<address> : 触发本地agent加入node
/v1/agent/force-leave/<node>>: 强制删除node
/v1/agent/check/register : 在本地agent增加一个检查项，使用PUT方法传输一个json格式的数据
/v1/agent/check/deregister/<checkID> : 注销一个本地agent的检查项
/v1/agent/check/pass/<checkID> : 设置一个本地检查项的状态为passing
/v1/agent/check/warn/<checkID> : 设置一个本地检查项的状态为warning
/v1/agent/check/fail/<checkID> : 设置一个本地检查项的状态为critical
/v1/agent/service/register : 在本地agent增加一个新的服务项，使用PUT方法传输一个json格式的数据
/v1/agent/service/deregister/<serviceID> : 注销一个本地agent的服务项
```

### catalog
catalog endpoints用来注册/注销nodes、services、checks

```shell
/v1/catalog/register : Registers a new node, service, or check
/v1/catalog/deregister : Deregisters a node, service, or check
/v1/catalog/datacenters : Lists known datacenters
/v1/catalog/nodes : Lists nodes in a given DC
/v1/catalog/services : Lists services in a given DC
/v1/catalog/service/<service> : Lists the nodes in a given service
/v1/catalog/node/<node> : Lists the services provided by a node
```

### health
health endpoints用来查询健康状况相关信息，该功能从catalog中单独分离出来
```shell
/v1/healt/node/<node>: 返回node所定义的检查，可用参数?dc=
/v1/health/checks/<service>: 返回和服务相关联的检查，可用参数?dc=
/v1/health/service/<service>: 返回给定datacenter中给定node中service
/v1/health/state/<state>: 返回给定datacenter中指定状态的服务，state可以是"any", "unknown", "passing", "warning", or "critical"，可用参数?dc=
```

### session
session endpoints用来create、update、destory、query sessions
```shell
/v1/session/create: Creates a new session
/v1/session/destroy/<session>: Destroys a given session
/v1/session/info/<session>: Queries a given session
/v1/session/node/<node>: Lists sessions belonging to a node
/v1/session/list: Lists all the active sessions
```
### acl
acl endpoints用来create、update、destory、query acl
```shell
/v1/acl/create: Creates a new token with policy
/v1/acl/update: Update the policy of a token
/v1/acl/destroy/<id>: Destroys a given token
/v1/acl/info/<id>: Queries the policy of a given token
/v1/acl/clone/<id>: Creates a new token by cloning an existing token
/v1/acl/list: Lists all the active tokens
```

### event
event endpoints用来fire新的events、查询已有的events
```shell
/v1/event/fire/<name>: 触发一个新的event，用户event需要name和其他可选的参数，使用PUT方法
/v1/event/list: 返回agent知道的events
```
### status
status endpoints用来或者consul 集群的信息
```shell
/v1/status/leader : 返回当前集群的Raft leader
/v1/status/peers : 返回当前集群中同事
```


## 高可用
Consul Cluster集群架构图如下:

![架构图](https://www.consul.io/assets/images/consul-arch-420ce04a.png)

为保证consul集群服务的可靠性,一般需要配置3~5个`server`, `client`集群可以配置n个(n>=0),当`server-leader`问题,其他`server`会重新选举`server-leader`,从而保证集群的可靠性。


## 参考资料
- [Consul服务手册](http://www.liangxiansen.cn/2017/04/06/consul/#%E4%BD%BF%E7%94%A8consul)

- [Consul官方文档](https://www.consul.io/intro/getting-started/install.html)

- [Consul中文翻译](http://www.consul.la/start)