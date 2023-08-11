## 云原生

### 1. Docker 是什么？

- 是实现容器技术的一种工具
- 是一个开源的应用容器引擎
- 使用 C/S 架构模式，通过远程 API 来管理
- 可以打包一个应用及依赖包到一个轻量级、可移植的容器中

### 2. 虚拟化是什么？

- 可以理解成虚拟机技术
- 一个主机可以部署多个虚拟机，每个虚拟机又可以部署多个应用
- 对于主机来说，虚拟机就是一个普通文件

### 3. 虚拟化的缺点是什么？

- 资源占用多：每个虚拟机都是完整的操作系统，需要给它分配大量系统资源
- 冗余步骤多：一个完整的操作系统，一些系统级别的步骤无法避免，比如用户登录
- 启动慢：启动操作系统需要多久，启动虚拟机就要多久

### 4. Docker 有什么优势？

- 资源占用少：每个容器都共享主机的资源，容器需要多少就用多少
- 启动快：一条命令即可将容器启动，而容器启动时一般会将服务或应用一并启动

### 5. Docker 与虚拟化的不同

1. 启动速度不同

docker 启动快速属于秒级别。虚拟机通常需要几分钟去启动。

2. 性能损耗不同

docker 需要的资源更少，docker 在操作系统级别进行虚拟化，docker 容器和内核交互，几乎没有性能损耗，性能优于通过 Hypervisor 层与内核层的虚拟化。

3. 系统利用率不同

docker 更轻量，docker 的架构可以共用一个内核与共享应用程序库，所占内存极小。同样的硬件环境，Docker 运行的镜像数远多于虚拟机数量，对系统的利用率非常高。

4. 隔离性不同

与虚拟机相比，docker 隔离性更弱，docker 属于进程之间的隔离，虚拟机可实现系统级别隔离。

5. 安全性不同

docker 的安全性也更弱。Docker 的租户 root 和宿主机 root 等同，一旦容器内的用户从普通用户权限提升为 root 权限，它就直接具备了宿主机的 root 权限，进而可进行无限制的操作。虚拟机租户 root 权限和宿主机的 root 虚拟机权限是分离的。

1. 可管理性不同

docker 的集中化管理工具还不算成熟。各种虚拟化技术都有成熟的管理工具，例如 VMware vCenter 提供完备的虚拟机管理能力。

7. 可用和可恢复性不同

docker 对业务的高可用支持是通过快速重新部署实现的。虚拟化具备负载均衡，高可用，容错，迁移和数据保护等经过生产实践检验的成熟保障机制，VMware 可承诺虚拟机 99.999% 高可用，保证业务连续性。

8. 创建、删除速度不同

虚拟化创建是分钟级别的，Docker 容器创建是秒级别的，Docker 的快速迭代性，决定了无论是开发、测试、部署都可以节约大量时间。

9. 交付、部署速度不同

虚拟机可以通过镜像实现环境交付的一致性，但镜像分发无法体系化；Docker 在 Dockerfile 中记录了容器构建过程，可在集群中实现快速分发和快速部署;

#### 虚拟化

虚拟化帮助我们在单个物理服务器上运行和托管多个操作系统。在虚拟化中，管理程序为客户操作系统提供了一个虚拟机。VM 形成了硬件层的抽象，因此主机上的每个 VM 都可以充当物理机。

#### 容器化

容器化为我们提供了一个独立的环境来运行我们的应用程序。我们可以在单个服务器或 VM 上使用相同的操作系统部署多个应用程序。容器构成了应用层的抽象，所以每个容器代表一个不同的应用。

### 6. 什么是镜像？

- 创建容器的模板
- 同一个镜像可以创建多个不同的容器

### 7. 什么是容器？

- 通过镜像生成的运行实例
- 不同容器之间是相互隔离，独立运行的
- 通常一个容器就是一个应用或一个服务，也是我们常说的微服务

### 8. Docker 如何做到资源隔离

Namespace 对内核资源进行隔离，使得容器中的进程到可以在单独的命名空间中运行，并且只可访问当前容器命名空间的资源。

### 9. OpenTelemetry 是什么

OpenTelemetry 是一组 API、SDK、工具和集成，旨在创建和管理遥测数据，例如 Trace、Metrics 和 Logs。该项目提供了一个与供应商无关的实现，可以将其配置为将遥测数据发送到您选择的后端。

### 10. Jaeger 是什么

Jaeger 是用于追踪分布式服务之间事务的开源软件，它为微服务场景而生。它主要用于分析多个服务的调用过程，图形化服务调用轨迹，是诊断性能问题、分析系统故障的利器。

### 11. Prometheus 是什么

Prometheus 是一个开源的系统监控和报警系统

样本：在时间序列中的每一个点称为一个样本（sample），样本由以下三部分组成：

- 指标（metric）：指标名称和描述当前样本特征的 labelsets；
- 时间戳（timestamp）：一个精确到毫秒的时间戳；
- 样本值（value）： 一个 folat64 的浮点型数据表示当前样本的值。

### 12. Dockerfile 常用指令

- FROM：指定基础镜像。
- MAINTAINER：指定镜像创建者信息。
- RUN：在新的镜像内部运行命令。
- CMD：指定容器启动时要运行的命令。
- EXPOSE：声明容器运行时需要监听的端口。
- ENV：设置环境变量。
- ADD：将文件或目录复制到容器中。
- COPY：将文件或目录复制到容器中。
- ENTRYPOINT：配置容器启动后执行的命令和参数。
- VOLUME：创建一个可以从本地主机或其他容器挂载的挂载点。

### 13. Docker 数据卷

Docker 数据卷是一个可供一个或多个容器使用的特殊目录，它绕过了 UFS，可以提供很多有用的特性：

- 数据卷可以在容器之间共享和重用。
- 对数据卷的修改会立马生效。
- 对数据卷的更新，不会影响镜像。
- 数据卷默认会一直存在，即使容器被删除。

Docker 数据卷有两种创建方式：在创建容器时创建数据卷和先创建好数据卷，然后在创建容器时挂载这个数据卷。

### 14. K8s 组件

#### Master

- apiserver 是 Master 节点——同时也是整个 Kubernetes 系统的唯一入口，它对外公开了一系列的 RESTful API，并且加上了验证、授权等功能，所有其他组件都只能和它直接通信，可以说是 Kubernetes 里的联络员。
- etcd 是一个高可用的分布式 Key-Value 数据库，用来持久化存储系统里的各种资源对象和状态，相当于 Kubernetes 里的配置管理员。注意它只与 apiserver 有直接联系，也就是说任何其他组件想要读写 etcd 里的数据都必须经过 apiserver。
- scheduler 负责容器的编排工作，检查节点的资源状态，把 Pod 调度到最适合的节点上运行，相当于部署人员。因为节点状态和 Pod 信息都存储在 etcd 里，所以 scheduler 必须通过 apiserver 才能获得。
- controller-manager 负责维护容器和节点等资源的状态，实现故障检测、服务迁移、应用伸缩等功能，相当于监控运维人员。同样地，它也必须通过 apiserver 获得存储在 etcd 里的信息，才能够实现对资源的各种操作。

#### Node

- kubelet 是 Node 的代理，负责管理 Node 相关的绝大部分操作，Node 上只有它能够与 apiserver 通信，实现状态报告、命令下发、启停容器等功能，相当于是 Node 上的一个“小管家”。
- kube-proxy 的作用有点特别，它是 Node 的网络代理，只负责管理容器的网络通信，简单来说就是为 Pod 转发 TCP/UDP 数据包，相当于是专职的“小邮差”。
- container-runtime 我们就比较熟悉了，它是容器和镜像的实际使用者，在 kubelet 的指挥下创建容器，管理 Pod 的生命周期，是真正干活的“苦力”。

### 15. K8s 不同功能会放在同一个容器中吗

K8s 中不同的功能可以放在同一个容器中，但是这并不是最佳实践。通常情况下，每个容器应该只负责一个进程或服务，这样可以更好地管理和维护容器。如果将多个服务放在同一个容器中，那么当其中一个服务出现问题时，可能会影响到其他服务的正常运行。

### 16. K8s 声明式 api 和普通 api 有什么差异

K8s 的 API 有两种类型：声明式 API 和命令式 API。声明式 API 是一种更加高级的 API，它允许用户通过 YAML 文件来描述所需的状态，而不是通过命令来创建或更新对象。声明式 API 会自动检测对象的状态，并根据所需的状态进行调整。命令式 API 则是通过命令行工具或客户端库来创建或更新对象，它需要用户手动指定所需的状态。

### 17. Dockerfile 合并多个 RUN 指令的原因

当我们编写 Dockerfile 时，可以合并多个 RUN 指令，减少不必要的镜像层的产生，并且在之后将多余的命令清理干净，只保留运行时需要的依赖。

比如，下面这个 Dockerfile：

```dockerfile
FROM ubuntu

RUN apt-get install vim -y

RUN apt-get remove vim -y
```

虽然这个操作创建的镜像中没有安装 Vim，但是镜像的大小和有 Vim 是一样的。原因就是，每条指令都会新加一个镜像层，执行 install vim 后添加了一层，执行 remove vim 后也会添加一层，而这一删除命令并不会减少整个镜像的大小。

### 18. docker stats 命令

docker stats 命令用于监视容器的实时资源使用情况，包括 CPU、内存、网络和磁盘等方面的信息。通过该命令，可以查看每个容器的 CPU 利用率、内存使用量、网络传输速度以及磁盘读写速度等指标，在容器出现性能问题时，可以帮助开发人员快速地定位问题所在。

使用 docker stats 命令可以列出正在运行的所有容器的实时资源使用情况，也可以通过指定容器名称或 ID 来获取特定容器的实时资源使用情况。默认情况下，docker stats 命令会每秒钟更新一次容器的资源使用情况，并按照容器 ID 或名称进行排序显示。

### 19. 容器技术原理

#### chroot

chroot 就是可以改变某进程的根目录，使这个程序不能访问目录之外的其他目录，这个跟我们在一个容器中是很相似的。

#### Namespace

Namespace 是 Linux 内核的一项功能，该功能对内核资源进行隔离，使得容器中的进程都可以在单独的命名空间中运行，并且只可以访问当前容器命名空间的资源。Namespace 可以隔离进程 ID、主机名、用户 ID、文件名、网络访问和进程间通信等相关资源。

Docker 主要用到以下五种命名空间。

- pid namespace：用于隔离进程 ID。
- net namespace：隔离网络接口，在虚拟的 net namespace 内用户可以拥有自己独立的 IP、路由、端口等。
- mnt namespace：文件系统挂载点隔离。
- ipc namespace：信号量,消息队列和共享内存的隔离。
- uts namespace：主机名和域名的隔离。

#### Cgroups

Cgroups 是一种 Linux 内核功能，可以限制和隔离进程的资源使用情况（CPU、内存、磁盘 I/O、网络等）。在容器的实现中，Cgroups 通常用来限制容器的 CPU 和内存等资源的使用。

#### 联合文件系统

联合文件系统，又叫 UnionFS，是一种通过创建文件层进程操作的文件系统，因此，联合文件系统非常轻快。Docker 使用联合文件系统为容器提供构建层，使得容器可以实现写时复制以及镜像的分层构建和存储。常用的联合文件系统有 AUFS、Overlay 和 Devicemapper 等。

### 20. Docker 核心概念

#### 镜像

通俗地讲，它是一个只读的文件和文件夹组合。它包含了容器运行时所需要的所有基础文件和配置信息，是容器启动的基础。

#### 容器

容器是镜像的运行实体。镜像是静态的只读文件，而容器带有运行时需要的可写文件层，并且容器中的进程属于运行状态。即**容器运行着真正的应用进程。容器有初建、运行、停止、暂停和删除五种状态。**

虽然容器的本质是主机上运行的一个进程，但是容器有自己独立的命名空间隔离和资源限制。也就是说，在容器内部，无法看到主机上的进程、环境变量、网络等信息，这是容器与直接运行在主机上进程的本质区别。

### 21. Docker 核心组件

- runC 是 Docker 官方按照 OCI 容器运行时标准的一个实现。通俗地讲，runC 是一个用来运行容器的轻量级工具，是真正用来运行容器的。
- containerd 是 Docker 服务端的一个核心组件，它是从 dockerd 中剥离出来的，它的诞生完全遵循 OCI 标准，是容器标准化后的产物。containerd 通过 containerd-shim 启动并管理 runC，可以说 containerd 真正管理了容器的生命周期。

`dockerd` 通过 gRPC 与 `containerd` 通信，由于 `dockerd` 与真正的容器运行时，`runC` 中间有了 `containerd` 这一 OCI 标准层，使得 `dockerd` 可以确保接口向下兼容。

### 21. Dockerfile 构建镜像特性

- Dockerfile 的每一行命令都会生成一个独立的镜像层，并且拥有唯一的 ID；
- Dockerfile 的命令是完全透明的，通过查看 Dockerfile 的内容，就可以知道镜像是如何一步步构建的；
- Dockerfile 是纯文本的，方便跟随代码一起存放在代码仓库并做版本管理。

分层的结构使得 Docker 镜像非常轻量，每一层根据镜像的内容都有一个唯一的 ID 值，当不同的镜像之间有相同的镜像层时，便可以实现不同的镜像之间共享镜像层的效果。

### 21. 使用 Dockerfile 进行构建的好处

- 易于版本化管理，Dockerfile 本身是一个文本文件，方便存放在代码仓库做版本管理，可以很方便地找到各个版本之间的变更历史；
- 过程可追溯，Dockerfile 的每一行指令代表一个镜像层，根据 Dockerfile 的内容即可很明确地查看镜像的完整构建过程；
- 屏蔽构建环境异构，使用 Dockerfile 构建镜像无须考虑构建环境，基于相同 Dockerfile 无论在哪里运行，构建结果都一致。

### 22. CMD 和 ENTRYPOINT

- Dockerfile 中如果使用了 `ENTRYPOINT` 指令，启动 Docker 容器时需要使用 --entrypoint 参数才能覆盖 Dockerfile 中的 `ENTRYPOINT` 指令，而使用 `CMD` 设置的命令则可以被 `docker run` 后面的参数直接覆盖。
- `ENTRYPOINT` 指令可以结合 `CMD` 指令使用，也可以单独使用，而 `CMD` 指令只能单独使用。

如果你希望你的镜像足够灵活，推荐使用 `CMD` 指令。如果你的镜像只执行单一的具体程序，并且不希望用户在执行 `docker run` 时覆盖默认程序，建议使用 `ENTRYPOINT`。

最后再强调一下，无论使用 `CMD` 还是 `ENTRYPOINT`，都尽量使用 `exec` 模式。

### 23. Docker 的安全问题

- 镜像安全
- Linux 内核隔离性不够

	尽管目前 Namespace 已经提供了非常多的资源隔离类型，但是仍有部分关键内容没有被完全隔离，其中包括一些系统的关键性目录（如 /sys、/proc 等），这些关键性的目录可能会泄露主机上一些关键性的信息，让攻击者利用这些信息对整个主机甚至云计算中心发起攻击。

- 所有容器共享主机内核

### 24. 为什么 Docker 需要 Namespace

当 Docker 新建一个容器时，它会创建六种 Namespace，然后将容器中的进程加入这些 Namespace 之中，使得 Docker 容器中的进程只能看到当前 Namespace 中的系统资源。

正是由于 Docker 使用了 Linux 的这些 Namespace 技术，才实现了 Docker 容器的隔离，可以说没有 Namespace，就没有 Docker 容器。

### 25. Cgroups 功能及核心概念

#### 功能

- 资源限制： 限制资源的使用量，例如我们可以通过限制某个业务的内存上限，从而保护主机其他业务的安全运行。
- 优先级控制：不同的组可以有不同的资源（ CPU 、磁盘 IO 等）使用优先级。
- 审计：计算控制组的资源使用情况。
- 控制：控制进程的挂起或恢复。

#### 概念

- 子系统（subsystem）：是一个内核的组件，一个子系统代表一类资源调度控制器。例如内存子系统可以限制内存的使用量，CPU 子系统可以限制 CPU 的使用时间。
- 控制组（cgroup）：表示一组进程和一组带有参数的子系统的关联关系。例如，一个进程使用了 CPU 子系统来限制 CPU 的使用时间，则这个进程和 CPU 子系统的关联关系称为控制组。
- 层级树（hierarchy）：是由一系列的控制组按照树状结构排列组成的。这种排列方式可以使得控制组拥有父子关系，子控制组默认拥有父控制组的属性，也就是子控制组会继承于父控制组。比如，系统中定义了一个控制组 c1，限制了 CPU 可以使用 1 核，然后另外一个控制组 c2 想实现既限制 CPU 使用 1 核，同时限制内存使用 2G，那么 c2 就可以直接继承 c1，无须重复定义 CPU 限制。

cgroups 的三个核心概念中，子系统是最核心的概念，因为子系统是真正实现某类资源的限制的基础。

### 26. Docker 相关的组件

- docker 

docker 是 Docker 客户端的一个完整实现，它是一个二进制文件，对用户可见的操作形式为 docker 命令，通过 docker 命令可以完成所有的 Docker 客户端与服务端的通信

- dockerd

dockerd 是 Docker 服务端的后台常驻进程，用来接收客户端发送的请求，执行具体的处理任务，处理完成后将结果返回给客户端。

- docker-init

在容器内部，当我们自己的业务进程没有回收子进程的能力时，在执行 docker run 启动容器时可以添加 --init 参数，此时 Docker 会使用 docker-init 作为 1 号进程，帮你管理容器内子进程，例如回收僵尸进程等。

- docker-proxy

docker-proxy 主要是用来做端口映射的。当我们使用 docker run 命令启动容器时，如果使用了 -p 参数，docker-proxy 组件就会把容器内相应的端口映射到主机上来，底层是依赖于 iptables 实现的。

### 27. Containerd 相关组件

- containerd

[containerd](https://github.com/containerd/containerd) 组件是从 Docker 1.11 版本正式从 dockerd 中剥离出来的，它的诞生完全遵循 OCI 标准，是容器标准化后的产物。containerd 完全遵循了 OCI 标准，并且是完全社区化运营的，因此被容器界广泛采用。

containerd 不仅负责容器生命周期的管理，同时还负责一些其他的功能：

- 镜像的管理，例如容器运行前从镜像仓库拉取镜像到本地；
- 接收 dockerd 的请求，通过适当的参数调用 runc 启动容器；
- 管理存储相关资源；
- 管理网络相关资源。

containerd 包含一个后台常驻进程，默认的 socket 路径为 /run/containerd/containerd.sock，dockerd 通过 UNIX 套接字向 containerd 发送请求，containerd 接收到请求后负责执行相关的动作并把执行结果返回给 dockerd。

如果你不想使用 dockerd，也可以直接使用 containerd 来管理容器，由于 containerd 更加简单和轻量，生产环境中越来越多的人开始直接使用 containerd 来管理容器。

- containerd-shim

containerd-shim 的主要作用是将 containerd 和真正的容器进程解耦，使用 containerd-shim 作为容器进程的父进程，从而实现重启 containerd 不影响已经启动的容器进程。

- ctr

ctr 实际上是 containerd-ctr，它是 containerd 的客户端，主要用来开发和调试，在没有 dockerd 的环境中，ctr 可以充当 docker 客户端的部分角色，直接向 containerd 守护进程发送操作容器的请求。

### 28. 容器运行时组件 runc

runc 是一个标准的 OCI 容器运行时的实现，它是一个命令行工具，可以直接用来创建和运行容器。负责真正意义上创建和启动容器。

### 29.  Libnetwork 常见网络模式

1. null 空网络模式：可以帮助我们构建一个没有网络接入的容器环境，以保障数据安全。
2. bridge 桥接模式：可以打通容器与容器间网络通信的需求。
3. host 主机网络模式：可以让容器内的进程共享主机网络，从而监听或修改主机网络。
4. container 网络模式：可以将两个容器放在同一个网络命名空间内，让两个业务通过 localhost 即可实现访问。

**bridge 桥接模式是 Docker 的默认网络模式，当我们创建容器时不指定任何网络模式，Docker 启动容器默认的网络模式为 bridge。**

### 30. Docker 卷的实现原理

Docker 卷的实现原理是在主机的 /var/lib/docker/volumes 目录下，根据卷的名称创建相应的目录，然后在每个卷的目录下创建 data 目录，在容器启动时如果使用 --mount 参数，Docker 会把主机上的目录直接映射到容器的指定目录下，实现数据持久化。

### 31. Kubernetes 生成对象的 YAML 模版

使用参数 `--dry-run=client -o yaml` 可以生成对象的 YAML 模板，简化编写工作。

### 32. 如何理解 Kubernetes 中的 Pod

**为了解决多应用联合运行的问题，同时还要不破坏容器的隔离，就需要在容器外面再建立一个“收纳舱”**，让多个容器既保持相对独立，又能够小范围共享网络、存储等资源，而且永远是“绑在一起”的状态。

Kubernetes 让 Pod 去编排处理容器，然后把 Pod 作为应用调度部署的**最小单位**，Pod 也因此成为了 Kubernetes 世界里的“原子”。
#### 如何使用 YAML 描述 Pod

```yml
# kubectl apply -f ngx-pod.yml
# kubectl logs ngx-pod
# kubectl delete -f ngx-pod.yaml
# kubectl delete pod ngx-pod

# kubectl explain pod.spec
# kubectl explain pod.spec.containers
# kubectl explain pod.spec.containers.env

apiVersion: v1
kind: Pod
metadata:
  name: ngx-pod
  labels:
    env: demo
    owner: L2ncE

spec:
  containers:
  - image: nginx:alpine
    name: ngx
    ports:
    - containerPort: 80
```

### 33. 进入/拷贝文件到 Pod

```sh
kubectl cp a.txt ngx-pod:/tmp
kubectl exec -it ngx-pod -- sh
```

### 34. Job/CronJob

“离线业务”可以分为两种。一种是“**临时任务**”，跑完就完事了，下次有需求了说一声再重新安排；另一种是“**定时任务**”，可以按时按点周期运行，不需要过多干预。

对应到 Kubernetes 里，“临时任务”就是 API 对象**Job**，“定时任务”就是 API 对象**CronJob**，使用这两个对象你就能够在 Kubernetes 里调度管理任意的离线业务了。

#### 如何使用 YAML 描述 Job

- apiVersion 不是 `v1`，而是 `batch/v1`。
- kind 是 `Job`，这个和对象的名字是一致的。
- metadata 里仍然要有 `name` 标记名字，也可以用 `labels` 添加任意的标签。

```yml
# kubectl apply -f job.yml

apiVersion: batch/v1
kind: Job
metadata:
  name: echo-job

spec:
  template:
    spec:
      restartPolicy: OnFailure
      containers:
      - image: busybox
        name: echo-job
        imagePullPolicy: IfNotPresent
        command: ["/bin/echo"]
        args: ["hello", "world"]
```

你会注意到 Job 的描述与 Pod 很像，但又有些不一样，主要的区别就在“spec”字段里，多了一个 `template` 字段，然后又是一个“spec”，显得有点怪。

它其实就是在 Job 对象里应用了组合模式，`template` 字段定义了一个“**应用模板**”，里面嵌入了一个 Pod，这样 Job 就可以从这个模板来创建出 Pod。

而这个 Pod 因为受 Job 的管理控制，不直接和 apiserver 打交道，也就没必要重复 apiVersion 等“头字段”，只需要定义好关键的 `spec`，描述清楚容器相关的信息就可以了，可以说是一个“无头”的 Pod 对象。

列出几个控制离线作业的重要字段：
- **activeDeadlineSeconds**，设置 Pod 运行的超时时间。
- **backoffLimit**，设置 Pod 的失败重试次数。
- **completions**，Job 完成需要运行多少个 Pod，默认是 1 个。
- **parallelism**，它与 completions 相关，表示允许并发运行的 Pod 数量，避免过多占用资源。

要注意这 4 个字段并不在 `template` 字段下，而是在 `spec` 字段下，所以它们是属于 Job 级别的，用来控制模板里的 Pod 对象。

#### 如何使用 YAML 描述 CronJob

```yml
# kubectl apply -f cronjob.yml

apiVersion: batch/v1
kind: CronJob
metadata:
  name: echo-cj

spec:
  schedule: '*/1 * * * *'
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - image: busybox
            name: echo-cj
            imagePullPolicy: IfNotPresent
            command: ["/bin/echo"]
            args: ["hello", "world"]
```

我们还是重点关注它的 `spec` 字段，你会发现它居然连续有三个 `spec` 嵌套层次：

- 第一个 `spec` 是 CronJob 自己的对象规格声明
- 第二个 `spec` 从属于“jobTemplate”，它定义了一个 Job 对象。
- 第三个 `spec` 从属于“template”，它定义了 Job 里运行的 Pod。

### 35. ConfigMap/Secret

应用程序有很多类别的配置信息，但从数据安全的角度来看可以分成两类：

- 一类是明文配置，也就是不保密，可以任意查询修改，比如服务端口、运行参数、文件路径等等。
- 另一类则是机密配置，由于涉及敏感信息需要保密，不能随便查看，比如密码、密钥、证书等等。

这两类配置信息本质上都是字符串，只是由于安全性的原因，在存放和使用方面有些差异，所以 Kubernetes 也就定义了两个 API 对象，**ConfigMap**用来保存明文配置，**Secret**用来保存秘密配置。

#### 如何使用 YAML 描述 ConfigMap

```yml
# kubectl apply  -f cm.yml
# kubectl delete -f cm.yml

apiVersion: v1
kind: ConfigMap
metadata:
  name: info

data:
  count: '10'
  debug: 'on'
  path: '/etc/systemd'
  greeting: |
    say hello to kubernetes.
```

既然 ConfigMap 要存储数据，我们就需要用另一个含义更明确的字段“**data**”。

#### 如何使用 YAML 描述 Secret

了解了 ConfigMap 对象，我们再来看 Secret 对象就会容易很多，它和 ConfigMap 的结构和用法很类似，不过在 Kubernetes 里 Secret 对象又细分出很多类，比如：

- 访问私有镜像仓库的认证信息
- 身份识别的凭证信息
- HTTPS 通信的证书和私钥
- 一般的机密信息（格式由用户自行解释）

```yml
# echo -n "123456" | base64 # MTIzNDU2
# echo -n "mysql" | base64  # bXlzcWw=

# kubectl apply  -f secret.yml
# kubectl delete -f secret.yml

apiVersion: v1
kind: Secret
metadata:
  name: user

data:
  name: cm9vdA==
  pwd: MTIzNDU2
  db: bXlzcWw=
```

#### 如何使用

因为 ConfigMap 和 Secret 只是一些存储在 etcd 里的字符串，所以如果想要在运行时产生效果，就必须要以某种方式“**注入**”到 Pod 里，让应用去读取。在这方面的处理上 Kubernetes 和 Docker 是一样的，也是两种途径：**环境变量**和**加载文件**。

##### 环境变量

```yml
# kubectl apply -f env-pod.yml
# kubectl get pod
# kubectl exec -it env-pod -- sh

# in Pod:
# echo $COUNT
# echo $GREETING
# echo $USERNAME
# echo $PASSWORD

apiVersion: v1
kind: Pod
metadata:
  name: env-pod

spec:
  containers:
  - env:
      - name: COUNT
        valueFrom:
          configMapKeyRef:
            name: info
            key: count
      - name: GREETING
        valueFrom:
          configMapKeyRef:
            name: info
            key: greeting
      - name: USERNAME
        valueFrom:
          secretKeyRef:
            name: user
            key: name
      - name: PASSWORD
        valueFrom:
          secretKeyRef:
            name: user
            key: pwd

    image: busybox
    name: busy
    imagePullPolicy: IfNotPresent
    command: ["/bin/sleep", "300"]

```

“**valueFrom**”字段指定了环境变量值的来源，可以是“**configMapKeyRef**”或者“**secretKeyRef**”，然后你要再进一步指定应用的 ConfigMap/Secret 的“**name**”和它里面的“**key**”，要当心的是这个“name”字段是 API 对象的名字，而不是 Key-Value 的名字。

##### Volume

在 Pod 里挂载 Volume 很容易，只需要在“**spec**”里增加一个“**volumes**”字段，然后再定义卷的名字和引用的 ConfigMap/Secret 就可以了。要注意的是 Volume 属于 Pod，不属于容器，所以它和字段“containers”是同级的，都属于“spec”。

```yml
# kubectl apply -f vol-pod.yml
# kubectl get pod
# kubectl exec -it vol-pod -- sh

# in Pod:
# cat /tmp/cm-items/greeting
# cat /tmp/sec-items/db

apiVersion: v1
kind: Pod
metadata:
  name: vol-pod

spec:
  volumes:
  - name: cm-vol
    configMap:
      name: info
  - name: sec-vol
    secret:
      secretName: user

  containers:
  - volumeMounts:
    - mountPath: /tmp/cm-items
      name: cm-vol
    - mountPath: /tmp/sec-items
      name: sec-vol

    image: busybox
    name: busy
    imagePullPolicy: IfNotPresent
    command: ["/bin/sleep", "300"]

```

首先需要 Volume 的定义，有了 Volume 的定义之后，就可以在容器里挂载了，这要用到“**volumeMounts**”字段，正如它的字面含义，可以把定义好的 Volume 挂载到容器里的某个路径下，所以需要在里面用“**mountPath**”“**name**”明确地指定挂载路径和 Volume 的名字。

因为这种形式上的差异，以 Volume 的方式来使用 ConfigMap/Secret，就和环境变量不太一样。环境变量用法简单，更适合存放简短的字符串，而 Volume 更适合存放大数据量的配置文件，在 Pod 里加载成文件后让应用直接读取使用。

### 36. 快速暴露 Kubernetes 服务

因为 Pod 都是运行在 Kubernetes 内部的私有网段里的，外界无法直接访问，想要对外暴露服务，需要使用一个专门的 `kubectl port-forward` 命令，它专门负责把本机的端口映射到在目标对象的端口号，有点类似 Docker 的参数 `-p`，经常用于 Kubernetes 的临时调试和测试。

下面我就把本地的“8080”映射到 WordPress Pod 的“80”，kubectl 会把这个端口的所有数据都转发给集群内部的 Pod：

```sh
kubectl port-forward wp-pod 8080:80 &
```

注意在命令的末尾使用了一个 `&` 符号，让端口转发工作在后台进行，这样就不会阻碍我们后续的操作。

如果想关闭端口转发，需要敲命令 `fg` ，它会把后台的任务带回到前台，然后就可以简单地用“Ctrl + C”来停止转发了。

