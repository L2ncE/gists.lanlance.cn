## 对象存储

### 1. 何为对象存储？

对象存储服务（Object Storage Service，OSS）是一种海量、安全、低成本、高可靠的云存储服务，适合存放任意类型的文件。容量和处理能力弹性扩展，多种存储类型供选择，全面优化存储成本。

### 2. MinIO 基础概念

- Object：存储到 MinIO 的基本对象，如文件、字节流，Anything…

- Bucket：用来存储 Object 的逻辑空间。每个 Bucket 之间的数据是相互隔离的。对于客户端而言，就相当于一个存放文件的顶层文件夹。

- Drive：即存储数据的磁盘，在 MinIO 启动时，以参数的方式传入。Minio 中所有的对象数据都会存储在 Drive 里。

- Set：即一组 Drive 的集合，分布式部署根据集群规模自动划分一个或多个 Set ，每个 Set 中的 Drive 分布在不同位置。一个对象存储在一个 Set 上。（For example: {1…64} is divided into 4 sets each of size 16.）

一个对象存储在一个 Set 上

一个集群划分为多个 Set

一个 Set 包含的 Drive 数量是固定的，默认由系统根据集群规模自动计算得出

一个 Set 中的 Drive 尽可能分布在不同的节点上

### 3. MinIO 的数据高可靠

Minio 使用了 Erasure Code 纠删码和 Bit Rot Protection 数据腐化保护这两个特性，所以 MinIO 的数据可靠性做的高。

Minio 纠删码可以在丢失一半的盘的情况下，仍可以保证数据安全。 而且 Minio 纠删码是作用在对象级别，可以一次恢复一个对象

### 4. SeaweedFS 特点

SeaweedFS 最初作为一个对象存储来有效地处理**小文件**。中央主服务器（master）只管理文件卷（volume），而不是管理中央主服务器中的所有文件元数据，它允许这些卷服务器管理文件及其元数据。这减轻了中央主服务器的并发压力，并将文件元数据传播到卷服务器，允许更快的文件访问(只需一个磁盘读取操作)。每个文件的元数据只有40字节的磁盘存储开销。使用 O(1)磁盘读取。

### 5. SeaweedFS Master 原理

Master Server 集群之间的副本同步是基于 Raft 协议。

Master Server 保存整个 Volume Server 的拓扑信息，结构: Topology -> Data Center-> Rack -> Data Node -> Volume。可以通过 Data Center-> Rack -> Data Node 查找一个 Data Node 上的所有 Volume 信息。

另外，也需要根据 Volume 去查找它所在的所有节点，所以 Leader Master 还维护着另一个结构: Topology -> Collection -> VolumeLayout-> Data Node List 其中，Collection 对 VolumeLayout 的组织是按备份方式分类区分，VolumeLayout 保存每个 volume id 到其具体位置 Data Node List 的映射，同时保存了 Volume 的可读写性质。

Leader Master 跟各个 Volume Server 通过心跳保持连接, Volume Server 通过心跳将本地的卷信息(增、删、过期等)上报给 Leader Master。

然后 Leader Master 再将 Volume 的位置信息通过 gRPC 同步 _（KeepConnected）_ 给其余 Master。所以, 当查询一个 Volume 的具体位置时，Leader Master 直接从本地的 Topology 中读取, 非 Leader Master 则从本地 Master Client 的 vidMap 中获取数据。

当要上传文件时，Leader Master 还负责分配一个全局唯一且递增的 id 作为 file id。

### 6. SeaweedFS Volume 原理

volume server 是文件实际存储的位置, 对文件的组织是按如下格式进行的：Store -> DiskLocations -> Volumes -> Needles

其中 DiskLocations 对应不同的目录, 每个目录中有很多 Volumes, 每个 Volume 实际是一个硬盘文件 .dat，其中分段保存一批上传的业务文件, 为了快速定位一个业务文件在 .dat 中的位置, 每个 Volume 对应个 .idx 文件, 用于保存所有业务文件在 .dat 文件中的偏移量和文件大小。为加快查询索引速度, Volume Server 启动时会将 .idx 读到 LevelDb 中，虽然一个业务文件在多个 Volume Server 作为主备, 但各 Volume Server 是对等的，平等接收外部请求, 当收到上传业务文件后, 会将文件传到其它机器。