## K8s 入门实战

### 如何理解 Kubernetes 中的 Pod

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

### Job/CronJob

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

### ConfigMap/Secret

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

### Deployment

Pod 只能管理容器，不能管理自身，所以就出现了 Deployment，由它来管理 Pod。用来管理 Pod，实现在线业务应用的新 API 对象，就是 Deployment。
#### 如何使用 YAML 描述 Deployment

```yml
# kubectl apply -f deploy.yml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: ngx-dep
  labels:
    app: ngx-dep

spec:
  replicas: 2
  selector:
    matchLabels:
      app: ngx-dep

  template:
    metadata:
      labels:
        app: ngx-dep
    spec:
      containers:
      - image: nginx:alpine
        name: nginx
        ports:
        - containerPort: 80
```

Kubernetes 采用的是这种“贴标签”的方式，通过在 API 对象的“metadata”元信息里加各种标签（labels），我们就可以使用类似关系数据库里查询语句的方式，筛选出具有特定标识的那些对象。**通过标签这种设计，Kubernetes 就解除了 Deployment 和模板里 Pod 的强绑定，把组合关系变成了“弱引用”**。

### DaemonSet

Deployment 并不关心这些 Pod 会在集群的哪些节点上运行，**在它看来，Pod 的运行环境与功能是无关的，只要 Pod 的数量足够，应用程序应该会正常工作**。

这个假设对于大多数业务来说是没问题的，比如 Nginx、WordPress、MySQL，它们不需要知道集群、节点的细节信息，只要配置好环境变量和存储卷，在哪里“跑”都是一样的。

但是有一些业务比较特殊，它们不是完全独立于系统运行的，而是与主机存在“绑定”关系，必须要依附于节点才能产生价值，比如说：

- 网络应用（如 kube-proxy），必须每个节点都运行一个 Pod，否则节点就无法加入 Kubernetes 网络。
- 监控应用（如 Prometheus），必须每个节点都有一个 Pod 用来监控节点的状态，实时上报信息。
- 日志应用（如 Fluentd），必须在每个节点上运行一个 Pod，才能够搜集容器运行时产生的日志数据。
- 安全应用，同样的，每个节点都要有一个 Pod 来执行安全审计、入侵检查、漏洞扫描等工作。

这些业务如果用 Deployment 来部署就不太合适了，因为 Deployment 所管理的 Pod 数量是固定的，而且可能会在集群里“漂移”，但，实际的需求却是要在集群里的每个节点上都运行 Pod，也就是说 Pod 的数量与节点数量保持同步。

所以，Kubernetes 就定义了新的 API 对象 DaemonSet，它在形式上和 Deployment 类似，都是管理控制 Pod，但管理调度策略却不同。DaemonSet 的目标是在集群的每个节点上运行且仅运行一个 Pod，就好像是为节点配上一只“看门狗”，忠实地“守护”着节点，这就是 DaemonSet 名字的由来。

#### 如何使用 YAML 描述 DaemonSet

```yml
# kubectl apply -f ds.yml

apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: redis-ds
  labels:
    app: redis-ds

spec:
  selector:
    matchLabels:
      name: redis-ds

  template:
    metadata:
      labels:
        name: redis-ds

    spec:
      containers:
      - name: redis5
        image: redis:5-alpine
        ports:
        - containerPort: 6379

      tolerations:
      # this toleration is to have the daemonset runnable on master nodes
      # remove it if your masters can't run pods
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
        operator: Exists
```

### Service

Service 使用了 iptables 技术，每个节点上的 kube-proxy 组件自动维护 iptables 规则，客户不再关心 Pod 的具体地址，只要访问 Service 的固定 IP 地址，Service 就会根据 iptables 规则转发请求给它管理的多个 Pod，是典型的负载均衡架构。

不过 Service 并不是只能使用 iptables 来实现负载均衡，它还有另外两种实现技术：性能更差的 userspace 和性能更好的 ipvs。
#### 如何使用 YAML 描述 Service

```yml
# kubectl apply -f svc.yml
# kubectl describe svc ngx-svc

apiVersion: v1
kind: Service
metadata:
  name: ngx-svc

spec:
  selector:
    app: ngx-dep

  ports:
  - port: 80
    protocol: TCP
    targetPort: 80

  #type: ClusterIP
  type: NodePort
```

`selector` 和 Deployment/DaemonSet 里的作用是一样的，用来过滤出要代理的那些 Pod。因为我们指定要代理 Deployment，所以 Kubernetes 就为我们自动填上了 ngx-dep 的标签，会选择这个 Deployment 对象部署的所有 Pod。

Service 对象有一个关键字段“**type**”，表示 Service 是哪种类型的负载均衡。前面我们看到的用法都是对集群内部 Pod 的负载均衡，所以这个字段的值就是默认的“**ClusterIP**”，Service 的静态 IP 地址只能在集群内访问。

除了“ClusterIP”，Service 还支持其他三种类型，分别是“**ExternalName**”“**LoadBalancer**”“**NodePort**”。不过前两种类型一般由云服务商提供。

如果我们在使用命令 `kubectl expose` 的时候加上参数 `--type=NodePort`，或者在 YAML 里添加字段 `type:NodePort`，那么 Service 除了会对后端的 Pod 做负载均衡之外，还会在集群里的每个节点上创建一个独立的端口，用这个端口对外提供服务，这也正是“NodePort”这个名字的由来。

#### 如何以域名的方式使用 Service

Service 对象的 IP 地址是静态的，保持稳定，这在微服务里确实很重要，不过数字形式的 IP 地址用起来还是不太方便。这个时候 Kubernetes 的 DNS 插件就派上了用处，它可以为 Service 创建易写易记的域名，让 Service 更容易使用。

namespace 的简写是“**ns**”，你可以使用命令 `kubectl get ns` 来查看当前集群里都有哪些名字空间，也就是说 API 对象有哪些分组。

Service 对象的域名完全形式是“**对象.名字空间.svc.cluster.local**”，但很多时候也可以省略后面的部分，直接写“**对象.名字空间**”甚至“**对象名**”就足够了，默认会使用对象所在的名字空间。

Kubernetes 也为每个 Pod 分配了域名，形式是“**IP 地址.名字空间.pod.cluster.local**”，但需要把 IP 地址里的 `.` 改成 `-` 。比如地址 `10.10.1.87`，它对应的域名就是 `10-10-1-87.default.pod`。

#### 缺点

1. 端口数量很有限。Kubernetes 为了避免端口冲突，默认只在“30000~32767”这个范围内随机分配，只有 2000 多个，而且都不是标准端口号，这对于具有大量业务应用的系统来说根本不够用。
2. 它会在每个节点上都开端口，然后使用 kube-proxy 路由到真正的后端 Service，这对于有很多计算节点的大集群来说就带来了一些网络通信成本，不是特别经济。
3. 它要求向外界暴露节点的 IP 地址，这在很多时候是不可行的，为了安全还需要在集群外再搭一个反向代理，增加了方案的复杂度。

### Ingress

Ingress 可以说是在七层上另一种形式的 Service，它同样会代理一些后端的 Pod，也有一些路由规则来定义流量应该如何分配、转发，只不过这些规则都使用的是 HTTP/HTTPS 协议。
#### Ingress Controller

Service 本身是没有服务能力的，它只是一些 iptables 规则，**真正配置、应用这些规则的实际上是节点里的 kube-proxy 组件**。如果没有 kube-proxy，Service 定义得再完善也没有用。

同样的，Ingress 也只是一些 HTTP 路由规则的集合，相当于一份静态的描述文件，真正要把这些规则在集群里实施运行，还需要有另外一个东西，这就是 `Ingress Controller`，它的作用就相当于 Service 的 kube-proxy，能够读取、应用 Ingress 规则，处理、调度流量。
#### Ingress Class

- 由于某些原因，项目组需要引入不同的 Ingress Controller，但 Kubernetes 不允许这样做；
- Ingress 规则太多，都交给一个 Ingress Controller 处理会让它不堪重负；
- 多个 Ingress 对象没有很好的逻辑分组方式，管理和维护成本很高；
- 集群里有不同的租户，他们对 Ingress 的需求差异很大甚至有冲突，无法部署在同一个 Ingress Controller 上。

所以，Kubernetes 就又提出了一个 `Ingress Class` 的概念，让它插在 Ingress 和 Ingress Controller 中间，作为流量规则和控制器的协调人，解除了 Ingress 和 Ingress Controller 的强绑定关系。
#### 如何使用 YAML 描述 Ingress/Ingress Class

```yml
# kubectl create ing ngx-ing --rule="ngx.test/=ngx-svc:80" $out
# kubectl create ing ngx-ing --rule="ngx.test/=ngx-svc:80" --class=ngx-ink $out

# curl 127.1/nginx-health
# curl 127.1:8081/nginx-ready

---

apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: ngx-ink

spec:
  controller: nginx.org/ingress-controller

---

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ngx-ing

  # customize the behaviors of nginx
  annotations:
    nginx.org/lb-method: round_robin

spec:
  ingressClassName: ngx-ink

  rules:
  - host: ngx.test
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: ngx-svc
            port:
              number: 80
```

### PersistentVolume

**作为存储的抽象，PV 实际上就是一些存储设备、文件系统**，比如 Ceph、GlusterFS、NFS，甚至是本地磁盘，管理它们已经超出了 Kubernetes 的能力范围，所以，一般会由系统管理员单独维护，然后再在 Kubernetes 里创建对应的 PV。要注意的是，PV 属于集群的系统资源，是和 Node 平级的一种对象，Pod 对它没有管理权，只有使用权。

这么多种存储设备，只用一个 PV 对象来管理还是有点太勉强了，不符合“单一职责”的原则，让 Pod 直接去选择 PV 也很不灵活。于是 Kubernetes 就又增加了两个新对象，**PersistentVolumeClaim**和**StorageClass**，用的还是“中间层”的思想，把存储卷的分配管理过程再次细化。

PersistentVolumeClaim，简称 PVC，从名字上看比较好理解，就是用来向 Kubernetes 申请存储资源的。PVC 是给 Pod 使用的对象，它相当于是 Pod 的代理，代表 Pod 向系统申请 PV。一旦资源申请成功，Kubernetes 就会把 PV 和 PVC 关联在一起，这个动作叫做“**绑定**”（bind）。

但是，系统里的存储资源非常多，如果要 PVC 去直接遍历查找合适的 PV 也很麻烦，所以就要用到 StorageClass。StorageClass 的作用有点像里的 IngressClass，它抽象了特定类型的存储系统（比如 Ceph、NFS），在 PVC 和 PV 之间充当“协调人”的角色，帮助 PVC 找到合适的 PV。也就是说它可以简化 Pod 挂载“虚拟盘”的过程，让 Pod 看不到 PV 的实现细节。

#### 使用 YAML 描述 PersistentVolume/PersistentVolumeClaim

```yml
# kubectl get pv
# kubectl get pvc

# kubectl exec -it host-pvc-pod -- sh
# echo aaa > /tmp/a.txt
#
# check node's /tmp/host-10m-pv

---

apiVersion: v1
kind: PersistentVolume
metadata:
  name: host-10m-pv

spec:
  storageClassName: host-test

  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 10Mi

  # mkdir -p /tmp/host-10m-pv/
  hostPath:
    path: /tmp/host-10m-pv/

---

# pvc
# try to find the most suitable pv
# capacity/accessModes
apiVersion: v1
kind: PersistentVolumeClaim

metadata:
  name: host-5m-pvc

spec:

  storageClassName: host-test

  accessModes:
    - ReadWriteOnce

  resources:
    requests:
      storage: 5Mi

---

# pod
apiVersion: v1
kind: Pod
metadata:
  name: host-pvc-pod

spec:

  volumes:
  - name: host-pvc-vol
    persistentVolumeClaim:
      claimName: host-5m-pvc

  containers:
    - name: ngx-pvc-pod
      image: nginx:alpine
      ports:
      - containerPort: 80
      volumeMounts:
      - name: host-pvc-vol
        mountPath: /tmp

---

```

PVC 的内容与 PV 很像，但它不表示实际的存储，而是一个“申请”或者“声明”，spec 里的字段描述的是对存储的“期望状态”。

所以 PVC 里的 `storageClassName`、`accessModes` 和 PV 是一样的，**但不会有字段 `capacity`，而是要用 `resources.request` 表示希望要有多大的容量**。

### StatefulSet

对于“有状态应用”，多个实例之间可能存在依赖关系，比如 master/slave、active/passive，需要依次启动才能保证应用正常运行，外界的客户端也可能要使用固定的网络标识来访问实例，而且这些信息还必须要保证在 Pod 重启后不变。

所以，Kubernetes 就在 Deployment 的基础之上定义了一个新的 API 对象，名字也很好理解，就叫 StatefulSet，专门用来管理有状态的应用。

#### 如何使用 YAML 描述 StatefulSet

```yml
---

apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis-sts

spec:
  # headless svc
  serviceName: redis-svc

  replicas: 2
  selector:
    matchLabels:
      app: redis-sts

  template:
    metadata:
      labels:
        app: redis-sts
    spec:
      containers:
      - image: redis:5-alpine
        name: redis
        ports:
        - containerPort: 6379

---

apiVersion: v1
kind: Service
metadata:
  name: redis-svc

spec:
  selector:
    app: redis-sts

  # headless
  clusterIP: None

  ports:
  - port: 6379
    protocol: TCP
    targetPort: 6379

---

```

Service 发现这些 Pod 不是一般的应用，而是有状态应用，需要有稳定的网络标识，所以就会为 Pod 再多创建出一个新的域名，格式是“**Pod 名.服务名.名字空间.svc.cluster.local**”。当然，这个域名也可以简写成“**Pod 名.服务名**”。

显然，在 StatefulSet 里的这两个 Pod 都有了各自的域名，也就是稳定的网络标识。那么接下来，外部的客户端只要知道了 StatefulSet 对象，就可以用固定的编号去访问某个具体的实例了，虽然 Pod 的 IP 地址可能会变，但这个有编号的域名由 Service 对象维护，是稳定不变的。

Service 原本的目的是负载均衡，应该由它在 Pod 前面来转发流量，但是对 StatefulSet 来说，这项功能反而是不必要的，因为 Pod 已经有了稳定的域名，外界访问服务就不应该再通过 Service 这一层了。所以，从安全和节约系统资源的角度考虑，**我们可以在 Service 里添加一个字段 `clusterIP: None` ，告诉 Kubernetes 不必再为这个对象分配 IP 地址**。

### 48. 如何实现 StatefulSet 的数据持久化

为了强调持久化存储与 StatefulSet 的一对一绑定关系，Kubernetes 为 StatefulSet 专门定义了一个字段“**volumeClaimTemplates**”，直接把 PVC 定义嵌入 StatefulSet 的 YAML 文件里。这样能保证创建 StatefulSet 的同时，就会为每个 Pod 自动创建 PVC，让 StatefulSet 的可用性更高。

```yml
---

apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis-pv-sts

spec:
  # headless svc
  serviceName: redis-pv-svc

  # pvc
  volumeClaimTemplates:
  - metadata:
      name: redis-100m-pvc
    spec:
      storageClassName: nfs-client
      accessModes:
        - ReadWriteMany
      resources:
        requests:
          storage: 100Mi

  replicas: 2
  selector:
    matchLabels:
      app: redis-pv-sts

  template:
    metadata:
      labels:
        app: redis-pv-sts
    spec:
      containers:
      - image: redis:5-alpine
        name: redis
        ports:
        - containerPort: 6379

        volumeMounts:
        - name: redis-100m-pvc
          mountPath: /data

---

apiVersion: v1
kind: Service
metadata:
  name: redis-pv-svc

spec:
  selector:
    app: redis-pv-sts

  # headless
  clusterIP: None

  ports:
  - port: 6379
    protocol: TCP
    targetPort: 6379

---
```

### Kubernetes 滚动更新

#### Kubernetes 如何定义应用版本

**在 Kubernetes 里应用的版本变化就是 `template` 里 Pod 的变化**，哪怕 `template` 里只变动了一个字段，那也会形成一个新的版本，也算是版本变化。

但 `template` 里的内容太多了，拿这么长的字符串来当做“版本号”不太现实，所以 Kubernetes 就使用了“摘要”功能，用摘要算法计算 `template` 的 Hash 值作为“版本号”，虽然不太方便识别，但是很实用。
#### Kubernetes 如何实现应用更新

执行命令 `kubectl apply` 来更新应用，因为改动了镜像名，Pod 模板变了，就会触发“版本更新”，然后用一个新命令：`kubectl rollout status`，来查看应用更新的状态。

“滚动更新”就是由 Deployment 控制的两个同步进行的“应用伸缩”操作，老版本缩容到 0，同时新版本扩容到指定值，是一个“此消彼长”的过程。
#### Kubernetes 如何管理应用更新

Kubernetes 的“滚动更新”功能确实非常方便，不需要任何人工干预就能简单地把应用升级到新版本，也不会中断服务，不过如果更新过程中发生了错误或者更新后发现有 Bug 该怎么办呢？

要解决这两个问题，我们还是要用 `kubectl rollout` 命令。

在应用更新的过程中，你可以随时使用 `kubectl rollout pause` 来暂停更新，检查、修改 Pod，或者测试验证，如果确认没问题，再用 `kubectl rollout resume` 来继续更新。

对于更新后出现的问题，Kubernetes 为我们提供了“后悔药”，也就是更新历史，你可以查看之前的每次更新记录，并且回退到任何位置，和我们开发常用的 Git 等版本控制软件非常类似。

查看更新历史使用的命令是 `kubectl rollout history`。**想要回退到上一个版本，就可以使用命令 `kubectl rollout undo`，也可以加上参数 `--to-revision` 回退到任意一个历史版本**。
#### Kubernetes 如何添加更新描述

**只需要在 Deployment 的 `metadata` 里加上一个新的字段 `annotations`**。`annotations` 字段的含义是“注解”“注释”，形式上和 `labels` 一样，都是 Key-Value，也都是给 API 对象附加一些额外的信息，但是用途上区别很大。

- `annotations` 添加的信息一般是给 Kubernetes 内部的各种对象使用的，有点像是“扩展属性”；
- `labels` 主要面对的是 Kubernetes 外部的用户，用来筛选、过滤对象的。

### 容器资源配额

**只要在 Pod 容器的描述部分添加一个新字段 `resources` 就可以了**，它就相当于申请资源的 `Claim`。

```yml
apiVersion: v1
kind: Pod
metadata:
  name: ngx-pod-resources

spec:
  containers:
  - image: nginx:alpine
    name: ngx
    ports:
    - containerPort: 80

    resources:
      requests:
        cpu: 10m
        memory: 100Mi
      limits:
        cpu: 20m
        memory: 200Mi
```

- “**requests**”，意思是容器要申请的资源，也就是说要求 Kubernetes 在创建 Pod 的时候必须分配这里列出的资源，否则容器就无法运行。
- “**limits**”，意思是容器使用资源的上限，不能超过设定值，否则就有可能被强制停止运行。

### 容器状态探针

Kubernetes 为检查应用状态定义了三种探针，它们分别对应容器不同的状态：

- **Startup**，启动探针，用来检查应用是否已经启动成功，适合那些有大量初始化工作要做，启动很慢的应用。
- **Liveness**，存活探针，用来检查应用是否正常运行，是否存在死锁、死循环。
- **Readiness**，就绪探针，用来检查应用是否可以接收流量，是否能够对外提供服务。

如果一个 Pod 里的容器配置了探针，**Kubernetes 在启动容器后就会不断地调用探针来检查容器的状态**：

- 如果 Startup 探针失败，Kubernetes 会认为容器没有正常启动，就会尝试反复重启，当然其后面的 Liveness 探针和 Readiness 探针也不会启动。
- 如果 Liveness 探针失败，Kubernetes 就会认为容器发生了异常，也会重启容器。
- 如果 Readiness 探针失败，Kubernetes 会认为容器虽然在运行，但内部有错误，不能正常提供服务，就会把容器从 Service 对象的负载均衡集合中排除，不会给它分配流量。

```yml
# kubectl explain pod.spec.containers.startupProbe
# kubectl explain pod.spec.containers.livenessProbe
# kubectl explain pod.spec.containers.readinessProbe
#
# kubectl logs ngx-pod-probe -f

---

# this cm will be mounted to /etc/nginx/conf.d
apiVersion: v1
kind: ConfigMap
metadata:
  name: ngx-conf

data:
  default.conf: |
    server {
      listen 80;
      location = /ready {
        return 200 'I am ready';
        #return 500 'I am not ready';
      }
      location / {
        default_type text/plain;
        return 200 "Nginx OK";
      }
    }

---

apiVersion: v1
kind: Pod
metadata:
  name: ngx-pod-probe

spec:
  volumes:
  - name: ngx-conf-vol
    configMap:
      name: ngx-conf

  containers:
  - image: nginx:alpine
    name: ngx
    ports:
    - containerPort: 80

    volumeMounts:
    - mountPath: /etc/nginx/conf.d
      name: ngx-conf-vol

    # probes are here

    startupProbe:
      periodSeconds: 1
      timeoutSeconds: 1
      exec:
        command: ["cat", "/var/run/nginx.pid"]
        #command: ["cat", "nginx.pid"]  # wrong pid file

    livenessProbe:
      periodSeconds: 10
      timeoutSeconds: 1
      #failureThreshold: 1
      tcpSocket:
        #port: 80
        port: 8080

    readinessProbe:
      periodSeconds: 5
      timeoutSeconds: 1
      httpGet:
        path: /ready
        port: 80

---

```

### Kubernetes 集群管理

当多团队、多项目共用 Kubernetes 的时候，为了避免这些问题的出现，我们就需要**把集群给适当地“局部化”，为每一类用户创建出只属于它自己的“工作空间”**。

**想要把一个对象放入特定的名字空间，需要在它的 `metadata` 里添加一个 `namespace` 字段**

```yml
# kubectl create ns test-ns
# kubectl run ngx --image=nginx:alpine

---

apiVersion: v1
kind: Namespace
metadata:
  name: test-ns

---

apiVersion: v1
kind: Pod
metadata:
  name: ngx
  namespace: test-ns

spec:
  containers:
  - image: nginx:alpine
    name: ngx

---

```

`kubectl apply` 创建这个对象之后，我们直接用 `kubectl get` 是看不到它的，因为默认查看的是“default”名字空间，**想要操作其他名字空间的对象必须要用 `-n` 参数明确指定**：

```sh
kubectl get pod -n test-ns
```

因为名字空间里的对象都从属于名字空间，所以在删除名字空间的时候一定要小心，一旦名字空间被删除，它里面的所有对象也都会消失。

#### 资源配额

有了名字空间，我们就可以像管理容器一样，给名字空间设定配额，把整个集群的计算资源分割成不同的大小，按需分配给团队或项目使用。

**名字空间的资源配额需要使用一个专门的 API 对象，叫做 `ResourceQuota`**，因为资源配额对象必须依附在某个名字空间上，所以在它的 `metadata` 字段里必须明确写出 `namespace`（否则就会应用到 default 名字空间）。

```yml
# kubectl create ns dev-ns $out
# kubectl create quota dev-qt $out
#
# kubectl explain quota.spec
# kubectl describe -n dev-ns quota dev-qt
#
# kubectl explain limits.spec.limits
#
# kubectl run ngx --image=nginx:alpine -n dev-ns
# kubectl describe pod ngx -n dev-ns

---

apiVersion: v1
kind: Namespace
metadata:
  name: dev-ns

---

apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-qt
  namespace: dev-ns

spec:
  hard:
    requests.cpu: 10
    requests.memory: 10Gi
    limits.cpu: 10
    limits.memory: 20Gi

    requests.storage: 100Gi
    persistentvolumeclaims: 100

    pods: 100
    configmaps: 100
    secrets: 100
    services: 10
    services.nodeports: 5

    count/jobs.batch: 1
    count/cronjobs.batch: 1
    count/deployments.apps: 1

---

apiVersion: v1
kind: LimitRange
metadata:
  name: dev-limits
  namespace: dev-ns

spec:
  limits:
  - type: Container
    defaultRequest:
      cpu: 200m
      memory: 50Mi
    default:
      cpu: 500m
      memory: 100Mi
  - type: Pod
    max:
      cpu: 800m
      memory: 200Mi

---

```

它需要在 `spec` 里使用 `hard` 字段，意思就是“**硬性全局限制**”。

- CPU 和内存配额，使用 `request.*`、`limits.*`，这是和容器资源限制是一样的。
- 存储容量配额，使 `requests.storage` 限制的是 PVC 的存储总量，也可以用 `persistentvolumeclaims` 限制 PVC 的个数。
- 核心对象配额，使用对象的名字（英语复数形式），比如 `pods`、`configmaps`、`secrets`、`services`。
- 其他 API 对象配额，使用 `count/name.group` 的形式，比如 `count/jobs.batch`、`count/deployments.apps`。

在名字空间加上了资源配额限制之后，它会有一个合理但比较“烦人”的约束：要求所有在里面运行的 Pod 都必须用字段 `resources` 声明资源需求，否则就无法创建。这个时候就要用到一个**很小但很有用的辅助对象了—— `LimitRange`，简称是 `limits`，它能为 API 对象添加默认的资源配额限制**。

- `spec.limits` 是它的核心属性，描述了默认的资源限制。
- `type` 是要限制的对象类型，可以是 `Container`、`Pod`、`PersistentVolumeClaim`。
- `default` 是默认的资源上限，对应容器里的 `resources.limits`，只适用于 `Container`。
- `defaultRequest` 默认申请的资源，对应容器里的 `resources.requests`，同样也只适用于 `Container`。
- `max`、`min` 是对象能使用的资源的最大最小值。

### Metrics Server

Metrics Server 是一个专门用来收集 Kubernetes 核心资源指标（metrics）的工具，它定时从所有节点的 kubelet 里采集信息，但是对集群的整体性能影响极小，每个节点只大约会占用 1m 的 CPU 和 2MB 的内存，所以性价比非常高。
#### HorizontalPodAutoscaler

它是专门用来自动伸缩 Pod 数量的对象，适用于 Deployment 和 StatefulSet，但不能用于 DaemonSet。HorizontalPodAutoscaler 的能力完全基于 Metrics Server，它从 Metrics Server 获取当前应用的运行指标，主要是 CPU 使用率，再依据预定的策略增加或者减少 Pod 的数量。

```yml
# kubectl autoscale deploy ngx-hpa-dep --min=1 --max=10 --cpu-percent=5 $out
# kubectl apply -f hpa.yml
#
# wait some minutes for hpa monitor
#
# kubectl exec -it test -- sh
# curl ngx-hpa-svc
# ab -c 10 -t 60 -n 1000000 'http://ngx-hpa-svc/'
#
# kubectl run -it test --image=httpd:alpine -- sh

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: ngx-hpa-dep

spec:
  replicas: 1
  selector:
    matchLabels:
      app: ngx-hpa-dep

  template:
    metadata:
      labels:
        app: ngx-hpa-dep
    spec:
      containers:
      - image: nginx:alpine
        name: nginx
        ports:
        - containerPort: 80

        resources:
          requests:
            cpu: 50m
            memory: 10Mi
          limits:
            cpu: 100m
            memory: 20Mi

---

apiVersion: v1
kind: Service
metadata:
  name: ngx-hpa-svc
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: ngx-hpa-dep

---

apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler
metadata:
  name: ngx-hpa

spec:
  maxReplicas: 10
  minReplicas: 2
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ngx-hpa-dep
  targetCPUUtilizationPercentage: 5

---

apiVersion: v1
kind: Pod
metadata:
  name: test
spec:
  containers:
  - image: httpd:alpine
    name: test

---

```

**注意在它的** `spec` **里一定要用 `resources` 字段写清楚资源配额**，否则 HorizontalPodAutoscaler 会无法获取 Pod 的指标，也就无法实现自动化扩缩容。