## 分布式系统典型实例

### MapReduce

#### 执行流程

1. 用户程序首先调用的 MapReduce 库将输入文件分成 M 个数据片度，每个数据片段的大小一般从 16MB 到 64MB（可以通过可选的参数来控制每个数据片段的大小）。然后用户程序在机群中创建大量的程序副本。
2. 这些程序副本中的有一个特殊的程序–master。副本中其它的程序都是 worker 程序，由 master 分配任务。有 M 个 Map 任务和 R 个 Reduce 任务将被分配，master 将一个 Map 任务或 Reduce 任务分配给一个空闲的 worker。
3. 被分配了 map 任务的 worker 程序读取相关的输入数据片段，从输入的数据片段中解析出 key/value pair，然后把 key/value pair 传递给用户自定义的 Map 函数，由 Map 函数生成并输出的中间 key/value pair，并缓存在内存中。
4. 缓存中的 key/value pair 通过分区函数分成 R 个区域，之后周期性的写入到本地磁盘上。缓存的 key/value pair 在本地磁盘上的存储位置将被回传给 master，由 master 负责把这些存储位置再传送给 Reduce worker
5. 当 Reduce worker 程序接收到 master 程序发来的数据存储位置信息后，使用 RPC 从 Map worker 所在主机的磁盘上读取这些缓存数据。当 Reduce worker 读取了所有的中间数据后，通过对 key 进行排序后使得具有相同 key 值的数据聚合在一起。由于许多不同的 key 值会映射到相同的 Reduce 任务上，因此必须进行排序。如果中间数据太大无法在内存中完成排序，那么就要在外部进行排序。
6. Reduce worker 程序遍历排序后的中间数据，对于每一个唯一的中间 key 值，Reduce worker 程序将这个 key 值和它相关的中间 value 值的集合传递给用户自定义的 Reduce 函数。Reduce 函数的输出被追加到所属分区的输出文件。
7. 当所有的 Map 和 Reduce 任务都完成之后，master 唤醒用户程序。在这个时候，在用户程序里的对 MapReduce 调用才返回。

#### 容错

##### worker 故障

master 与 worker 之间同步心跳，对于失效的 worker，根据其类型来做进一步处理：

- Map worker 故障：由于 Map 任务将数据临时存储在本地，所以需要重新执行。
- Reduce worker 故障：由于 Reduce 任务将数据存储在全局文件系统中 ，所以不需要重新执行。

##### master 故障

MapReduce 任务重新执行

#### 故障语义保证

当用户提供的 Map 和 Reduce 操作是输入确定性函数（即相同的输入产生相同的输出）时，MapReduce 的分布式实现在任何情况下的输出都和所有程序没有出现任何错误、顺序的执行产生的输出是一样的。

- Map worker 任务的原子提交：每个 Map 任务生成 R 个本地临时文件，当一个 Map 任务完成时，worker 发送一个包含 R 个临时文件名的完成消息给 master。如果 master 从一个已经完成的 Map 任务再次接收到一个完成消息，master 将忽略这个消息；
- Reduce worker 任务的原子提交：当 Reduce 任务完成时，Reduce worker 进程以原子的方式把临时文件重命名为最终的输出文件。如果同一个 Reduce 任务在多台机器上执行，针对同一个最终的输出文件将有多个重命名操作执行。MapReduce 依赖底层文件系统提供的重命名操作的原子性来保证最终的文件系统状态仅仅包含一个 Reduce 任务产生的数据。

#### 存储位置优化

核心思想：本地读文件以减少流量消耗

MapReduce 的 master 在调度 Map 任务时会考虑输入文件的位置信息，尽量将一个 Map 任务调度在包含相关输入数据拷贝的机器上执行；如果上述努力失败了，master 将尝试在保存有输入数据拷贝的机器附近的机器上执行 Map 任务（例如，分配到一个和包含输入数据的机器在一个交换机里的 worker 机器上执行）。

#### 备用任务

影响一个 MapReduce 的总执行时间最通常的因素是“落伍者”：在运算过程中，如果有一台机器花了很长的时间才完成最后几个 Map 或 Reduce 任务，导致 MapReduce 操作总的执行时间超过预期。

为了解决落伍者的问题，当一个 MapReduce 操作接近完成的时候，master 调度备用（backup）任务进程来执行剩下的、处于处理中状态（in-progress）的任务。无论是最初的执行进程、还是备用（backup）任务进程完成了任务，MapReduce 都把这个任务标记成为已经完成。此个机制通常只会占用比正常操作多几个百分点的计算资源。但能减少近 50% 的任务完成总时间。

#### 应用场景

- 计算 URL 访问频率：Map 函数处理日志中 web 页面请求的记录，然后输出 (URL,1)。Reduce 函数把相同 URL 的 value 值都累加起来，产生 (URL, 记录总数）结果。
- 网络链接倒排：Map 函数在源页面（source）中搜索所有的链接目标（target）并输出为 (target, source)。Reduce 函数把给定链接目标（target）的链接组合成一个列表，输出 (target, list(source))。
- 倒排索引：Map 函数分析每个文档输出一个（词，文档号）的列表，Reduce 函数的输入是一个给定词的所有（词，文档号），排序所有的文档号，输出（词，list（文档号）)。所有的输出集合形成一个简单的倒排索引，它以一种简单的算法跟踪词在文档中的位置。
- 分布式排序：Map 函数从每个记录提取 key，输出 (key, record)。Reduce 函数不改变任何的值。这个运算依赖分区机制和排序属性。

### GFS

#### 集群组成

除了客户端以外，一个 GFS 集群还包括一个 **Master** 节点和若干个 **Chunk Server**。它们会作为用户级进程运行在普通的 Linux 机器上。

在存储文件时，GFS 会把文件切分成若干个拥有固定长度的 Chunk（块）并存储。Master 在创建 Chunk 时会为它们赋予一个唯一的 64 位 Handle（句柄），并把它们移交给 Chunk Server，而 Chunk Server 则以普通文件的形式将每个 Chunk 存储在自己的本地磁盘上。为了确保 Chunk 的可用性，GFS 会把每个 Chunk 备份成若干个 Replica 分配到其他 Chunk Server 上。

GFS 的 Master 负责维护整个集群的元数据，包括集群的 Namespace（命名空间，即文件元数据）以及 Chunk Lease 管理、无用 Chunk 回收等系统级操作。Chunk Server 除了保存 Chunk 以外也会周期地和 Master 通过心跳信号进行通信，Master 也借此得以收集每个 Chunk Server 当前的状态，并向其发送指令。

鉴于整个集群只有一个 Master，客户端在和 GFS 集群通信时，首先会从 Master 处获取 GFS 的元数据，而实际文件的数据传输则会与 Chunk Server 直接进行，以避免 Master 成为整个系统的数据传输瓶颈；除此以外，客户端也会在一定时间内缓存 Master 返回的集群元数据。

#### 元数据

GFS 集群的元数据主要包括以下三类信息：

- 文件与 Chunk 的 Namespace
- 文件与 Chunk 之间的映射关系
- 每个 Chunk Replica 所在的位置

#### Chunk 的大小

GFS 选择了使用 64MB 作为 Chunk 的大小。

较大的 Chunk 主要带来了如下几个好处：

1. 降低客户端与 Master 通信的频率
2. 增大客户端进行操作时这些操作落到同一个 Chunk 上的概率
3. 减少 Master 所要保存的元数据的体积

不过，较大的 Chunk 会使得小文件占据额外的存储空间；一般的小文件通常只会占据一个 Chunk，这些 Chunk 也容易成为系统的负载热点。但正如之前所设想的需求那样，这样的文件在 Google 的场景下不是普遍存在的，这样的问题并未在 Google 中真正出现过。即便真的出现了，也可以通过提升这类文件的 Replica 数量来将负载进行均衡。

#### 数据完整性

每个 Chunk 都会以 Replica 的形式被备份在不同的 Chunk Server 中，而且用户可以为 Namespace 的不同部分赋予不同的备份策略。

为了保证数据完整，每个 Chunk Server 都会以校验和的形式来检测自己保存的数据是否有损坏；在侦测到损坏数据后，Chunk Server 也可以利用其它 Replica 来恢复数据。

### Raft

#### 节点类型

- `Leader`：集群内最多只会有一个 leader，负责发起心跳，响应客户端，创建日志，同步日志。
- `Candidate`：leader 选举过程中的临时角色，由 follower 转化而来，发起投票参与竞选。
- `Follower`：接受 leader 的心跳和日志同步数据，投票给 candidate。

在博士论文和实际生产系统中，其实又增加了两种身份：

- `Learner`：不具有选举权，参与日志复制过程但不计数的节点。可以作为新节点加入集群时的过渡状态以提升可用性，也可以作为一种类似于 binlog 的对 leader 日志流进行订阅的角色，比如可以参考 PingCAP 公司 tikv 和 tiflash 的架构。
- `Pre candidate`：刚刚发起竞选，还在等待 `Pre-Vote` 结果的临时状态，取决于 `Pre-Vote` 的结果，可能进化为 candidate，可能退化为 follower。

#### 节点状态

每一个节点都应该有的持久化状态：

- `currentTerm`：当前任期，保证重启后任期不丢失。
- `votedFor`：在当前 term，给哪个节点投了票，值为 null 或 `candidate id`。即使节点重启，Raft 算法也能保证每个任期最多只有一个 leader。
- `log[]`：已经 committed 的日志，保证状态机可恢复。

每一个节点都应该有的非持久化状态：

- `commitindex`：已提交的最大 index。leader 节点重启后可以通过 appendEntries rpc 逐渐得到不同节点的 matchIndex，从而确认 commitIndex，follower 只需等待 leader 传递过来的 commitIndex 即可。
- `lastApplied`：已被状态机应用的最大 index。raft 算法假设了状态机本身是易失的，所以重启后状态机的状态可以通过 log[] （部分 log 可以压缩为 snapshot) 来恢复。

leader 的非持久化状态：

- `nextindex[]`：为每一个 follower 保存的，应该发送的下一份 `entry index`；初始化为本地 last index + 1。
- `matchindex[]`：已确认的，已经同步到每一个 follower 的 `entry index`。初始化为 0，根据复制状态不断递增，  

    （注：每次选举后，leader 的此两个数组都应该立刻重新初始化并开始探测）

#### 任期

Raft 将时间划分成为任意不同长度的 term。term 用连续的数字进行表示。每一个 term 的开始都是一次选举，一个或多个 candidate 会试图成为 leader。如果一个 candidate 赢得了选举，它就会在该 term 担任 leader。在某些情况下，选票会被均分，即 `split vote`（例如总数为偶数节点时两个 candidate 节点各获得了两票），此时无法选出该 term 的 leader，那么在该 term 的选举超时后将会开始另一个 term 的选举。

term 在 Raft 算法中充当逻辑时钟（类似于 Lamport timestamp）的作用，这会允许服务器节点查明一些过期的信息比如过期的 leader。

每个节点都会存储当前 term 号，这一编号在整个时间内单调增长。当服务器之间通信的时候会交换当前 term 号；如果一个服务器的当前 term 号比其他人小，那么他会更新自己的 term 到较大的 term 值。如果一个 candidate 或者 leader 发现自己的 term 过期了，那么他会立即退回 follower。如果一个节点接收到一个包含过期 term 号的请求，那么它会拒绝或忽略这个请求。这实际上就是一个 Lamport 逻辑时钟的具体实现。

#### 日志

- `entry`：Raft 中，将每一个事件都称为一个 entry，每一个 entry 都有一个表明它在 log 中位置的 index（之所以从 1 开始是为了方便 `prevLogIndex` 从 0 开始）。只有 leader 可以创建 entry。entry 的内容为 `<term, index, cmd>`，其中 cmd 是可以应用到状态机的操作。在 raft 组大部分节点都接收这条 entry 后，entry 可以被称为是 committed 的。
    
- `log`：由 entry 构成的数组，只有 leader 可以改变其他节点的 log。 entry 总是先被 leader 添加进本地的 log 数组中去，然后才发起共识请求，获得 quorum 同意后才会被 leader 提交给状态机。follower 只能从 leader 获取新日志和当前的 commitIndex，然后应用对应的 entry 到自己的状态机。

#### 领导人选举

Raft 使用心跳来维持 leader 身份。任何节点都以 follower 的身份启动。 leader 会定期的发送心跳给所有的 follower 以确保自己的身份。每当 follower 收到心跳后，就刷新自己的 electionElapsed，重新计时。

（后文中，会将预设的选举超时称为 electionTimeout，而将当前经过的选举耗时称为 electionElapsed）

一旦一个 follower 在指定的时间内没有收到任何 RPC（称为 electionTimeout），则会发起一次选举。 当 follower 试图发起选举后，其身份转变为 candidate，在增加自己的 term 后， 会向所有节点发起 RequestVoteRPC 请求，candidate 的状态会一直持续直到：

- 赢得选举
- 其他节点赢得选举
- 一轮选举结束，无人胜出

选举的方式非常简单，谁能获取到多数选票 `(N/2 + 1)`，谁就成为 leader。 在一个 candidate 节点等待投票响应的时候，它有可能会收到其他节点声明自己是 leader 的心跳， 此时有两种情况：

- 该请求的 term 和自己一样或更大：说明对方已经成为 leader，自己立刻退为 follower。
- 该请求的 term 小于自己：拒绝请求并返回当前 term 以让请求节点更新 term。

为了防止在同一时间有太多的 follower 转变为 candidate 导致无法选出绝对多数， Raft 采用了随机选举超时（`randomized election timeouts`）的机制， 每一个 candidate 在发起选举后，都会随机化一个新的选举超时时间， 一旦超时后仍然没有完成选举，则增加自己的 term，然后发起新一轮选举。 在这种情况下，应该能在较短的时间内确认出 leader。 （因为 term 较大的有更大的概率压倒其他节点）

通过一个节点在一个 term 只能给一个节点投票，Raft 保证了对于给定的一个 term 最多只有一个 leader，从而避免了选举导致的 `split brain` 以确保 safety；通过不同节点每次随机化选举超时时间，Raft 在实践中（注意：并没有在理论上）避免了活锁以确保 liveness。

#### 日志同步

leader 被选举后，则负责所有的客户端请求。每一个客户端请求都包含一个命令，该命令可以被作用到 RSM。

leader 收到客户端请求后，会生成一个 entry，包含 `<index, term, cmd>`，再将这个 entry 添加到自己的日志末尾后，向所有的节点广播该 entry。

follower 如果同意接受该 entry，则在将 entry 添加到自己的日志后，返回同意。

如果 leader 收到了多数的成功答复，则将该 entry 应用到自己的 RSM，之后可以称该 entry 是 committed 的。该 committed 信息会随着随后的 AppendEntries 或 Heartbeat RPC 被传达到其他节点。

Raft 保证下列两个性质：

- 如果在两个日志（节点）里，有两个 entry 拥有相同的 index 和 term，那么它们一定有相同的 cmd；
- 如果在两个日志（节点）里，有两个 entry 拥有相同的 index 和 term，那么它们前面的 entry 也一定相同。

#### 选举限制

因为 leader 的强势地位，所以 Raft 在投票阶段就确保选举出的 leader 一定包含了整个集群中目前已 committed 的所有日志。

当 candidate 发送 RequestVoteRPC 时，会带上最后一个 entry 的信息。 所有的节点收到该请求后，都会比对自己的日志，如果发现自己的日志更新一些，则会拒绝投票给该 candidate。 （Pre-Vote 同理，如果 follower 认为 Pre-Candidate 没有资格的话，会拒绝 PreVote）

判断日志新旧的方式：获取请求的 entry 后，比对自己日志中的最后一个 entry。首先比对 term，如果自己的 term 更大，则拒绝请求。如果 term 一样，则比对 index，如果自己的 index 更大（说明自己的日志更长），则拒绝请求。

#### 节点崩溃

如果 leader 崩溃，集群中的所有节点在 electionTimeout 时间内没有收到 leader 的心跳信息就会触发新一轮的选主。总而言之，最终集群总会选出唯一的 leader 。按论文中的说法，计算一次 RPC 耗时高达 `30～40ms` 时，`99.9%` 的选举依然可以在 `3s` 内完成，但一般一个机房内一次 RPC 只需 1ms。当然，选主期间整个集群对外是不可用的。

如果 follower 和 candidate 奔溃相对而言就简单很多，因为 Raft 所有的 RPC 都是幂等的，所以 Raft 中所有的请求，只要超时，就会无限的重试。follower 和 candidate 崩溃恢复后，可以收到新的请求，然后按照上面谈论过的追加或拒绝 entry 的方式处理请求。

#### 日志压缩

Raft 的日志在正常运行期间会增长以合并更多的客户请求，但是在实际的系统中，Raft 的日志无法不受限制地增长。随着日志的增长，日志会占用更多空间，并且需要花费更多时间进行重放。如果没有某种机制可以丢弃日志中累积的过时信息，这最终将导致可用性问题。因此需要定时去做 snapshot。

snapshot 会包括：

- 状态机当前的状态。
- 状态机最后一条应用的 entry 对应的 index 和 term。
- 集群最新配置信息。
- 为了保证 exactly-once 线性化语义的去重表。

各个节点自行择机完成自己的 snapshot 即可，如果 leader 发现需要发给某一个 follower 的 nextIndex 已经被做成了 snapshot，则需要将 snapshot 发送给该 follower。注意 follower 拿到非过期的 snapshot 之后直接覆盖本地所有状态即可，不需要留有部分 entry，也不会出现 snapshot 之后还存在有效的 entry。因此 follower 只需要判断 `InstallSnapshot RPC` 是否过期即可。过期则直接丢弃，否则直接替换全部状态即可。

snapshot 可能会带来两个问题：

1. 做 snapshot 的策略？  
    一般为定时或者定大小，达到阈值即做 snapshot，做完后对状态机和 raft log 进行原子性替换即可。
    
2. 做 snapshot 时是否还可继续提供写请求？  
    一般情况下，做 snapshot 期间需要保证状态机不发生变化，也就是需要保证 snapshot 期间状态机不处理写请求。当然 raft 层依然可以去同步，只是状态机不能变化，即不能 apply 新提交的日志到状态机中而已。要想做的更好，可以对状态机采用 `copy-on-write` 的复制来不阻塞写请求。

#### 禅让

有时候，会希望取消当前 leader 的管理权，比如：

- leader 节点因为运维原因需要重启；
- 有其他更适合当 leader 的节点；

直接将 leader 节点停机的话，其他节点会等待 electionTimeout 后进入选举状态， 这期间会集群会停止响应。为了避免这一段不可用的时间，可以采用禅让机制（`leadership transfer`）。

禅让的步骤为：

1. leader 停止响应客户端请求；
2. leader 向 target 节点发起一次日志同步；
3. leader 向 target 发起一次 TimeoutNowRPC，target 收到该请求后立刻发起一轮投票。

#### 预投票

一个暂时脱离集群网络的节点，在重新加入集群后会干扰到集群的运行。

因为当一个节点和集群失去联系后，在等待 electionTimeout 后，它就会增加自己的 term 并发起选举， 因为联系不上其他节点，所以在 electionTimeout 后，它会继续增加自己的 term 并继续发起选举。

一段时间以后，它的 term 就会显著的高于原集群的 term。如果此后该节点重新和集群恢复了联络， 它的高 term 会导致 leader 立刻退位，并重新举行选举。

为了避免这一情形，引入了 Pre-Vote 的机制。在该机制下，一个 candidate 必须在获得了多数赞同的情形下， 才会增加自己的 term。一个节点在满足下述条件时，才会赞同一个 candidate：

- 该 candidate 的日志足够新；
- 当前节点已经和 leader 失联（electionTimeout）。

也就是说，candidate 会先发起一轮 Pre-Vote，获得多数同意后，更新自己的 term， 再发起一轮 RequestVoteRPC。

这种情形下，脱离集群的节点，只会不断的发起 Pre-Vote，而不会更新自己的 term。