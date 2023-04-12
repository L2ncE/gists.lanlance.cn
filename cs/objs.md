## 对象存储

### 1. 何为对象存储？

对象存储服务（Object Storage Service，OSS）是一种海量、安全、低成本、高可靠的云存储服务，适合存放任意类型的文件。容量和处理能力弹性扩展，多种存储类型供选择，全面优化存储成本。

### 2. MinIO 基础概念

- Object：存储到 Minio 的基本对象，如文件、字节流，Anything…

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
