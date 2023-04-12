## 微服务

### 1. 什么是注册中心

- 服务提供者（RPC Server）：在启动时，向 Registry 注册自身服务，并向 Registry 定期发送心跳汇报存活状态。
- 服务消费者（RPC Client）：在启动时，向 Registry 订阅服务，把 Registry 返回的服务节点列表缓存在本地内存中，并与 RPC Sever 建立连接。
- 服务注册中心（Registry）：用于保存 RPC Server 的注册信息，当 RPC Server 节点发生变更时，Registry 会同步变更，RPC Client 感知后会刷新本地 内存中缓存的服务节点列表。

### 2. CAP 理论

- 一致性(Consistency)：所有节点在同一时间具有相同的数据；
- 可用性(Availability) ：保证每个请求不管成功或者失败都有响应；
- 分隔容忍(Partition tolerance) ：系统中任意信息的丢失或失败不会影响系统的继续运作。

关于 P 的理解，我觉得是在整个系统中某个部分，挂掉了，或者宕机了，并不影响整个系统的运作或者说使用，而可用性是，某个系统的某个节点挂了，但是并不影响系统的接受或者发出请求。

CAP 不可能都取，只能取其中 2 个

### 3. 分布式系统协议 Raft

[从 Raft 原理到实践](https://mp.weixin.qq.com/s?__biz=Mzg3OTU5NzQ1Mw==&mid=2247485759&idx=1&sn=41957e94a2c69426befafd373fbddcc5&chksm=cf034bddf874c2cb52a7aafea5cd194e70308c7d4ad74183db8a36d3747122be1c7a31b84ee3&token=179167416&lang=zh_CN#rd)

### 4. Consul 的主要特征

- CP 模型，使用 Raft 算法来保证强一致性，不保证可用性；
- 支持服务注册与发现、健康检查、KV Store 功能。
- 支持多数据中心，可以避免单数据中心的单点故障，而其部署则需要考虑网络延迟, 分片等情况等。

### 5. Consul 多数据中心

若两个 DataCenter，他们通过 Internet 互联，同时请注意为了提高通信效率，只有 Server 节点才加入跨数据中心的通信。

在单个数据中心中，Consul 分为 Client 和 Server 两种节点（所有的节点也被称为 Agent），Server 节点保存数据，Client 负责健康检查及转发数据请求到 Server；Server 节点有一个 Leader 和多个 Follower，Leader 节点会将数据同步到 Follower，Server 的数量推荐是 3 个或者 5 个，在 Leader 挂掉的时候会启动选举机制产生一个新的 Leader。

集群内的 Consul 节点通过 gossip 协议（流言协议）维护成员关系，也就是说某个节点了解集群内现在还有哪些节点，这些节点是 Client 还是 Server。

集群内数据的读写请求既可以直接发到 Server，也可以通过 Client 使用 RPC 转发到 Server，请求最终会到达 Leader 节点，在允许数据延时的情况下，读请求也可以在普通的 Server 节点完成。

### 6. Consul 的底层通讯协议 Gossip

gossip 协议也称之为流行病协议，它的信息传播行为类似流行病，或者森林的大火蔓延一样，一个接着一个，最终导致全局都收到某一个信息。

在 gossip 协议的网络中，有很多节点交叉分布，当其中的一个节点收到某条信息的时候，它会随机选择周围的几个节点去通知这个信息，收到信息的节点也会接着重复这个过程，直到网络中所有的节点都收到这条信息，才算信息同步完成。

在某个时刻下，网络节点中的信息可能是不对称的，gossip 协议不是一个强一致性的协议，而是最终一致性的协议，理解了这一层，我们去看 consul 的日志的时候，就能有一些端倪了，因为 consul 服务网络在运行的过程中，如果有新的服务注册进来，那么其他的节点会收到某个服务或者节点加入的信息。

### 7. Gossip 的优缺点

优点：

- 扩展性好，加入网络方便
- 容错性好，某个节点离开网络，不会影响整体的消息传播
- 去中心化，Gossip 协议的网络中，不存在中心节点的概念，每个节点都可以成为消息的第一个传播者，只要网络可达，信息就能散播到全网。
- 一致性收敛：这种一传十、十传百的消息传递机制，能够保证消息快速收敛，并保证最终一致性。

缺点：

- 消息延迟：这个是由它的特性决定的，消息的扩散需要时间，这中间各个节点的消息是不一致的。
- 消息冗余：A 节点告知 B 的信息，B 可能会反过来告知 A，这个时候 A 本身已经包含这个消息，却还要处理 B 的请求，这会造成消息的冗余，提高节点处理信息的压力。