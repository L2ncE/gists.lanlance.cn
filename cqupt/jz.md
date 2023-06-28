## 计算机组织与结构

### 冯诺依曼计算机

- 采用二进制形式表示数据和指令；指令由操作码和地址码组成
- 将程序和数据存放在存储器中，使计算机在工作时从存储器取出指令加以执行，自动完成计算任务：“存储程序”和“程序控制”
- 指令的执行是顺序的，即一般按照指令在存储器中存放的顺序执行，程序分支由转移指令实现
- 计算机由存储器、运算器、控制器、输入和输出设备五大基本部件组成，规定了 5 部分的基本功能

### 数的表示

- 真值：现实中真实的数值
- 机器数：计算机中用 0 和 1 数码组合表达的数值
- 定点数：固定小数点的位置表达数值的机器数
  - 定点整数：将小数点固定在机器数的最右侧表达的整数
  - 定点小数：将小数点固定在机器数的最左侧表达的小数
- 浮点数：小数点浮动表达的实数
- 无符号数：只表达 0 和正整数的定点整数
- 有符号数：表达负整数、0 和正整数的定点整数
  - 符号位需要占用一个位，常用机器数的最高位
  - 0 表示正数、1 表示负数
  - 具有原码、反码、补码、移码

#### 数的机器码表示

![](https://picture.lanlance.cn/i/2023/06/27/649a54d7b6453.png)

> 掌握原码、补码、反码、移码的定义、特性、相互转换

#### 浮点数的表示方法

![](https://picture.lanlance.cn/i/2023/06/27/649a554725f26.png)

#### 规格化表示原则

![](https://picture.lanlance.cn/i/2023/06/27/649a5575c78bf.png)

### 存储器

#### 分类

- 按存储介质分
  - 半导体存储器：用半导体器件组成的存储器
  - 磁表面存储器：用磁性材料做成的存储器
- 按存储方式分
  - 随机存储器：任何存储单元的内容都能被随机存取，且存取时间和存储单元的物理位置无关
  - 顺序存储器：只能按某种顺序来存取，存取时间和存储单元的物理位置有关
- 按存储器的读写功能分：ROM，RAM
- 按信息的可保存性分：非永久记忆，永久记忆
- 按在计算机系统中的作用分：主存、辅存、高速缓存、控制存储器

#### SRAM/DRAM

- SRAM：用作小容量、高效率的内存多用作 Cache
- DRAM：用作主存，需要定期对存储矩阵所有行逐一刷新

#### 存储器与 CPU 的连接方式

- CPU 对存储器进行读/写操作，首先由地址总线给出地址信号，然后要对存储器发出读操作或写操作的控制信号，最后在数据总线上进行信息交流
- 所以存储器与 CPU 之间需要
  - 地址线的连接
  - 数据线的连接
  - 控制线的连接
- 存储器芯片的容量是有限的,为了满足实际存储器的容量要求，需要对存储器进行扩展

#### 存储器拓展

- 位拓展法

  只加长每个存储单元的字长，而不增加存储单元的数量

- 字拓展法

  仅增加存储单元的数量，而各单元的位数不变

- 字位同时拓展法

  既增加存储单元的数量，也加长各单元的位数

### Cache 存储器

- 在相对容量较大而速度较慢的主存与高速处理器之间设置的少量但快速的存储器
- 主要目的：提高存储器速度
- 为追求高速，包括管理在内的全部功能由硬件实现

#### Cache 命中率

![](https://picture.lanlance.cn/i/2023/06/27/649a58518091b.png)

#### Cache 访问效率

![](https://picture.lanlance.cn/i/2023/06/27/649a585e0f02b.png)

#### Cache 结构

- Cache 的数据块称为行（线 Line，槽 Slot
- 主存的数据块称为块（Block）
- 行与块是等长的，包含 k=2w 个主存字
- Cache 由数据存储器和标签存储器组成
  - 数据存储器：高速缓存主存数据
  - 标签存储器：保存数据所在主存的地址信息

#### 主存与 Cache 的地址映射

Cache 通过地址映射(mapping)的方法确定主存块与 Cache 行之间的对应关系，确定一个主存块应该存放到哪个 Cache 行中

- 全相联映射(fully associative mapping)

  可以将一个主存块存储到任意一个 Cache 行

  - 优点：命中率较高，Cache 的存储空间利用率高
  - 缺点：线路复杂，成本高，速度低

- 直接映射(direct mapping)

  将一个主存块存储到唯一的一个 Cache 行

  - 优点：硬件简单，容易实现
  - 缺点：命中率低， Cache 的存储空间利用率低

- 组相联映射(set associative mapping)

  可以将一个主存块存储到唯一的一个 Cache 组中任意一个行

  - 组间采用直接映射，组内为全相联
  - 硬件较简单，速度较快，命中率较高

#### 替换问题

- 新主存块要进入 Cache，决定替换哪个原主存块
- 直接映射，只能替换唯一的一个 Cache 行
- 全相联和组相联，需要选择替换策略（算法）

1. 最不常用(LFU: least-frequently used)
   替换使用次数最少的块
2. 最近最少使用法(LRU: least-recently used)
   本指替换近期最少使用的块，实际实现的是替换最久没有被使用的块
3. 随机法(random)
   随意选择被替换的块，不依赖以前的使用情况

### 指令系统

#### 指令格式

![](https://picture.lanlance.cn/i/2023/06/27/649a5bee4b805.png)

1. 单字长二地址指令
2. 操作码字段 OP 长度为 7 位，可指定 128 条指令
3. 源寄存器和目标寄存器都是通用寄存器（可分别指定 16 个）。两个操作数均在寄存器中，所以是寄存器－寄存器型指令
4. 这种指令结构常用于算术逻辑运算类指令

#### 操作码拓展技术

![](https://picture.lanlance.cn/i/2023/06/27/649a5c3b9b4cb.png)

#### 常用数据寻址方式

- 隐含寻址：在指令中不明显地给出操作数的地址
- 寄存器寻址：指令中给出的操作数地址不是内存的地址单元号，而是通用寄存器的编号。即操作数不放在内存中，而是放在通用寄存器中
- 立即寻址：指令的地址字段指出的不是操作数的地址，而直接是操作数本身
- 直接寻址：在指令格式的地址字段中，直接给出操作数在内存的地址
- 寄存器间接寻址：指令中指定的寄存器中的内容不是操作数，而是操作数的地址
- 基址(寄存器相对)寻址：基址寄存器的内容加上指令中给定的形式地址(偏移量)，形成操作数的有效地址

![](https://picture.lanlance.cn/i/2023/06/27/649a5c9a03015.png)

### 中央处理器

#### CPU 的基本组成

- 控制器完成对整个计算机系统操作的协调与指挥
  - 控制机器从内存中取出一条指令，并指出下一条指令在内存中的位置
  - 对指令进行译码，并产生相应的操作控制信号，送往相应的部件，启动规定的动作
  - 指挥并控制 CPU、内存与输入/输出（I/O）设备之间数据流动的方向
- 运算器是数据加工处理部件，所进行的全部操作由控制器发出的控制信号指挥
  - 执行所有的算术运算
  - 执行所有的逻辑运算，并进行逻辑测试

#### 指令周期

- 一个完整的指令周期由若干机器周期：
  - 取指周期——间址周期——执行周期——中断周期
- 所有指令的第一个机器周期必为取指周期
- 一个基本的 CPU 周期包含 4 个时钟周期，对于某些 CPU 周期可以包含更多的时钟周期
- 不同指令的指令周期所包含的时钟周期个数不一定相同

#### 时序信号

- 计算机的协调动作需要时间标志，而且需要采用多级时序体制。而时间标志则用时序信号来体现
- 硬布线控制器中，时序信号往往采用主状态周期-节拍电位-节拍脉冲三级体制
  - 主状态周期（指令周期）：包含若干个节拍周期，可以用一个触发器的状态持续时间来表示
  - 节拍电位（机器周期）：表示一个 CPU 周期的时间，包含若干个节拍脉冲
  - 节拍脉冲（时钟周期）：表示较小的时间单位
- 微程序控制器中，时序信号则一般采用节拍电位-节拍脉冲二级体制

![](https://picture.lanlance.cn/i/2023/06/27/649a5f3c470c4.png)

#### 微指令和微程序

- 微指令
  - 一个 CPU 周期中，实现一定操作功能的一组微命令的组合
  - 微指令一般包含操作控制字段和顺序控制字段
    - 操作控制：用于发出管理和指挥全机工作的控制信号
    - 顺序控制：用于决定产生下一条微指令的地址
  - 所有的微指令都存放于控制存储器中，使用微地址访问
- 微程序
  - 能实现一条机器指令功能的多条微指令序列
  - 每条机器指令都对应着一段微程序

#### 微程序控制器

微程序控制器主要构成部件

- 控制存储器（CM）
  - 存放实现全部指令系统的微指令
  - 由只读存储器构成，要求速度快，读出周期短
- 微指令寄存器

  存放由控制存储器读出的一条微指令信息

  - 微地址寄存器：决定将要访问的下一条微指令的地址
  - 微命令寄存器：保存一条微指令的操作控制字段和判别测试字段的信息

- 地址转移逻辑
  - 用于跳跃寻址微指令时，承担自动完成修改微地址的任务

#### 并行处理技术

并行性（Parallelism）：

在同一时刻或是同一时间间隔内完成两种或两种以上性质相同或不相同的工作

- 同时性（Simultaneity）：同一时刻发生的并行性
- 并发性（Concurrency）：同一个时间间隔内发生的并行性

并行性的等级

- 指令内部并行：微操作之间（相容微命令信号）
- 指令级并行（ILP：Instruction Level Parallel）
- 线程级并行（TLP：Thread Level Parallel ）
- 程序级并行
- 系统级并行：分布式系统、多机系统、机群系统

#### 提高并行性的技术途径

- 时间重叠（Time-interleaving）＝ 时间并行
  - 多个过程在时间上相互错开，轮流重叠地使用同一套硬件设备的各个部分
- 资源重复（Resource-replication）＝ 空间并行
  - 通过重复设置资源（尤其是硬件资源），提高性能
- 资源共享（Resource-sharing）
  - 使多个任务按一定时间顺序轮流使用同一套硬件设备

#### CPU 流水线

- 流水线实际上是把一个功能部件分解成多个独立的子功能部件（一个任务也就分成了几个子任务，每个子任务由一个子功能部件完成），并依靠多个子功能部件并行工作来缩短所有任务的执行时间
- 流水线有助于提高整个程序（所有任务）的吞吐率，但并没有减少每个指令（任务）的执行时间
- 流水线各个功能段所需时间应尽量相等。否则，时间长的功能段将成为流水线的“瓶颈”

#### 流水线的主要问题

- 结构相关（资源冲突）：当指令重叠执行过程中，硬件资源满足不了指令重叠执行的要求
- 数据相关（数据冲突） ：在同时执行的多条指令中，一条指令依赖前一条指令的执行结果（数据）却无法得到
- 控制相关（控制冲突）：流水线遇到分支指令或其他改变 PC 值的指令

#### CPU 性能评价

![](https://picture.lanlance.cn/i/2023/06/27/649a61a5d3f7f.png)

### 总线系统

总线是构成计算机系统的互连机构，是多个系统功能部件之间进行数据传送的公共通路

#### 总线的结构形态

- 内部总线：CPU 内部连接各寄存器及运算部件之间的总线
- 系统总线：CPU 同计算机系统的其他高速功能部件，如存储器、通道等互相连接的总线
- I/O 总线：中低速 I/O 设备间互相连接的总线

![](https://picture.lanlance.cn/i/2023/06/27/649a61feeef01.png)

#### 总线仲裁

- 主设备(Master)：控制总线完成数据传输
- 从设备(Slave)：被动实现数据交换
- 总线仲裁：决定当前控制总线的主设备
  - 集中仲裁：中央仲裁器负责
  - 分布仲裁：比较各个主设备仲裁号决定

![](https://picture.lanlance.cn/i/2023/06/27/649a622cd3ce4.png)

### 外围设备

#### 显示设备

- 像素：组成图像的最小单位，显示器上的发光点
- 分辨率：显示器所能表示的像素个数
- 分辨率＝水平点数 × 垂直点数；如 1280×1024
- 彩色显示器：每个像素由红、绿、蓝三色组成
- 如果红、绿、蓝三色都用 8 个二进制位表达（RGB），则彩色图像就具有 224（16M）种颜色

#### 磁盘

- 记录面
  - 磁盘片表面
  - 一个盘片有上下两个记录面
- 磁道
  - 记录面上一系列同心圆
  - 最外圈为 0 磁道 ，依次为 1、2、……、N 磁道
  - 每个磁道的存储容量均相同
  - 不同盘片的相同磁道构成一个柱面
- 扇区
  - 同心圆上的一段磁道区域
  - 每个扇区的存储容量也相同

![](https://picture.lanlance.cn/i/2023/06/27/649a62bbbbfe0.png)

#### 存储密度

- 道密度：沿磁盘半径方向单位长度上的磁道数，单位为道/英寸
- 位密度：是磁道单位长度上能记录的二进制代码位数，单位为位/英寸
- 面密度：位密度和道密度的乘积，单位为位/平方英寸

相关概念

- 道距：相邻两磁道中心线之间的距离；
- 道宽：磁化轨迹的宽度。

#### 存储容量

- 存储容量=记录面数 × 每面磁道数 × 磁道容量
- 非格式化容量
  - 磁记录表面可以利用的磁化单元总数
- 格式化容量
  - 按照某种特定的记录格式所能存储信息的总量，也就是用户可以真正使用的容量。
  - 格式化容量一般是非格式化容量的 60%—70%。

#### 平均存取时间

- 存取时间=找道时间+等待时间
  - 定位时间（找道时间）
    - 将磁头定位至所要求的磁道上所需的时间
  - 等待时间
    - 找道完成后，盘片将所要访问信息转到磁头下方的时间
- 平均找道时间
  - 最大与最小找道时间的平均值，约为 10~20ms
- 平均等待时间
  - 与磁盘转速有关，是磁盘旋转一周时间的一半
  - 硬盘转速为 7200 转/分，故平均等待时间约为 4ms。

### 输入输出系统

#### I/O 接口电路

- 计算机的外围(外部)设备多种多样
- 工作原理、驱动方式、信息格式、以及工作速度方面彼此差别很大
- 外设不能与 CPU 直接相连，必须经过中间电路（I/O 接口电路）再与系统相连
- I/O 接口电路是位于系统与外设间、用来协助完成数据传送和控制任务的逻辑电路

![](https://picture.lanlance.cn/i/2023/06/27/649a6392bb254.png)

#### I/O 端口的编址

- I/O 端口（Port）泛指 I/O 地址，对应 I/O 接口寄存器
- 一个接口电路可以具有多个 I/O 端口，每个端口用来保存和交换不同的信息
- 数据寄存器、状态寄存器和控制寄存器占有的 I/O 地址常依次被称为数据端口、状态端口和控制端口，用于保存数据、状态和控制信息
- 输入、输出端口可以是同一个 I/O 地址
- 接口电路占用的 I/O 端口有两类编排形式
  - I/O 端口单独编址
  - I/O 端口与存储器统一编址

#### CPU 对外围设备的管理方式

![](https://picture.lanlance.cn/i/2023/06/27/649a63d605a7f.png)

#### CPU 和外设之间信息交换的方式

- 程序控制下的数据传送
  - 通过 CPU 执行程序中的 I/O 指令来完成传送
  - 程序查询方式
  - 程序中断方式
- 直接存储器存取 DMA 方式
  - 外设经 DMA 控制器向 CPU 申请总线，由 DMA 控制器利用系统总线完成外设和存储器间的数据传送
- 通道方式
  - 通道(I/O 处理器)管理外设，完成传送和数据处理
- 外围处理机方式
  - 通道方式的进一步发展，基本独立于主机工作

#### 程序中断方式

- 处理器在执行程序过程中，被内部或外部的事件所打断，转去执行一段预先安排好的中断服务程序；服务结束后，又返回原来的断点，继续执行原来的程序
- 中断源：引起中断的事件或原因
- 例如：
  - 外设的数据传送请求
  - 系统定时请求
  - 电源掉电等故障
  - 运算出错等错误
  - 程序异常或调试请求

![](https://picture.lanlance.cn/i/2023/06/27/649a643f7469b.png)

#### DMA 方式

- 克服程序控制传送的不足：
  - 外设 →CPU→ 存储器
  - 外设 ←CPU← 存储器
- 直接存储器存取 DMA：
  - 外设 → 存储器
  - 外设 ← 存储器
- 特点比较：
  - 查询传送： 简单实用，效率较低
  - 中断传送：外设主动，可与 CPU 并行工作，但每次传送需要大量额外时间开销
  - DMA 传送：CPU 释放总线，由 DMA 控制器管理，外设直接和存储器进行数据传送，适合大量、快速数据传送