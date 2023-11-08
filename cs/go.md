## Golang

### 1. 数组与切片的区别

1. 切片的底层数据是数组，是对数组的封装，数组是定长的，长度定义好之后，不能再更改。
2. 底层数组是可以被多个切片同时指向的，因此对一个切片的元素进行操作是有可能影响到其他切片的。

### 2. 切片的容量是怎样增长的

在 1.18 之前

> 当原 slice 容量小于 1024 的时候，新 slice 容量变成原来的 2 倍；原 slice 容量超过 1024，新 slice 容量变成原来的 1.25 倍。

在 1.18 之后

> 当原 slice 容量 (oldcap) 小于 256 的时候，新 slice(newcap) 容量为原来的 2 倍；原 slice 容量超过 256，新 slice 容量 newcap = oldcap+(oldcap+3\*256)/4

进行内存对齐之后，新 slice 的容量是要 `大于等于` 按照前半部分生成的 `newcap`。

### 3. append 函数

append 函数执行完后，返回的是一个全新的 slice，并且对传入的 slice 并不影响。

### 4. 向一个 nil 的 slice 添加元素会发生什么？

其实 `nil slice` 或者 `empty slice` 都是可以通过调用 append 函数来获得底层数组的扩容。最终都是调用 `mallocgc` 来向 Go 的内存管理器申请到一块内存，然后再赋给原来的 `nil slice` 或 `empty slice`，然后摇身一变，成为“真正”的 `slice` 了。

### 5. 切片作为函数参数

当 slice 作为函数参数时，就是一个普通的结构体。其实很好理解：若直接传 slice，在调用者看来，实参 slice 并不会被函数中的操作改变；若传的是 slice 的指针，在调用者看来，是会被改变原 slice 的

不管传的是 slice 还是 slice 指针，如果改变了 slice 底层数组的数据，会反应到实参 slice 的底层数据。

### 6. map 的实现原理

Go 语言采用的是哈希查找表，并且使用链表解决哈希冲突。

#### 结构

map 的结构体是 hmap，bmap 就是我们常说的“桶”，桶里面会最多装 8 个 key，这些 key 之所以会落入同一个桶，是因为它们经过哈希计算后，哈希结果是“一类”的。在桶内，又会根据 key 计算出来的 hash 值的高 8 位来决定 key 到底落入桶内的哪个位置（一个桶内最多有 8 个位置）。

当 map 的 key 和 value 都不是指针，并且 size 都小于 128 字节的情况下，会把 bmap 标记为不含指针，这样可以避免 gc 时扫描整个 hmap。

#### 哈希函数

在程序启动时，会检测 cpu 是否支持 aes，如果支持，则使用 aes hash，否则使用 memhash。

#### key 定位

用最后的 B 个 bit 位，例如 01010，值为 10，也就是 10 号桶。这个操作实际上就是取余操作，但是取余开销太大，所以代码实现上用的位操作代替。

再用哈希值的高 8 位，找到此 key 在 bucket 中的位置，这是在寻找已有的 key。最开始桶内还没有 key，新加入的 key 会找到第一个空位，放入。

buckets 编号就是桶编号，当两个不同的 key 落在同一个桶中，也就是发生了哈希冲突。冲突的解决手段是用链表法：在 bucket 中，从前往后找到第一个空位。这样，在查找某个 key 时，先找到对应的桶，再去遍历 bucket 中的 key。

如果在 bucket 中没找到，并且 overflow 不为空，还要继续去 overflow bucket 中寻找，直到找到或是所有的 key 槽位都找遍了，包括所有的 overflow bucket。

#### 扩容条件

在向 map 插入新 key 的时候，会进行条件检测，符合下面这 2 个条件，就会触发扩容：

1. 装载因子超过阈值，源码里定义的阈值是 6.5。
2. overflow 的 bucket 数量过多：当 B 小于 15，也就是 bucket 总数 2^B 小于 2^15 时，如果 overflow 的 bucket 数量超过 2^B；当 B >= 15，也就是 bucket 总数 2^B 大于等于 2^15，如果 overflow 的 bucket 数量超过 2^15。

对于条件 1，元素太多，而 bucket 数量太少，很简单：将 B 加 1，bucket 最大数量（2^B）直接变成原来 bucket 数量的 2 倍。于是，就有新老 bucket 了。注意，这时候元素都在老 bucket 里，还没迁移到新的 bucket 来。而且，新 bucket 只是最大数量变为原来最大数量（2^B）的 2 倍（2^B * 2）。

对于条件 2，其实元素没那么多，但是 overflow bucket 数特别多，说明很多 bucket 都没装满。解决办法就是开辟一个新 bucket 空间，将老 bucket 中的元素移动到新 bucket，使得同一个 bucket 中的 key 排列地更紧密。这样，原来，在 overflow bucket 中的 key 可以移动到 bucket 中来。结果是节省空间，提高 bucket 利用率，map 的查找和插入效率自然就会提升。

#### 扩容过程

由于 map 扩容需要将原有的 key/value 重新搬迁到新的内存地址，如果有大量的 key/value 需要搬迁，会非常影响性能。因此 Go map 的扩容采取了一种称为“渐进式”地方式，原有的 key 并不会一次性搬迁完毕，每次最多只会搬迁 2 个 bucket。

### 7. slice 和 map 分别作为函数参数时有什么区别？

makemap 和 makeslice 的区别，带来一个不同点：当 map 和 slice 作为函数参数时，在函数参数内部对 map 的操作会影响 map 自身；而对 slice 却不会（之前讲 slice 的文章里有讲过）。

主要原因：一个是指针（\*hmap），一个是结构体（slice）。Go 语言中的函数传参都是值传递，在函数内部，参数会被 copy 到本地。\*hmap 指针 copy 完之后，仍然指向同一个 map，因此函数内部对 map 的操作会影响实参。而 slice 被 copy 后，会成为一个新的 slice，对它进行的操作不会影响到实参。

### 8. 如何实现两种 get 操作

Go 语言中读取 map 有两种语法：带 comma 和 不带 comma。当要查询的 key 不在 map 里，带 comma 的用法会返回一个 bool 型变量提示 key 是否在 map 中；而不带 comma 的语句则会返回一个 key 对应 value 类型的零值。如果 value 是 int 型就会返回 0，如果 value 是 string 类型，就会返回空字符串。

源码里，函数命名不拘小节，直接带上后缀 1，2，mapaccess2 函数返回值多了一个 bool 型变量，两者的代码也是完全一样的，只是在返回值后面多加了一个 false 或者 true。

### 9. key 为什么是无序的

map 在扩容后，会发生 key 的搬迁，原来落在同一个 bucket 中的 key，搬迁后，有些 key 就要远走高飞了（bucket 序号加上了 2^B）。而遍历的过程，就是按顺序遍历 bucket，同时按顺序遍历 bucket 中的 key。搬迁后，key 的位置发生了重大的变化，有些 key 飞上高枝，有些 key 则原地不动。这样，遍历 map 的结果就不可能按原来的顺序了。

当我们在遍历 map 时，并不是固定地从 0 号 bucket 开始遍历，每次都是从一个随机值序号的 bucket 开始遍历，并且是从这个 bucket 的一个随机序号的 cell 开始遍历。这样，即使你是一个写死的 map，仅仅只是遍历它，也不太可能会返回一个固定序列的 key/value 对了。

### 10. float 类型可以作为 map 的 key 吗

从语法上看，是可以的。Go 语言中只要是可比较的类型都可以作为 key。

除开 slice，map，functions 这几种类型，其他类型都是 OK 的。

当用 float64 作为 key 的时候，先要将其转成 uint64 类型，再插入 key 中。

最后说结论：float 型可以作为 key，但是由于精度的问题，会导致一些诡异的问题，慎用之。

### 11. 可以边遍历边删除吗

map 并不是一个线程安全的数据结构。同时读写一个 map 是未定义的行为，如果被检测到，会直接 panic。

上面说的是发生在多个协程同时读写同一个 map 的情况下。 如果在同一个协程内边遍历边删除，并不会检测到同时读写，理论上是可以这样做的。但是，遍历的结果就可能不会是相同的了，有可能结果遍历结果集中包含了删除的 key，也有可能不包含，这取决于删除 key 的时间：是在遍历到 key 所在的 bucket 时刻前或者后。

一般而言，这可以通过读写锁来解决：sync.RWMutex。

读之前调用 RLock() 函数，读完之后调用 RUnlock() 函数解锁；写之前调用 Lock() 函数，写完之后，调用 Unlock() 解锁。

另外，sync.Map 是线程安全的 map，也可以使用。

### 12. 可以对 map 的元素取地址吗

无法对 map 的 key 或 value 进行取址。

如果通过其他 hack 的方式，例如 unsafe.Pointer 等获取到了 key 或 value 的地址，也不能长期持有，因为一旦发生扩容，key 和 value 的位置就会改变，之前保存的地址也就失效了。

### 13. 如何比较两个 map 相等

#### map 深度相等的条件

1. 都为 nil
2. 非空、长度相等，指向同一个 map 实体对象
3. 相应的 key 指向的 value “深度”相等

直接将使用 map1 == map2 是错误的。这种写法只能比较 map 是否为 nil。

因此只能是遍历 map 的每个元素，比较元素是否都是深度相等。

### 14. map 是线程安全的吗

map 不是线程安全的。

在查找、赋值、遍历、删除的过程中都会检测写标志，一旦发现写标志置位（等于 1），则直接 panic。赋值和删除函数在检测完写标志是复位之后，先将写标志位置位，才会进行之后的操作。

### 15. Go 语言与鸭子类型的关系

鸭子类型是一种动态语言的风格，在这种风格中，一个对象有效的语义，不是由继承自特定的类或实现特定的接口，而是由它 " 当前方法和属性的集合 " 决定。Go 作为一种静态语言，通过接口实现了鸭子类型，实际上是 Go 的编译器在其中作了隐匿的转换工作。

### 16. 值接收者和指针接收者的区别

如果实现了接收者是值类型的方法，会隐含地也实现了接收者是指针类型的方法。

### 17. iface 和 eface 的区别是什么

iface 和 eface 都是 Go 中描述接口的底层结构体，区别在于 iface 描述的接口包含方法，而 eface 则是不包含任何方法的空接口：interface{}。

### 18. 接口的动态类型和动态值

接口值的零值是指动态类型和动态值都为 nil。当仅且当这两部分的值都为 nil 的情况下，这个接口值就才会被认为 接口值 == nil。

### 19. 编译器自动检测类型是否实现接口

```go
var _ io.Writer = (*myWriter)(nil)
```

编译器会由此检查 \*myWriter 类型是否实现了 io.Writer 接口。

上述赋值语句会发生隐式地类型转换，在转换的过程中，编译器会检测等号右边的类型是否实现了等号左边接口所规定的函数。

### 20. 类型转换和断言的区别

类型转换、类型断言本质都是把一个类型转换成另外一个类型。不同之处在于，类型断言是对接口变量进行的操作。

### 21. channel 底层的数据结构

buf 指向底层循环数组，只有缓冲型的 channel 才有。

sendx，recvx 均指向底层循环数组，表示当前可以发送和接收的元素位置索引值（相对于底层数组）。

sendq，recvq 分别表示被阻塞的 goroutine，这些 goroutine 由于尝试读取 channel 或向 channel 发送数据而被阻塞。

waitq 是 sudog 的一个双向链表，而 sudog 实际上是对 goroutine 的一个封装

lock 用来保证每个读 channel 或写 channel 的操作都是原子的。

从函数原型来看，创建的 chan 是一个指针。所以我们能在函数间直接传递 channel，而不用传递 channel 的指针。

### 22. 从一个关闭的 channel 仍然能读出数据吗

从一个有缓冲的 channel 里读数据，当 channel 被关闭，依然能读出有效值。只有当返回的 ok 为 false 时，读出的数据才是无效的。

### 23. 操作 channel 的情况总结

| 操作     | nil channel | closed channel     | not nil, not closed channel                                                          |
| -------- | ----------- | ------------------ | ------------------------------------------------------------------------------------ |
| close    | panic       | panic              | 正常关闭                                                                             |
| 读 <- ch | 阻塞        | 读到对应类型的零值 | 阻塞或正常读取数据。缓冲型 channel 为空或非缓冲型 channel 没有等待发送者时会阻塞     |
| 写 ch <- | 阻塞        | panic              | 阻塞或正常写入数据。非缓冲型 channel 没有等待接收者或缓冲型 channel buf 满时会被阻塞 |

总结一下，发生 panic 的情况有三种：向一个关闭的 channel 进行写操作；关闭一个 nil 的 channel；重复关闭一个 channel。

读、写一个 nil channel 都会被阻塞。

### 24. 如何优雅地关闭 channel

不要从一个 receiver 侧关闭 channel，也不要在有多个 sender 时，关闭 channel。

解决方案就是增加一个传递关闭信号的 channel，receiver 通过信号 channel 下达关闭数据 channel 指令。senders 监听到关闭信号后，停止发送数据。

和上面的情况不同，这里有 M 个 receiver，由 receiver 直接关闭 stopCh 的话，就会重复关闭一个 channel，导致 panic。因此需要增加一个中间人，M 个 receiver 都向它发送关闭 dataCh 的“请求”，中间人收到第一个请求后，就会直接下达关闭 dataCh 的指令（通过关闭 stopCh，这时就不会发生重复关闭的情况，因为 stopCh 的发送方只有中间人一个）。另外，这里的 N 个 sender 也可以向中间人发送关闭 dataCh 的请求。

### 25. channel 发送和接收元素的本质是什么

就是说 channel 的发送和接收操作本质上都是 “值的拷贝”

### 26. channel 在什么情况下会引起资源泄漏

goroutine 操作 channel 后，处于发送或接收阻塞状态，而 channel 处于满或空的状态，一直得不到改变。同时，垃圾回收器也不会回收此类资源，进而导致 gouroutine 会一直处于等待队列中，不见天日。

另外，程序运行过程中，对于一个 channel，如果没有任何 goroutine 引用了，gc 会对其进行回收操作，不会引起内存泄漏。

### 27. channel 的应用

- 停止信号
- 任务定时
- 解耦生产方和消费方
- 控制并发数

### 28. context 是什么

它是 goroutine 的上下文，包含 goroutine 的运行状态、环境、现场等信息。

context 主要用来在 goroutine 之间传递上下文信息，包括：取消信号、超时时间、截止时间、k-v 等。

### 29. context 有什么作用

context 用来解决 goroutine 之间退出通知、元数据传递的功能。

- 传递共享的数据（RequestID）
- 取消 goroutine
- 防止 goroutine 泄漏

### 30. context.Value 的查找过程是怎样的

取值的过程，实际上是一个递归查找的过程

它会顺着链路一直往上找，比较当前节点的 key 是否是要找的 key，如果是，则直接返回 value。否则，一直顺着 context 往前，最终找到根节点（一般是 emptyCtx），直接返回一个 nil。所以用 Value 方法的时候要判断结果是否为 nil。

因为查找方向是往上走的，所以，父节点没法获取子节点存储的值，子节点却可以获取父节点的值。

### 31. 什么情况下需要使用反射

1. 不能明确接口调用哪个函数，需要根据传入的参数在运行时决定。
2. 不能明确传入函数的参数类型，需要在运行时处理任意对象。

### 32. 不推荐使用反射的理由

- 与反射相关的代码，经常是难以阅读的。在软件工程中，代码可读性也是一个非常重要的指标。
- Go 语言作为一门静态语言，编码过程中，编译器能提前发现一些类型错误，但是对于反射代码是无能为力的。所以包含反射相关的代码，很可能会运行很久，才会出错，这时候经常是直接 panic，可能会造成严重的后果。
- 反射对性能影响还是比较大的，比正常代码运行速度慢一到两个数量级。所以，对于一个项目中处于运行效率关键位置的代码，尽量避免使用反射特性。

### 33. 反射的应用

IDE 中的代码自动补全功能、对象序列化（encoding/json）、fmt 相关函数的实现、ORM（全称是：Object Relational Mapping，对象关系映射）

### 34. 如何比较两个对象完全相同

DeepEqual 函数的参数是两个 interface，实际上也就是可以输入任意类型，输出 true 或者 flase 表示输入的两个变量是否是“深度”相等。

### 35. Go 指针和 unsafe.Pointer 有什么区别

限制一：Go 的指针不能进行数学运算。

限制二：不同类型的指针不能相互转换。

限制三：不同类型的指针不能使用 == 或 != 比较。

限制四：不同类型的指针变量不能相互赋值。

unsafe 包提供了 2 点重要的能力：

- 任何类型的指针和 unsafe.Pointer 可以相互转换。
- uintptr 类型和 unsafe.Pointer 可以相互转换。

### 36. 如何利用 unsafe 获取 slice&map 的长度

我们可以通过 unsafe.Pointer 和 uintptr 进行转换，得到 slice 的字段值。

```go
var Len = *(*int)(unsafe.Pointer(uintptr(unsafe.Pointer(&s)) + uintptr(8)))

// &s => pointer => uintptr => pointer => *int => int
```

和 slice 不同的是，makemap 函数返回的是 hmap 的指针

我们依然能通过 unsafe.Pointer 和 uintptr 进行转换，得到 hamp 字段的值，只不过，现在 count 变成二级指针了

```go
count := **(**int)(unsafe.Pointer(&mp))

// &mp => pointer => **int => int
```

### 37. 如何实现字符串和 byte 切片的零拷贝转换

需要共享底层 Data 和 Len 就可以实现 zero-copy。

```go
func string2bytes(s string) []byte {
    return *(*[]byte)(unsafe.Pointer(&s))
}
func bytes2string(b []byte) string{
    return *(*string)(unsafe.Pointer(&b))
}
```

原理上是利用指针的强转

### 38. goroutine 和线程的区别

- 内存占用
  创建一个 goroutine 的栈内存消耗为 2 KB，实际运行过程中，如果栈空间不够用，会自动进行扩容。创建一个 thread 则需要消耗 1 MB 栈内存，而且还需要一个被称为 “a guard page” 的区域用于和其他 thread 的栈空间进行隔离。
- 创建和销毀
  Thread 创建和销毀都会有巨大的消耗，因为要和操作系统打交道，是内核级的，通常解决的办法就是线程池。而 goroutine 因为是由 Go runtime 负责管理的，创建和销毁的消耗非常小，是用户级。
- 切换
  当 threads 切换时，需要保存各种寄存器，以便将来恢复，而 goroutines 切换只需保存三个寄存器：Program Counter, Stack Pointer and BP。一般而言，线程切换会消耗 1000-1500 纳秒，一个纳秒平均可以执行 12-18 条指令。所以由于线程切换，执行指令的条数会减少 12000-18000。Goroutine 的切换约为 200 ns，相当于 2400-3600 条指令。

### 39. 为什么要 scheduler

Go scheduler 可以说是 Go 运行时的一个最重要的部分了。Runtime 维护所有的 goroutines，并通过 scheduler 来进行调度。Goroutines 和 threads 是独立的，但是 goroutines 要依赖 threads 才能执行。

Go 程序执行的高效和 scheduler 的调度是分不开的。

### 40. scheduler 底层原理

有三个基础的结构体来实现 goroutines 的调度。g，m，p。

g 代表一个 goroutine，它包含：表示 goroutine 栈的一些字段，指示当前 goroutine 的状态，指示当前运行到的指令地址，也就是 PC 值。

m 表示内核线程，包含正在运行的 goroutine 等字段。

p 代表一个虚拟的 Processor，它维护一个处于 Runnable 状态的 g 队列，m 需要获得 p 才能运行 g。

当然还有一个核心的结构体：sched，它总览全局。

Runtime 起始时会启动一些 G：垃圾回收的 G，执行调度的 G，运行用户代码的 G；并且会创建一个 M 用来开始 G 的运行。随着时间的推移，更多的 G 会被创建出来，更多的 M 也会被创建出来。

当然，在 Go 的早期版本，并没有 p 这个结构体，m 必须从一个全局的队列里获取要运行的 g，因此需要获取一个全局的锁，当并发量大的时候，锁就成了瓶颈。后来在大神 Dmitry Vyokov 的实现里，加上了 p 结构体。每个 p 自己维护一个处于 Runnable 状态的 g 的队列，解决了原来的全局锁问题。

Go 程序启动后，会给每个逻辑核心分配一个 P（Logical Processor）；同时，会给每个 P 分配一个 M（Machine，表示内核线程），这些内核线程仍然由 OS scheduler 来调度。

| 状态      | 解释                                                                                                                             |
| --------- | -------------------------------------------------------------------------------------------------------------------------------- |
| Waiting   | 等待状态，goroutine 在等待某件事的发生。例如等待网络数据、硬盘；调用操作系统 API；等待内存同步访问条件 ready，如 atomic, mutexes |
| Runnable  | 就绪状态，只要给 M 我就可以运行                                                                                                  |
| Executing | 运行状态。goroutine 在 M 上执行指令，这是我们想要的                                                                              |

### 41. goroutine 的调度时机

| 情形            | 说明                                                                                                                                                                                    |
| --------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 使用关键字 `go` | go 创建一个新的 goroutine，Go scheduler 会考虑调度                                                                                                                                      |
| GC              | 由于进行 GC 的 goroutine 也需要在 M 上运行，因此肯定会发生调度。当然，Go scheduler 还会做很多其他的调度，例如调度不涉及堆访问的 goroutine 来运行。GC 不管栈上的内存，只会回收堆上的内存 |
| 系统调用        | 当 goroutine 进行系统调用时，会阻塞 M，所以它会被调度走，同时一个新的 goroutine 会被调度上来                                                                                            |
| 内存同步访问    | atomic，mutex，channel 操作等会使 goroutine 阻塞，因此会被调度走。等条件满足后（例如其他 goroutine 解锁了）还会被调度上来继续运行                                                       |

### 42. 什么是 M:N 模型

Runtime 会在程序启动的时候，创建 M 个线程（CPU 执行调度的单位），之后创建的 N 个 goroutine 都会依附在这 M 个线程上执行。这就是 M:N 模型

在同一时刻，一个线程上只能跑一个 goroutine。当 goroutine 发生阻塞（例如上篇文章提到的向一个 channel 发送数据，被阻塞）时，runtime 会把当前 goroutine 调度走，让其他 goroutine 来执行。目的就是不让一个线程闲着，榨干 CPU 的每一滴油水。

### 43. 什么是工作窃取

当一个 P 发现自己的 LRQ 已经没有 G 时，会从其他 P “偷” 一些 G 来运行。看看这是什么精神！自己的工作做完了，为了全局的利益，主动为别人分担。这被称为 Work-stealing

### 44. GMP

G 取 goroutine 的首字母，主要保存 goroutine 的一些状态信息以及 CPU 的一些寄存器的值，例如 IP 寄存器，以便在轮到本 goroutine 执行时，CPU 知道要从哪一条指令处开始执行。

M 取 machine 的首字母，它代表一个工作线程，或者说系统线程。G 需要调度到 M 上才能运行，M 是真正工作的人。结构体 m 就是我们常说的 M，它保存了 M 自身使用的栈信息、当前正在 M 上执行的 G 信息、与之绑定的 P 信息。当 M 没有工作可做的时候，在它休眠前，会“自旋”地来找工作：检查全局队列，查看 network poller，试图执行 gc 任务，或者“偷”工作。

P 取 processor 的首字母，为 M 的执行提供“上下文”，保存 M 执行 G 时的一些资源，例如本地可运行 G 队列，memeory cache 等。一个 M 只有绑定 P 才能执行 goroutine，当 M 被阻塞时，整个 P 会被传递给其他 M ，或者说整个 P 被接管。

### 45. 什么是 GC，有什么作用？

当程序向操作系统申请的内存不再需要时，垃圾回收主动将其回收并供其他代码进行内存申请时候复用，或者将其归还给操作系统，这种针对内存级别资源的自动回收过程，即为垃圾回收。

### 46. 根对象到底是什么？

根对象在垃圾回收的术语中又叫做根集合，它是垃圾回收器在标记过程时最先检查的对象，包括：

- 全局变量：程序在编译期就能确定的那些存在于程序整个生命周期的变量。
- 执行栈：每个 goroutine 都包含自己的执行栈，这些执行栈上包含栈上的变量及指向分配的堆内存区块的指针。
- 寄存器：寄存器的值可能表示一个指针，参与计算的这些指针可能指向某些赋值器分配的堆内存区块。

### 47. STW 是什么意思？

STW 在垃圾回收过程中为了保证实现的正确性、防止无止境的内存增长等问题而不可避免的需要停止赋值器进一步操作对象图的一段过程。

在这个过程中整个用户代码被停止或者放缓执行， STW 越长，对用户代码造成的影响（例如延迟）就越大

### 48. Go 语言 GC(垃圾回收) 的工作原理

最常见的垃圾回收算法有标记清除 (Mark-Sweep) 和引用计数 (Reference Count)，Go 语言采用的是标记清除算法。并在此基础上使用了三色标记法和写屏障技术，提高了效率。

标记清除收集器是跟踪式垃圾收集器，其执行过程可以分成标记（Mark）和清除（Sweep）两个阶段：

- 标记阶段 — 从根对象出发查找并标记堆中所有存活的对象；
- 清除阶段 — 遍历堆中的全部对象，回收未被标记的垃圾对象并将回收的内存加入空闲链表。

标记清除算法的一大问题是在标记期间，需要暂停程序（Stop the world，STW），标记结束之后，用户程序才可以继续执行。为了能够异步执行，减少 STW 的时间，Go 语言采用了三色标记法。

三色标记算法将程序中的对象分成白色、黑色和灰色三类。

- 白色：不确定对象。
- 灰色：存活对象，子对象待处理。
- 黑色：存活对象。

标记开始时，所有对象加入白色集合（这一步需 STW ）。首先将根对象标记为灰色，加入灰色集合，垃圾搜集器取出一个灰色对象，将其标记为黑色，并将其指向的对象标记为灰色，加入灰色集合。重复这个过程，直到灰色集合为空为止，标记阶段结束。那么白色对象即可需要清理的对象，而黑色对象均为根可达的对象，不能被清理。

三色标记法因为多了一个白色的状态来存放不确定对象，所以后续的标记阶段可以并发地执行。当然并发执行的代价是可能会造成一些遗漏，因为那些早先被标记为黑色的对象可能目前已经是不可达的了。所以三色标记法是一个 false negative（假阴性）的算法。

三色标记法并发执行仍存在一个问题，即在 GC 过程中，对象指针发生了改变。比如下面的例子：

```txt
A (黑) -> B (灰) -> C (白) -> D (白)

正常情况下，D 对象最终会被标记为黑色，不应被回收。但在标记和用户程序并发执行过程中，用户程序删除了 C 对 D 的引用，而 A 获得了 D 的引用。标记继续进行，D 就没有机会被标记为黑色了（A 已经处理过，这一轮不会再被处理）。
```

```txt
A (黑) -> B (灰) -> C (白)
↓
D (白)
```

为了解决这个问题，Go 使用了内存屏障技术，它是在用户程序读取对象、创建新对象以及更新对象指针时执行的一段代码，类似于一个钩子。垃圾收集器使用了写屏障（Write Barrier）技术，当对象新增或更新时，会将其着色为灰色。这样即使与用户程序并发执行，对象的引用发生改变时，垃圾收集器也能正确处理了。

一次完整的 GC 分为四个阶段：

1. 标记准备 (Mark Setup，需 STW)，打开写屏障 (Write Barrier)
2. 使用三色标记法标记（Marking, 并发）
3. 标记结束 (Mark Termination，需 STW)，关闭写屏障。
4. 清理 (Sweeping, 并发)

### 49. SingleFlight

一般情况下我们在写一写对外的服务的时候都会有一层 cache 作为缓存，用来减少底层数据库的压力，但是在遇到例如 redis 抖动或者其他情况可能会导致大量的 cache miss 出现。

这时候就可以使用 singleflight 库了，直译过来就是单飞，这个库的主要作用就是将一组相同的请求合并成一个请求，实际上只会去请求一次，然后对所有的请求返回相同的结果。

### 50. Hertz Handler 中两个上下文的原因

核心原因在于请求上下文（RequestContext）的生命周期无法优雅的按需延长， 最终在各种设计权衡下，我们在路由的处理函数签名中增加一个标准的上下文入参，通过分离出生命周期长短各异的两个上下文的方式，从根本上解决各种因为上下文生命周期不一致导致的异常问题

### 52. mutex 有几种模式？

正常模式

所有 goroutine 按照 FIFO 的顺序进行锁获取，被唤醒的 goroutine 和新请求锁的 goroutine 同时进行锁获取，通常新请求锁的 goroutine 更容易获取锁 (持续占有 cpu)，被唤醒的 goroutine 则不容易获取到锁。公平性：否。

饥饿模式

所有尝试获取锁的 goroutine 进行等待排队，新请求锁的 goroutine 不会进行锁获取 (禁用自旋)，而是加入队列尾部等待获取锁。公平性：是。

### 53. string

底层有一个指针指向 []byte , 还有一个长度

string 并不能被修改

### 54. sync.map 原理

底层是两个 map，一个 read map，一个 dirty map，一开始读 read map，没有数据则加锁穿透去读 dirty map，并且读 dirty map 会记录一个计数，计数满的时候 read map 用 dirty map 进行覆盖

### 55. gRPC 有几种通信方式

gRPC 有四种通信方式

- 简单 RPC：客户端发送一个请求给服务端，服务端返回一个响应给客户端，就像一次普通的函数调用。
- 服务端流式 RPC：客户端发送一个请求给服务端，服务端返回一个数据流给客户端，适用于服务端返回的数据量比较大的情况。
- 客户端流式 RPC：客户端发送一个数据流给服务端，服务端返回一个响应给客户端，适用于客户端发送的数据量比较大的情况。
- 双向流式 RPC：双方都可以发送一个数据流到对方，适用于需要在长时间内保持连接并交换大量数据的场景。

### 56. new 和 make 的区别

make 和 new 都可以用来分配内存，但是它们的使用场景不同。make 只能用来分配及初始化类型为 slice、map、chan 的数据；而 new 可以分配任意类型的数据。new 分配返回的是指针，即类型“\*Type”；而 make 返回引用，即 Type。new 分配的空间会被清零；make 分配空间后，会进行初始化。

### 57. 如何终止一个运行中的协程

在 Go 中，可以使用 context 包来取消正在运行的 goroutine。context 包提供了一种在 goroutine 之间传递请求作用域的方法，包括取消信号。

### 58. slice 深度拷贝

在 Golang 中，slice 是一个引用类型，它的底层实现是一个结构体，包含了指向底层数组的指针、长度和容量。当你对一个 slice 进行拷贝时，你只是拷贝了这个结构体，而不是底层数组。因此，如果你修改了一个 slice 的元素，那么原始的 slice 和拷贝的 slice 都会受到影响。

如果你想要深度拷贝一个 slice，可以使用内置的 copy 函数。例如，如果你有两个 slice：sliceA 和 sliceB，你可以使用以下代码将 sliceB 深度拷贝到 sliceA：

```go
copy(sliceA, sliceB)
```

这将创建一个新的底层数组，并将其复制到 sliceA 中。这样，如果你修改了 sliceA 中的元素，sliceB 不会受到影响。

### 59. 协程创建子协程，如果子协程发生 panic，协程会 panic 吗

在 Go 语言中，如果一个协程发生 panic，它会导致整个程序退出。如果在多协程并发环境下，一个协程发生 panic，其他协程不会因为这个协程的 panic 而挂掉，每个协程只能捕获到自己的 panic，不能捕获其他协程的 panic。因此，如果子协程发生 panic，父协程不会 panic。

### 60. defer 的执行顺序

在 Go 语言中，defer 语句会在当前函数返回之前执行 defer 注册的函数。defer 语句的执行顺序是后进先出，也就是说，先被 defer 的语句最后执行。如果有多个 defer 语句，它们会以 LIFO（后进先出）的顺序执行。

### 61. = 和 := 的区别？

=是赋值变量，:=是定义变量。

### 62. 如何判断一个变量在栈还是在堆

在 Go 语言中，编译器会自动决定把一个变量放在栈还是放在堆，编译器会做逃逸分析 (escape analysis)，当发现变量的作用域没有跑出函数范围，就可以在栈上，反之则必须分配在堆。

如果变量离开作用域后没有被引用，则优先分配到栈上，否则分配到堆上。那么如何判断是否发生了逃逸呢？

`go build -gcflags '-m -m -l' xxx.go.`

关于逃逸的可能情况：变量大小不确定，变量类型不确定，变量分配的内存超过用户栈最大值，暴露给了外部指针。

### 63. Tag 的应用场景

在 Go 语言中，标签（tag）是一个结构体字段的元信息，可以在运行时通过反射读取。标签可以是任何字符串，但通常用于存储结构体字段的元数据，例如验证规则、ORM 映射等。标签的长度不受规格的限制。

除此之外，Go 语言中的 tags 还可以通过 `go build -tags` 实现编译控制，例如：项目中有如下文件代表不同的运行环境，通过 tag 控制不同环境下要编译的文件。

- json 序列化或反序列化时字段的名称
- db: sqlx 模块中对应的数据库字段名
- form: gin 框架中对应的前端的数据字段名
- binding: 搭配 form 使用, 默认如果没查找到结构体中的某个字段则不报错值为空, binding 为 required 代表没找到返回错误给前端

### 64. 怎么样输出一个有序的 map

- 创建一个 slice 来存储 map 中的 key。
- 遍历 map 并将 key 添加到 slice 中。
- 对 slice 进行排序。
- 遍历排序后的 slice 并输出 map 中的值。

### 65. Gin 框架的路由是如何实现的

gin 框架的路由是基于 httprouter 实现的，采用类似字典树一样的数据结构来存储路由与 handle 方法的映射。这也是框架高性能的原因之一。

### 66. Go 内存模型

Go 语言内存模型是基于 happens-before 关系的，它定义了在并发编程中，对共享变量进行读写时，不同的 goroutine 之间会产生什么样的同步和可见性保证。happens-before 关系是指在一个 goroutine 中，按照程序顺序，前面的操作 happens-before 于后续的任意操作；在不同的 goroutine 中，如果两个操作没有 happens-before 关系，那么它们就可以并发执行。

### 67. 锁释放后，等待中的 goroutine 中哪一个会优先获取 Mutex 呢？

Mutex 的获取是公平的，即等待时间最长的 goroutine 会最先尝试获取锁。具体来说，当 Mutex 被释放时，等待队列中的第一个 goroutine 会被唤醒，并尝试重新获取锁。如果该 goroutine 未能成功获取锁，则会继续等待，直到下一次 Mutex 被释放。因此，等待时间最长的 goroutine 会最优先获取 Mutex。

### 68. Mutex 底层实现

#### 初版的互斥锁

通过一个 flag 变量，标记当前的锁是否被某个 goroutine 持有。如果这个 flag 的值是 1，就代表锁已经被持有，那么，其它竞争的 goroutine 只能等待；如果这个 flag 的值是 0，就可以通过 CAS（compare-and-swap，或者 compare-and-set）将这个 flag 设置为 1，标识锁被当前的这个 goroutine 持有了。

Mutex 结构体包含两个字段：

- 字段 key：是一个 flag，用来标识这个排外锁是否被某个 goroutine 所持有，如果 key 大于等于 1，说明这个排外锁已经被持有；
- 字段 sema：是个信号量变量，用来控制等待 goroutine 的阻塞休眠和唤醒。

#### 第二版互斥锁

虽然 Mutex 结构体还是包含两个字段，但是第一个字段已经改成了 state，它的含义也不一样了。

state 是一个复合型的字段，一个字段包含多个意义，这样可以通过尽可能少的内存来实现互斥锁。这个字段的第一位（最小的一位）来表示这个锁是否被持有，第二位代表是否有唤醒的 goroutine，剩余的位数代表的是等待此锁的 goroutine 数。所以，state 这一个字段被分成了三部分，代表三个数据。

**相对于初版的设计，这次的改动主要就是，新来的 goroutine 也有机会先获取到锁，甚至一个 goroutine 可能连续获取到锁，打破了先来先得的逻辑。但是，代码复杂度也显而易见。**

#### 第三版互斥锁

如果新来的 goroutine 或者是被唤醒的 goroutine 首次获取不到锁，它们就会通过自旋（spin，通过循环不断尝试，spin 的逻辑是在 runtime 实现的）的方式，尝试检查锁是否被释放。在尝试一定的自旋次数后，再执行原来的逻辑。

对于临界区代码执行非常短的场景来说，这是一个非常好的优化。因为临界区的代码耗时很短，锁很快就能释放，而抢夺锁的 goroutine 不用通过休眠唤醒方式等待调度，直接 spin 几次，可能就获得了锁。

#### 最新版本

通过加入饥饿模式，可以避免把机会全都留给新来的 goroutine，保证了请求锁的 goroutine 获取锁的公平性，对于我们使用锁的业务代码来说，不会有业务一直等待锁不被处理。

### 69. 等待一个 Mutex 的 goroutine 数最大是多少？

Go 语言中 Mutex 的 goroutine 数最大是 536870911，这个数字是由 Mutex 的 state 类型决定的，目前是 int32，由于 3 个字节代表了状态，还有：2^(32 – 3) – 1 等于 536870911。

### 70. 如何设计一个可重入锁

- 方案一：goroutine id
- 方案二：token

### 71. 对 select 和 case 的理解

在 Go 语言中，select 语句类似于 switch 语句，但是 case 语句是针对通信的，即通道上的发送或接收操作。每个 case 必须是一个通道操作，要么是发送要么是接收。select 语句会监听所有指定的通道上的操作，一旦其中一个通道准备好就会执行相应的代码块。如果多个通道都准备好，那么 select 语句会随机选择一个通道执行。

### 72. 如何分析锁竞争的激烈程度

使用 Grafana 等监控关键互斥锁上等待的 goroutine 的数量，是我们分析锁竞争的激烈程度的一个重要指标。

### 73. RWMutex 的使用场景

如果你遇到可以明确区分 reader 和 writer goroutine 的场景，且有大量的并发读、少量的并发写，并且有强烈的性能需求，你就可以考虑使用读写锁 RWMutex 替换 Mutex。

### 74. Go 中 RWMutex 的策略

Go 标准库中的 RWMutex 设计是 Write-preferring 方案。一个正在阻塞的 Lock 调用会排除新的 reader 请求到锁。

写优先的设计意味着，如果已经有一个 writer 在等待请求锁的话，它会阻止新来的请求锁的 reader 获取到锁，所以优先保障 writer。当然，如果有一些 reader 已经请求了锁的话，新请求的 writer 也会等待已经存在的 reader 都释放锁之后才能获取。所以，写优先级设计中的优先权是针对新来的请求而言的。这种设计主要避免了 writer 的饥饿问题。

### 75. RWMutex 的 3 个踩坑点

1. 不可复制
2. 重入导致死锁
3. 释放未加锁的 RWMutex

### 76. 使用 WaitGroup 时的常见错误

1. 计数器设置为负值
2. 不期望的 Add 时机
3. 前一个 Wait 还没结束就重用 WaitGroup

#### 如何避免

- 不重用 WaitGroup。新建一个 WaitGroup 不会带来多大的资源开销，重用反而更容易出错。
- 保证所有的 Add 方法调用都在 Wait 之前。
- 不传递负数给 Add 方法，只通过 Done 来给计数值减 1。
- 不做多余的 Done 方法调用，保证 Add 的计数值和 Done 方法调用的数量是一样的。
- 不遗漏 Done 方法的调用，否则会导致 Wait hang 住无法返回。

### 77. noCopy：辅助 vet 检查

noCopy 字段的类型是 noCopy，它只是一个辅助的、用来帮助 vet 检查用的类型:

```go
type noCopy struct{}

// Lock is a no-op used by -copylocks checker from `go vet`.
func (*noCopy) Lock()   {}
func (*noCopy) Unlock() {}
```

如果你想要自己定义的数据结构不被复制使用，或者说，不能通过 vet 工具检查出复制使用的报警，就可以通过嵌入 noCopy 这个数据类型来实现。

### 78. Cond 怎么用

- Signal 方法，允许调用者 Caller 唤醒一个等待此 Cond 的 goroutine。如果此时没有等待的 goroutine，显然无需通知 waiter；如果 Cond 等待队列中有一个或者多个等待的 goroutine，则需要从等待队列中移除第一个 goroutine 并把它唤醒。在其他编程语言中，比如 Java 语言中，Signal 方法也被叫做 notify 方法。调用 Signal 方法时，不强求你一定要持有 c.L 的锁。
- Broadcast 方法，允许调用者 Caller 唤醒所有等待此 Cond 的 goroutine。如果此时没有等待的 goroutine，显然无需通知 waiter；如果 Cond 等待队列中有一个或者多个等待的 goroutine，则清空所有等待的 goroutine，并全部唤醒。在其他编程语言中，比如 Java 语言中，Broadcast 方法也被叫做 notifyAll 方法。同样地，调用 Broadcast 方法时，也不强求你一定持有 c.L 的锁。
- Wait 方法，会把调用者 Caller 放入 Cond 的等待队列中并阻塞，直到被 Signal 或者 Broadcast 的方法从等待队列中移除并唤醒。

Go 标准库提供 Cond 原语的目的是，为等待 / 通知场景下的并发问题提供支持。Cond 通常应用于等待某个条件的一组 goroutine，等条件变为 true 的时候，其中一个 goroutine 或者所有的 goroutine 都会被唤醒执行。

### 79. Cond 的常见错误

- 调用 Wait 的时候没有加锁
  - 如果调用 Wait 之前不加锁的话，就有可能 Unlock 一个未加锁的 Locker。
- 没有检查等待条件是否满足
  - waiter goroutine 被唤醒不等于等待条件被满足，只是有 goroutine 把它唤醒了而已。你也可以理解为，等待者被唤醒，只是得到了一次检查的机会而已。

### 80. Once 的使用场景

Once 常常用来初始化单例资源，或者并发访问只需初始化一次的共享资源，或者在测试的时候初始化一次测试资源。

### 81. 如何实现一个 Once

一个正确的 Once 实现要使用一个互斥锁，这样初始化的时候如果有并发的 goroutine，就会进入 doSlow 方法。互斥锁的机制保证只有一个 goroutine 进行初始化，同时利用双检查的机制（double-checking），再次判断 o.done 是否为 0，如果为 0，则是第一次执行，执行完毕后，就将 o.done 设置为 1，然后释放锁。

即使此时有多个 goroutine 同时进入了 doSlow 方法，因为双检查的机制，后续的 goroutine 会看到 o.done 的值为 1，也不会再次执行 f。

这样既保证了并发的 goroutine 会等待 f 完成，而且还不会多次执行 f。

```go
type Once struct {
    done uint32
    m    Mutex
}

func (o *Once) Do(f func()) {
    if atomic.LoadUint32(&o.done) == 0 {
        o.doSlow(f)
    }
}


func (o *Once) doSlow(f func()) {
    o.m.Lock()
    defer o.m.Unlock()
    // 双检查
    if o.done == 0 {
        defer atomic.StoreUint32(&o.done, 1)
        f()
    }
}
```

### 82. 使用 Once 可能出现的 2 种错误

1. 死锁

如果 f 中再次调用这个 Once 的 Do 方法的话，就会导致死锁的情况出现。这还不是无限递归的情况，而是的的确确的 Lock 的递归调用导致的死锁。

2. 未初始化

如果 f 方法执行的时候 panic，或者 f 执行初始化资源的时候失败了，这个时候，Once 还是会认为初次执行已经成功了，即使再次调用 Do 方法，也不会再次执行 f。

我们可以自己实现一个类似 Once 的并发原语，既可以返回当前调用 Do 方法是否正确完成，还可以在初始化失败后调用 Do 方法再次尝试初始化，直到初始化成功才不再初始化了。

### 83. 使用 struct 类型做 Map 的 key 有什么坑

如果 struct 的某个字段值修改了，查询 map 时无法获取它 add 进去的值。

如果要使用 struct 作为 key，我们要保证 struct 对象在逻辑上是不可变的，这样才会保证 map 的逻辑没有问题。

### 84. 使用 Map 的常见错误

- 未初始化
- 并发读写

### 85. 使用 sync.Map 的特殊场景

- 只会增长的缓存系统中，一个 key 只写入一次而被读很多次；
- 多个 goroutine 为不相交的键集读、写和重写键值对。

### 86. sync.Map 的实现

- 空间换时间。通过冗余的两个数据结构（只读的 read 字段、可写的 dirty），来减少加锁对性能的影响。对只读字段（read）的操作不需要加锁。
- 优先从 read 字段读取、更新、删除，因为对 read 字段的读取不需要锁。
- 动态调整。miss 次数多了之后，将 dirty 数据提升为 read，避免总是从 dirty 中加锁读取。
- double-checking。加锁之后先还要再检查 read 字段，确定真的不存在才操作 dirty 字段。
- 延迟删除。删除一个键值只是打标记，只有在提升 dirty 字段为 read 字段的时候才清理删除的数据。

### 87. sync.Pool 的实现原理

每次垃圾回收的时候，Pool 会把 victim 中的对象移除，然后把 local 的数据给 victim，这样的话，local 就会被清空，而 victim 就像一个垃圾分拣站，里面的东西可能会被当做垃圾丢弃了，但是里面有用的东西也可能被捡回来重新使用。

victim 中的元素如果被 Get 取走，那么这个元素就很幸运，因为它又“活”过来了。但是，如果这个时候 Get 的并发不是很大，元素没有被 Get 取走，那么就会被移除掉，因为没有别人引用它的话，就会被垃圾回收掉。

### 88. sync.Pool 的坑

- 内存泄漏
  - 在使用 sync.Pool 回收 buffer 的时候，一定要检查回收的对象的大小。如果 buffer 太大，就不要回收了，否则就太浪费了。
- 内存浪费

### 89. Context 的 Done 方法返回什么

如果 Done 没有被 close，Err 方法返回 nil；如果 Done 被 close，Err 方法会返回 Done 被 close 的原因。

### 90. 对一个地址的赋值是原子操作吗？

对于现代的多处理多核的系统来说，由于 cache、指令重排，可见性等问题，我们对原子操作的意义有了更多的追求。

atomic 包提供的方法会提供内存屏障的功能，所以，atomic 不仅仅可以保证赋值的数据完整性，还能保证数据的可见性，一旦一个核更新了该地址的值，其它处理器总是能读取到它的最新值。

### 91. 使用信号量的常见错误

- 请求了资源，但是忘记释放它；
- 释放了从未请求的资源；
- 长时间持有一个资源，即使不需要它；
- 不持有一个资源，却直接使用它。

### 92. 使用信号量的场景

在批量获取资源的场景中，我建议你尝试使用官方扩展的信号量。

### 93. CyclicBarrier 的用处

CyclicBarrier 允许一组 goroutine 彼此等待，到达一个共同的执行点。同时，因为它可以被重复使用，所以叫循环栅栏。具体的机制是，大家都在栅栏前等待，等全部都到齐了，就抬起栅栏放行。

### 94. ErrGroup 的作用

ErrGroup 是 Go 官方提供的一个同步扩展库。我们经常会碰到需要将一个通用的父任务拆成几个小任务并发执行的场景，其实，将一个大的任务拆成几个小任务并发执行，可以有效地提高程序的并发度。

ErrGroup 就是用来应对这种场景的。它和 WaitGroup 有些类似，但是它提供功能更加丰富：

- 和 Context 集成；
- error 向上传播，可以把子任务的错误传递给 Wait 的调用者。

### 95. Gin 为什么使用前缀树作为路由数据结构

1. 高效的路由匹配：前缀树可以实现高效的路由匹配。在 Gin 框架中，路由是通过比较 URL 路径和已注册的路由路径来进行匹配的。使用前缀树可以将 URL 路径分解为一系列的字符节点，并通过前缀匹配快速定位到对应的路由节点，减少了不必要的比较操作，提高了路由匹配的效率。
    
2. 灵活的路由配置：前缀树允许在每个节点上存储额外的信息，例如 HTTP 方法（GET、POST 等）和处理函数。这样，Gin 框架可以根据前缀树节点上存储的信息，灵活地配置路由规则和处理逻辑。节点的子节点可以代表路径的不同部分，使得可以方便地构建出各种路由规则，支持动态路由和参数匹配。
    
3. 节省空间：前缀树可以共享相同的前缀，从而节省存储空间。在 Gin 框架中，相同前缀的路由路径可以共享相同的节点，避免了冗余存储。这对于大规模的路由配置来说，可以显著减少内存占用。

### 96. Go 的多路复用模型与数据结构

在 Go 语言中，多路复用模型通常指的是使用 `net` 包中的 `Listen` 和 `Accept` 函数实现的并发网络服务器。它允许服务器同时处理多个客户端连接，而无需为每个连接创建一个新的操作系统线程。

多路复用模型的核心数据结构是 `net.Listener` 和 `net.Conn`。`net.Listener` 用于监听指定的网络地址，接受客户端的连接请求，而 `net.Conn` 代表一个客户端连接。

通常，多路复用模型使用 `goroutine` 和 `channel` 来实现并发处理多个连接。当有新的连接到达时，`Accept` 函数会返回一个新的 `net.Conn` 对象，然后可以将该对象传递给一个新的 `goroutine` 进行处理。这样，每个连接都可以在独立的 `goroutine` 中执行，实现并发处理。

### 97. 如果将 Listener 关闭，那么之前已经 Accept 的连接是否会关闭

在 Go 中，如果你关闭了一个 `net.Listener`，那么之前已经通过该监听器接受的连接不会自动关闭。关闭监听器只会停止接受新的连接请求，已经接受的连接将继续保持打开状态。

如果你希望在关闭监听器时同时关闭所有已经接受的连接，你需要在关闭监听器之前显式地关闭每个已接受的连接。

### 98.  如何判断读文件结束了

1. 使用 `bufio.NewScanner` 创建一个文件扫描器 `scanner`，然后通过循环调用 `scanner.Scan()` 来逐行读取文件内容。当 `scanner.Scan()` 返回 `false` 时，表示已经读取到文件末尾。
2. 使用 `file.Read` 从文件中读取数据，并判断读取的字节数 `n`。当 `n` 为 0 时，表示已经读取到文件末尾。

### 99. 如何在打开一次文件后，读到末尾，不重新打开文件，再读一次

可以使用 `os.Seek` 和 `os.File.Seek` 函数将文件指针移动到文件的开头，以便重新读取文件内容。

```go
// 将文件指针移动到开头
	_, err = file.Seek(0, 0)
```

### 100. new 一个 map 结构会有什么问题

使用 `make` 函数来创建一个空的 `map` 结构是推荐的做法，而不是使用 `new` 关键字。使用 `new` 关键字创建 `map` 结构可能会导致以下问题：

1. `new` 函数返回的是指向该类型零值的指针。而 `map` 类型的零值是 `nil`，不能直接用于存储键值对。如果你使用 `new` 创建一个 `map`，会得到一个 `nil` 指针，当你尝试向该 `map` 中存储键值对时，会触发运行时错误。
    
2. `new` 函数只会为 `map` 类型分配存储空间，而不会进行初始化。这意味着你无法立即开始向该 `map` 中添加键值对，因为它的内部数据结构还没有被初始化。如果你尝试在一个通过 `new` 创建的 `map` 上执行添加操作，会导致运行时错误。

### 101. 传数组和传切片有什么区别

当数组作为函数参数时，函数操作的是数组的一个副本，不会影响原始数组；当切片作为函数参数时，函数操作的是切片的引用，会影响原始切片。

### 102. 为什么 bmap 里面存储的是八个键值对

1. 内存分配效率：固定大小的桶可以在预分配的内存块上操作，避免频繁的内存分配和释放，提高性能。
2. 冲突处理：哈希表中可能存在冲突，即不同的键映射到了同一个桶中。通过使用固定大小的桶，可以在桶内部使用更高效的冲突解决方法，例如链表或开放寻址法。
3. 空间利用：通过固定大小的桶，可以在一定程度上减少内存空间的浪费。如果每个桶的大小过小，会导致内存碎片和额外的内存开销；如果每个桶的大小过大，会导致内存浪费。

### 103. 多分支映射的声明

开发中有种常见的 **多分支路由** 需求，即定义一组条件和对应逻辑，判断传入参数值，并路由到对应分支逻辑上执行。通常我们会用 **switch case** 语句来处理，比如下面是个格式化 Level 的例子：

```go
type Level int

const (
    LevelA Level = 1
    ...
    LevelC Level = 3
)

func (l Level) Format() string {
    switch l {
    case LevelA:
        return "A"
    ...
    case LevelC:
        return "C"
    }
    return "" 
} 
```

由于 switch case 是顺序判读条件的，当分支数量变多时(大约八个以上)，switch case 效率会下降。考虑到分支数量的可扩展性，多数有经验的工程师会使用 **map** 来代替 switch case 做条件映射。比如上例可改为：

```go
...

var levels = map[Level]string {
    LevelA: "A",
    ...
    LevelC: "C",
}

func (l Level) Format() string {
    return levels[l]
} 
```

如果 const 定义为数字并且数值不大，还可以使用 **slice** 来代替 map，比上述两种做法更好。Go 源码里也有类似做法，如 syscall.Errno 的实现等。对于上例，可以修改为：

```go
var levels = [...]string {
    LevelA: "A",
    ...
    LevelC: "C",
}

func (l Level) Format() string {
    return levels[int(l)]
}
```

### 104. 刀法灵活的切片

#### 容量声明

1. 使用 var 声明切片不会开辟内存空间；直接赋空值会分配内存空间，后续的扩容会额外重新分配内存。

```go
// Bad Case: 赋空值会占内存空间，并在 append 后重新分配新的内存空间
buf := []string{}
buf = append(buf, "a", "b", "c"...)

// Good Case: var 声明不占内存空间，并在 append 后第一次直接的分配适当的内存空间
var buf []string
buf = append(buf, "a", "b", "c"...)
```

2. 声明切片时，可以预留冗余容量，这样当调整切片大小时，仍然是基于当前物理地址操作，而不会开辟新的内存空间。

```go
// Base Case: 没有预留容量，append 后分配了新的内存空间，并复制转移了原切片数据，开销较大
buf := make([]string, 3)
buf = append(buf, "a", "b", "c"...)

// Good Case: 预留了足够的容量，append 后，buf 仍然指向当前的物理地址
buf := make([]string, 3, 8)
buf = append(buf, "a", "b", "c"...)
```

#### 下标操作

1. 切片不能越界访问超过 len 的下标，但是可以调整切片大小，灵活访问

```go
var buf = make([]string, 3, 8)

// Bad Case: 不能越界访问下标，因此不能直接访问 下标6
_ = buf[6]

// Good Case: 可以这么访问/修改 下标6
_ = buf[:8][6]
buf[:8][6] = "a"
```

2. 结合 append 可以实现在数组指定位置插入或者删除元素，而不占用新的存储空间

```go
buf := make([]string, 0, 8)
buf = append(buf, "a", "b", "c", "d") // buf = "a","b","c","d"

// 1. 删除元素 "c"
buf = append(buf[:2], buf[3:]...) // 删除了 下标2 对应的元素 "c", 操作后 buf = "a","b","d"

// 2. 插入元素 "c"
buf = append(buf[:3], buf[2:]...) // 下标2 复制了两遍, 数组长度增加 1
buf[2] = "c" // 将第一个 下标2 改为 "c"，实现插入，操作后 buf = "a","b","c","d"
```

3. 共享同一块内存地址的多个切片，内容相互可见；但是如果 append 导致了扩容，新创建的切片将无法共享内存

```go
a := []string{"a", "b", "c", "d"}
b := a[:2]

// 1. 修改 b, 内存地址不变，a 同样被修改
b = append(b, "e", "f") // 修改后，a = "a","b","e","f"

// 2. 继续 append 导致扩容，b 被分配了新的内存空间，不再和 a 共享
b = append(b, "m", "n") // 修改后，a = "a","b","e","f"; b = "a","b","e","f","m","n"
```

### 105. unsafe.Pointer 最佳实践

根据定义知道以下几点：

- []byte 与 string 在定义上的区别仅在于多一个 cap 属性
- string 的 str 指向的是内容是不可修改的，每更改一次 string 的值，就要重分配一次内存

那么思考一下，如果我们可以 **保证 []byte 是只读的**，比如做反序列化、打日志等，符合不修改的定义，是不是可以不用复制，直接零成本和 string 互转呢 ？答案是可以，通过 unsafe.Pointer 可以实现，展示如下：

```go
func ZeroCopyStringToBytes(s string) (b []byte) {
        bh := (*reflect.SliceHeader)(unsafe.Pointer(&b))
        sh := (*reflect.StringHeader)(unsafe.Pointer(&s))
        bh.Data = sh.Data
        bh.Len = sh.Len
        bh.Cap = sh.Len
        return b
}

func ZeroCopyBytesToString(b []byte) string {
        return *(*string)(unsafe.Pointer(&b))
}
```

通过这种方式进行类型转换，可以规避大量的 copy、内存分配和 GC 开销，在处理大量数据流、序列化等场景下收益十分可观。但是一定要注意：必须保证 []byte 不会被修改。

### 106. 字符串拼接速度对比

```go
// 以下比较, 序号越小越快

// 简单场景，+ 拼接即可
1. s := "Error" + ": " + err.Error()
2. s := fmt.Sprintf("Error: %s", err.Error())

// string 和数字拼接，strconv 先转再 + 更快
1. s := "hello world " + strconv.Itoa(2022)
2. s := fmt.Sprintf("hello world %d", 2022)

// string 和 []byte 拼接，可以使用 1.4节 的 "零拷贝转换"; 类型强转会造成 copy
bytes := make([]byte, 1024)
1. s := "abc " + ZeroCopyBytesToString(bytes)
2. s := fmt.Sprintf("abc %s", bytes)
3. s := "abc" + string(bytes)

// 大量拼接, strings.Builder 最快, 但是操作较为繁琐
var builder strings.Builder
builder.Grow(length)
builder.WriteString("abc...")
builder.WriteString(strconv.Itoa(123...))
builder.Write([]byte...)
...
s := builder.String()
builder.Reset()
```

### 107. 格式化优化技巧

以打印日志举例，常常会见到使用 `%v`、`%+v`、`%#v` 来打印 struct 类型的参数，但是这种方式会带来大量反射，计算开销是惊人的，其效率甚至低于 `json.Marshal`。

对于这种情况，最佳实践是在 struct 上实现 `.String()` 方法，并使用 `%s` 占位，来实现 format；当然即使是偷懒，也建议使用 `json.Marshal` 处理一下，效果要比 `%v` 好。

**Bad Case**:

```go
var arg = &Arg{}

// Bad Case: 使用 %v 打印 struct 参数
logs.Info("... %v", arg)
```

**Good Case**:

```go
var arg = &Arg{}

// Good Case: 实现 String() 方法，并使用 %s 打印 struct 参数
func (arg *Arg) String() string {
    return arg.A + arg.B + ...
}

logs.Info("... %s", arg)

// Good Case: 直接拼接
logs.Info("... " + arg.String())
```

**Lazy Case**:

```go
var arg = &Arg{}

// Lazy Case: 使用 json 先行序列化
logs.Info("... %s", json.Marshal(arg))

// Lazy Case: 使用其他序列化工具 先行序列化
import "github.com/bytedance/sonic"
logs.Info("... %s", sonic.Marshal(arg))
...
```

### 108. 编译后执行效率优化

#### goto 减少 cache miss

在 Go 开发中，频繁出现的 if 条件分支会增大编译产物体积，编译后分支逻辑太多，CPU 执行容易导致 cache miss，而因此更新 cache line 会短暂的中断 CPU 计算，影响效率。

针对这一点，如果代码中有大量的 if return 逻辑，我们推荐使用 goto 语法来减少编译产物的体积，从而减少 cache miss。以下举例展示

**Bad Case**:

```go
// Bad Case: 方法存在大量 if return 逻辑
func A() (err error) {
    if err = cause1; err != nil {
        return FormatError(err)
    }
    if err = cause2; err != nil {
        return FormatError(err)
    }
    ...
    return nil
}
```

**Good Case**:

```go
// Good Case: if return 改为 if goto，并统一 return
func A() (err error) {
    if err = cause1; err != nil {
        goto Error 
    }
    if err = cause2; err != nil {
        goto Error 
    }
    ...
    return nil
Error:
    return FormatError(err)
}
```

#### 传值与传指针对比

在 Go 程序中，管理内存同时使用 栈 和 堆 两种数据结构，简单来说，栈一般用来存放临时变量，内存分配和释放较快；堆一般用来生命周期不确定的变量，内存分配慢，还有 GC 代价，因此 堆变量 比 栈变量 开销大很多。

当变量具有指针对象时，内存会逃逸到堆上，从这个角度考虑，**传指针** 比 **传值** 开销更大；另一方面，**传值** 比 **传指针** 复制了更多的数据。两方面综合看，**传值 or 传指针** 比较的其实是 **复制开销** 和 **堆开销** 哪个更小。因此我们可以给出最佳实践：

1. 如果参数本身数据量不大，复制速度更快，那么传值更优；
2. 如果参数本身数据量大，复制慢，那么传指针效率更高。

以下展示一些对比 case：

**Case 1**: struct 较小时，应直接传递值

```go
type Args struct{
    Num1 int
    ...
    Num5 int
}
var arg = Args{...}

// Bad Case: Args 仅有 5个 int 值，使用 &Args{} 造成内存逃逸，效率较低
func A(arg *Args){...}
func ...
    A(&arg)

// Good Case: 直接 复制 Args 本身(5个 int)，效率更高
func A(arg Args){...}
func ...
    A(arg)  
```

**Case 2**: slice 较小时，可以定义为数组

```go
// Bad Case: 临时变量定义为 []int 会造成内存逃逸，效率较低
func ...
    tmp := []int{1, 2, 3}
    
// Good Case: 临时变量定义为数组，分配在栈上，效率更高
func ...
    tmp := [3]int{1, 2, 3}
```

### 109. 相辅相成的协程

#### 协程复用

每个 Goroutine 初始 stack size 为 2KB ，在协程内调用函数时，会通过 `morestack` 判断是否需要栈扩张，如需要，则需要调用 `copystack` 拷贝完整的栈。

而在现实中，大部分线上程序的运行模式都有一定的固定规律，例如一个 Server 所有请求都会经过一系列固定的函数调用链，而如果这些函数最终会导致 stack 被固定扩张到 8KB，那么这个程序每一次请求都会重复调用 `copystack` 。基于此，我们希望每一个请求都能**尽可能复用前一个请求已经扩张好栈**的 Goroutine：

```go
type Pool struct {
   tasks chan func()
}

func (p *Pool) Go(task func()) {
   select {
   case p.tasks <- task: // reuse exist worker
      return
   default:
   }
   go func() { // start a new worker
      for {
         task()
         task = <-p.tasks // waiting for new task
      }
   }()
}
```

以上是一个最简单的 Goroutine 池实现，我们并未考虑超时销毁等逻辑。如果该系统的并发度是 100 ，那么我们仅需要常驻 100 个 Goroutine 便可以完成所有任务。

#### 协程协作

Go 社区有一句金玉良言是「不要通过共享内存来通信，而应该通过通信来共享内存」。由于协程可以轻量切换上下文的，我们可以轻易地通过通信的方式将内存对象“传输”给其他需要使用的协程，而无需通过加互斥锁的方式来保证并发安全性。

在并发编程中，生产者消费者模型是最常见的编程场景，我们就以此举例，来看看在 Go 中如何达成多协程并发协作的目的。利用 Go 语言内置的 channel 数据结构，我们仅需几行代码就能够实现一个最简单的消息队列：

**简单的生产者消费者模型**

```go
type Factory struct {
   queue chan string
}

func (f *Factory) Produce(msg string) {
   f.queue <- val
}

func (f *Factory) Consume() {
   for msg := range f.queue {
      handler(msg)
   }
}
```

#### 批量消费

基于 Channel 实现的消息队列有一个缺陷是获取一个元素的代价过大，且无法一次操作就获得全部的元素。在许多场景中我们希望能够批量消费当前所有元素。此时我们可以利用加锁的链表实现消息队列，同时利于 Channel 作为协程间的信号通知器：

```go
type ListFactory struct {
   mu       sync.Mutex
   queue    []string
   wakeup   chan struct{} // size=1
}

func NewListFactory() *ListFactory {
    f := new(ListFactory)
    f.wakeup = make(chan struct{}, 1)
    return f
}

func (f *ListFactory) Produce(msg string) {
   f.mu.Lock()
   f.queue = append(f.queue, msg)
   if len(f.queue) == 1 {
      select {
      case f.wakeup <- struct{}{}:
      default:
      }
   }
   f.mu.Unlock()
}

func (f *ListFactory) Consume() {
   for {
      f.mu.Lock()
      cache := f.queue
      f.queue = nil
      f.mu.Unlock()
      batchHandler(cache)

      <-f.wakeup
   }
}
```

#### 提升消费效率

在上面的实现里，我们的消费者 Goroutine 每当队列中有 1 条消息，就有可能会唤醒执行。而在有些场景下，我们希望消费者能够积蓄一定数量的消息再进行批量处理，典型的场景如异步的日志写入，数据的合并发送等。最简单的做法是每次 Sleep 一个窗口时间去积累消息发送，但这样做的代价是，即便后续没有任何数据需要写入，我们整体消费速度都会被拖长到至少一个窗口时间。

在这种情况下，我们可以巧用 `runtime.Gosched()` 实现有限度的延迟等待:

```go
func (f *ListFactory) Consume() {
   for {
      runtime.Gosched() // 等待生产者发送更多消息
      f.mu.Lock()
      cache := f.queue
      f.queue = nil
      f.mu.Unlock()
      batchHandler(cache)

      <-f.wakeup
   }
}

func main() {
   N := 32
   f := new(ListFactory)
   go func() {
      for i := 0; i < N; i++ {
         f.Produce("hello")
         runtime.Gosched() // 模拟线上系统，降低生产速度。每次调度机会都只生产一条消息。
      }
   }()
   f.Consume()
}
```

在程序处于繁忙状态时，会有大量（假设为 N 个） Goroutine 不停调用 Produce 函数，而消费者的 Goroutine 仅有 1 个，假设每个 Goroutine 的调度机会相同，那么消费者 Goroutine 每获得一次调度机会，意味着生产者 Goroutine 至少已经生产了 N 条消息，这样便批量消费至少 N 条消息（理想情况下）。

而如果该模型处于空闲状态下，生产者的 Goroutine 数量非常少，而此时即便消费者调用 runtime.Gosched()，也能够很快由于没有其他 Goroutine 需要被调度而获得执行机会，从而尽快地消费。

### 110. 精打细算的锁

#### 降低锁粒度

假设我们需要实现一个网站浏览量计数器，其中 UV 是不太容易变更的变量，PV 是高频变更的变量。我们可以实现如下的代码：

```go
type Website struct {
   mu sync.Mutex
   uv int // unique visitors, rarely change
   pv int // page views, frequently change
}

func (w *Website) Add(uv, pv bool) {
   w.mu.Lock()
   if uv {
      w.uv++
   }
   if pv {
      w.pv++
   }
   w.mu.Unlock()
}
```

上述代码将两个不同访问频率的变量都用同一个锁控制，加大了锁冲突的概率，尤其是对于不频繁改变的 UV 变量来说增加了没必要的锁冲突。所以我们可以将这两个变量用不同的锁来保护：

```go
func (w *WebsiteSplit) AddUV() {
   w.muv.Lock()
   w.uv++
   w.muv.Unlock()
}

func (w *WebsiteSplit) AddPV() {
   w.mpv.Lock()
   w.pv++
   w.mpv.Unlock()
}
```

#### 分片锁

既然并发时使用同一个变量容易产生冲突，那么我们还有另一个思路是在不同情况下去使用不同的变量以减少冲突。分片锁就是这样的实现。

分片锁顾名思义要求该场景首先是要能够被分片的，典型场景如 Map：

```go
func NewMap() *Map {
   return &Map{
      locks: make([]sync.Mutex, runtime.GOMAXPROCS(0)),
      mm:    make([]map[string]string, runtime.GOMAXPROCS(0)),
   }
}

type Map struct {
   locks []sync.Mutex
   mm    []map[string]string
}

func (o *Map) Set(key, val string) {
   shard := hash(key) % len(o.locks)
   o.locks[shard].Lock()
   o.mm[shard][key] = val
   o.locks[shard].Unlock()
}
```

这里我们将分片数量设置为 Go runtime 中 Process 的数量，每次 Set 时，都根据 key 计算属于哪一个分片，然后只去修改该分片的对象。理论上我们能够将锁冲突的概率降低到分片数分之一。

### 111. 追求极致的 No GC

#### 复用对象

我们假设某个程序每 1 ms 会创建一个新对象，每个对象会被使用 10 ms 的时间，那么该程序每秒会创建 1000 个对象，而同一时刻，系统内仅仅只有 10 个对象正在被使用。在这个例子中我们很容易发现，我们完全可以仅创建 10 个对象，使它们反复被程序使用，这样就大大降低了创建的对象数，从而大大降低了 GC 扫描对象的成本。

不过在实际编程时，我们往往还要考虑到多个协程间并发申请对象的问题。好在 Go 标准库为我们贴心地提供了 sync.Pool 来实现该需求：

```go
var objectPool sync.Pool

func NewObject() *Object {
   obj := objectPool.Get()
   if obj == nil { return &Object{} }
   return obj.(*Object)
}

type Object struct {
   Name string
}

func (o *Object) Recycle() {
   o.Name = "" // reset object
   objectPool.Put(o)
}

func main() {
   obj := NewObject()
   obj.Recycle()
}
```

对于我们想要复用的对象，我们只需要为其实现 Recycle() 方法，在程序确定对象不再需要被使用时，调用 .Recycle() ，便可将其重置并释放，留给程序下一次需要对象时使用。

需要注意的是，对象能够被复用的前提是我们能够精确控制对象的使用生命周期，如果我们提前释放了一个正在其他地方被使用的对象，有可能引发不可预料的错误。

### 112. 数组与编译

```go
arr1 := [3]int{1, 2, 3}
arr2 := [...]int{1, 2, 3}
```

上述两种声明方式在运行期间得到的结果是完全相同的，后一种声明方式在编译期间就会被转换成前一种，这也就是编译器对数组大小的推导。

1. 当元素数量小于或者等于 4 个时，会直接将数组中的元素放置在栈上；
2. 当元素数量大于 4 个时，会将数组中的元素放置到静态区并在运行时取出；

数组访问越界是非常严重的错误，Go 语言中可以在编译期间的静态类型检查判断数组越界，数组和字符串的一些简单越界错误都会在编译期间发现，例如：直接使用整数或者常量访问数组；但是如果使用变量去访问数组或者字符串时，编译器就无法提前发现错误，我们需要 Go 语言运行时阻止不合法的访问：

```go
arr[4]: invalid array index 4 (out of bounds for 3-element array)
arr[i]: panic: runtime error: index out of range [4] with length 3
```

Go 语言运行时在发现数组、切片和字符串的越界操作会由运行时的 [`runtime.panicIndex`](https://draveness.me/golang/tree/runtime.panicIndex) 和 [`runtime.goPanicIndex`](https://draveness.me/golang/tree/runtime.goPanicIndex) 触发程序的运行时错误并导致崩溃退出。

### 113. 切片与编译

从切片的定义我们能推测出，切片在编译期间的生成的类型只会包含切片中的元素类型，即 `int` 或者 `interface{}` 等。

使用 `append` 关键字向切片中追加元素也是常见的切片操作，中间代码生成阶段的 [`cmd/compile/internal/gc.state.append`](https://draveness.me/golang/tree/cmd/compile/internal/gc.state.append) 方法会根据返回值是否会覆盖原变量，选择进入两种流程，最大的区别在于得到的新切片是否会赋值回原变量。如果我们选择覆盖原有的变量，就不需要担心切片发生拷贝影响性能，因为 Go 语言编译器已经对这种常见的情况做出了优化。

相比于依次拷贝元素，[`runtime.memmove`](https://draveness.me/golang/tree/runtime.memmove) 能够提供更好的性能。需要注意的是，整块拷贝内存仍然会占用非常多的资源，在大切片上执行拷贝操作时一定要注意对性能的影响。

### 114. 理解 Go 中哈希表的原理

#### 数据结构

```go
type hmap struct {
	count     int
	flags     uint8
	B         uint8
	noverflow uint16
	hash0     uint32

	buckets    unsafe.Pointer
	oldbuckets unsafe.Pointer
	nevacuate  uintptr

	extra *mapextra
}

type mapextra struct {
	overflow    *[]*bmap
	oldoverflow *[]*bmap
	nextOverflow *bmap
}
```

1. `count` 表示当前哈希表中的元素数量；
2. `B` 表示当前哈希表持有的 `buckets` 数量，但是因为哈希表中桶的数量都 2 的倍数，所以该字段会存储对数，也就是 `len(buckets) == 2^B`；
3. `hash0` 是哈希的种子，它能为哈希函数的结果引入随机性，这个值在创建哈希表时确定，并在调用哈希函数时作为参数传入；
4. `oldbuckets` 是哈希在扩容时用于保存之前 `buckets` 的字段，它的大小是当前 `buckets` 的一半；

哈希表 [`runtime.hmap`](https://draveness.me/golang/tree/runtime.hmap) 的桶是 [`runtime.bmap`](https://draveness.me/golang/tree/runtime.bmap)。每一个 [`runtime.bmap`](https://draveness.me/golang/tree/runtime.bmap) 都能存储 8 个键值对，当哈希表中存储的数据过多，单个桶已经装满时就会使用 `extra.nextOverflow` 中桶存储溢出的数据。

上述两种不同的桶在内存中是连续存储的，我们在这里将它们分别称为正常桶和溢出桶，上图中黄色的 [`runtime.bmap`](https://draveness.me/golang/tree/runtime.bmap) 就是正常桶，绿色的 [`runtime.bmap`](https://draveness.me/golang/tree/runtime.bmap) 是溢出桶，溢出桶是在 Go 语言还使用 C 语言实现时使用的设计，由于它能够减少扩容的频率所以一直使用至今。

桶的结构体 [`runtime.bmap`](https://draveness.me/golang/tree/runtime.bmap) 在 Go 语言源代码中的定义只包含一个简单的 `tophash` 字段，`tophash` 存储了键的哈希的高 8 位，通过比较不同键的哈希的高 8 位可以减少访问键值对次数以提高性能：

```go
type bmap struct {
	tophash [bucketCnt]uint8
}
```

#### 初始化

##### 字面量

```go
hash := map[string]int{
	"1": 2,
	"3": 4,
	"5": 6,
}
```

当哈希表中的元素数量少于或者等于 25 个时，编译器会将字面量初始化的结构体转换成以下的代码，将所有的键值对一次加入到哈希表中：

```go
hash := make(map[string]int, 3)
hash["1"] = 2
hash["3"] = 4
hash["5"] = 6
```

这种初始化的方式与的[数组](https://draveness.me/golang/docs/part2-foundation/ch03-datastructure/golang-array/)和[切片](https://draveness.me/golang/docs/part2-foundation/ch03-datastructure/golang-array-and-slice/)几乎完全相同，由此看来集合类型的初始化在 Go 语言中有着相同的处理逻辑。

一旦哈希表中元素的数量超过了 25 个，编译器会创建两个数组分别存储键和值，这些键值对会通过 for 循环加入哈希。

不过无论使用哪种方法，使用字面量初始化的过程都会使用 Go 语言中的关键字 `make` 来创建新的哈希并通过最原始的 `[]` 语法向哈希追加元素。
##### 运行时

当创建的哈希被分配到栈上并且其容量小于 `BUCKETSIZE = 8` 时，Go 语言在编译阶段会快速初始化哈希，这也是编译器对小容量的哈希做的优化。
除了上述特定的优化之外，无论 `make` 是从哪里来的，只要我们使用 `make` 创建哈希，Go 语言编译器都会在[类型检查](https://draveness.me/golang/docs/part1-prerequisite/ch02-compile/golang-typecheck/)期间将它们转换成 [`runtime.makemap`](https://draveness.me/golang/tree/runtime.makemap)，使用字面量初始化哈希也只是语言提供的辅助工具，最后调用的都是 [`runtime.makemap`](https://draveness.me/golang/tree/runtime.makemap)。

这个函数会按照下面的步骤执行：

1. 计算哈希占用的内存是否溢出或者超出能分配的最大值；
2. 调用 [`runtime.fastrand`](https://draveness.me/golang/tree/runtime.fastrand) 获取一个随机的哈希种子；
3. 根据传入的 `hint` 计算出需要的最小需要的桶的数量；
4. 使用 [`runtime.makeBucketArray`](https://draveness.me/golang/tree/runtime.makeBucketArray) 创建用于保存桶的数组；[`runtime.makeBucketArray`](https://draveness.me/golang/tree/runtime.makeBucketArray) 会根据传入的 `B` 计算出的需要创建的桶数量并在内存中分配一片连续的空间用于存储数据

- 当桶的数量小于 2^4 时，由于数据较少、使用溢出桶的可能性较低，会省略创建的过程以减少额外开销；
- 当桶的数量多于 2^4 时，会额外创建 2^(B-4) 个溢出桶；

#### 读写操作

##### 访问

[`runtime.mapaccess1`](https://draveness.me/golang/tree/runtime.mapaccess1) 会先通过哈希表设置的哈希函数、种子获取当前键对应的哈希，再通过 [`runtime.bucketMask`](https://draveness.me/golang/tree/runtime.bucketMask) 和 [`runtime.add`](https://draveness.me/golang/tree/runtime.add) 拿到该键值对所在的桶序号和哈希高位的 8 位数字。哈希会依次遍历正常桶和溢出桶中的数据，它会先比较哈希的高 8 位和桶中存储的 `tophash`，后比较传入的和桶中的值以加速数据的读写。用于选择桶序号的是哈希的最低几位，而用于加速访问的是哈希的高 8 位，这种设计能够减少同一个桶中有大量相等 `tophash` 的概率影响性能。

每一个桶都是一整片的内存空间，当发现桶中的 `tophash` 与传入键的 `tophash` 匹配之后，我们会通过指针和偏移量获取哈希中存储的键 `keys[0]` 并与 `key` 比较，如果两者相同就会获取目标值的指针 `values[0]` 并返回。

另一个同样用于访问哈希表中数据的 [`runtime.mapaccess2`](https://draveness.me/golang/tree/runtime.mapaccess2) 只是在 [`runtime.mapaccess1`](https://draveness.me/golang/tree/runtime.mapaccess1) 的基础上多返回了一个标识键值对是否存在的 `bool` 值。

##### 写入

函数会根据传入的键拿到对应的哈希和桶，然后通过遍历比较桶中存储的 `tophash` 和键的哈希，如果找到了相同结果就会返回目标位置的地址。其中 `inserti` 表示目标元素的在桶中的索引，`insertk` 和 `val` 分别表示键值对的地址，获得目标地址之后会通过算术计算寻址获得键值对 `k` 和 `val`。

上述的 for 循环会依次遍历正常桶和溢出桶中存储的数据，整个过程会分别判断 `tophash` 是否相等、`key` 是否相等，遍历结束后会从循环中跳出。

如果当前桶已经满了，哈希会调用 [`runtime.hmap.newoverflow`](https://draveness.me/golang/tree/runtime.hmap.newoverflow) 创建新桶或者使用 [`runtime.hmap`](https://draveness.me/golang/tree/runtime.hmap) 预先在 `noverflow` 中创建好的桶来保存数据，新创建的桶不仅会被追加到已有桶的末尾，还会增加哈希表的 `noverflow` 计数器。

##### 扩容

[`runtime.mapassign`](https://draveness.me/golang/tree/runtime.mapassign) 函数会在以下两种情况发生时触发哈希的扩容：

1. 装载因子已经超过 6.5；
2. 哈希使用了太多溢出桶；

根据触发的条件不同扩容的方式分成两种，如果这次扩容是溢出的桶太多导致的，那么这次扩容就是等量扩容 `sameSizeGrow`。

哈希在扩容的过程中会通过 [`runtime.makeBucketArray`](https://draveness.me/golang/tree/runtime.makeBucketArray) 创建一组新桶和预创建的溢出桶，随后将原有的桶数组设置到 `oldbuckets` 上并将新的空桶设置到 `buckets` 上，溢出桶也使用了相同的逻辑更新。

我们在 [`runtime.hashGrow`](https://draveness.me/golang/tree/runtime.hashGrow) 中还看不出来等量扩容和翻倍扩容的太多区别，等量扩容创建的新桶数量只是和旧桶一样，该函数中只是创建了新的桶，并没有对数据进行拷贝和转移。哈希表的数据迁移的过程在是 [`runtime.evacuate`](https://draveness.me/golang/tree/runtime.evacuate) 中完成的，它会对传入桶中的元素进行再分配。

[`runtime.evacuate`](https://draveness.me/golang/tree/runtime.evacuate) 会将一个旧桶中的数据分流到两个新桶，所以它会创建两个用于保存分配上下文的 [`runtime.evacDst`](https://draveness.me/golang/tree/runtime.evacDst) 结构体，这两个结构体分别指向了一个新桶。

如果这是等量扩容，那么旧桶与新桶之间是一对一的关系，所以两个 [`runtime.evacDst`](https://draveness.me/golang/tree/runtime.evacDst) 只会初始化一个。而当哈希表的容量翻倍时，每个旧桶的元素会都分流到新创建的两个桶中。

>我们简单总结一下哈希表扩容的设计和原理，哈希在存储元素过多时会触发扩容操作，每次都会将桶的数量翻倍，扩容过程不是原子的，而是通过 [`runtime.growWork`](https://draveness.me/golang/tree/runtime.growWork) 增量触发的，在扩容期间访问哈希表时会使用旧桶，向哈希表写入数据时会触发旧桶元素的分流。除了这种正常的扩容之外，为了解决大量写入、删除造成的内存泄漏问题，哈希引入了 `sameSizeGrow` 这一机制，在出现较多溢出桶时会整理哈希的内存减少空间的占用。

##### 删除

如果想要删除哈希中的元素，就需要使用 Go 语言中的 `delete` 关键字，这个关键字的唯一作用就是将某一个键对应的元素从哈希表中删除，无论是该键对应的值是否存在，这个内建的函数都不会返回任何的结果。

### 115. 字符串原理设计

只读只意味着字符串会分配到只读的内存空间，但是 Go 语言只是不支持直接修改 `string` 类型变量的内存空间，我们仍然可以通过在 `string` 和 `[]byte` 类型之间反复转换实现修改这一目的：

1. 先将这段内存拷贝到堆或者栈上；
2. 将变量的类型转换成 `[]byte` 后并修改字节数据；
3. 将修改后的字节数组转换回 `string`；

与切片的结构体相比，字符串只少了一个表示容量的 `Cap` 字段，而正是因为切片在 Go 语言的运行时表示与字符串高度相似，所以我们经常会说字符串是一个只读的切片类型。

### 116. 函数调用

Go 通过栈传递函数的参数和返回值，在调用函数之前会在栈上为返回值分配合适的内存空间，随后将入参从右到左按顺序压栈并拷贝参数，返回值会被存储到调用方预留好的栈空间上，我们可以简单总结出以下几条规则：

1. 通过堆栈传递参数，入栈的顺序是从右到左，而参数的计算是从左到右；
2. 函数返回值通过堆栈传递并由调用者预先分配内存空间；
3. 调用函数时都是传值，接收方会对入参进行复制再计算；
#### 参数传递

不同语言会选择不同的方式传递参数，Go 语言选择了传值的方式，**无论是传递基本类型、结构体还是指针，都会对传递的参数进行拷贝**。

### 117. 接口 itab 结构体设计

#### itab 结构体

[`runtime.itab`](https://draveness.me/golang/tree/runtime.itab) 结构体是接口类型的核心组成部分，每一个 [`runtime.itab`](https://draveness.me/golang/tree/runtime.itab) 都占 32 字节，我们可以将其看成接口类型和具体类型的组合，它们分别用 `inter` 和 `_type` 两个字段表示：

```go
type itab struct { // 32 字节
	inter *interfacetype
	_type *_type
	hash  uint32
	_     [4]byte
	fun   [1]uintptr
}
```

Go

除了 `inter` 和 `_type` 两个用于表示类型的字段之外，上述结构体中的另外两个字段也有自己的作用：

- `hash` 是对 `_type.hash` 的拷贝，当我们想将 `interface` 类型转换成具体类型时，可以使用该字段快速判断目标类型和具体类型 [`runtime._type`](https://draveness.me/golang/tree/runtime._type) 是否一致；
- `fun` 是一个动态大小的数组，它是一个用于动态派发的虚函数表，存储了一组函数指针。虽然该变量被声明成大小固定的数组，但是在使用时会通过原始指针获取其中的数据，所以 `fun` 数组中保存的元素数量是不确定的；

### 118. 反射法则

1. 从 `interface{}` 变量可以反射出反射对象；
2. 从反射对象可以获取 `interface{}` 变量；
3. 要修改反射对象，其值必须可设置；

```go
v := reflect.ValueOf(1)
v.Interface().(int)
```

从反射对象到接口值的过程是从接口值到反射对象的镜面过程，两个过程都需要经历两次转换：

- 从接口值到反射对象：
    - 从基本类型到接口类型的类型转换；
    - 从接口类型到反射对象的转换；
- 从反射对象到接口值：
    - 反射对象转换成接口类型；
    - 通过显式类型转换变成原始类型；

由于 Go 语言的函数调用都是传值的，所以我们得到的反射对象跟最开始的变量没有任何关系，那么直接修改反射对象无法改变原始变量，程序为了防止错误就会崩溃。

想要修改原变量只能使用如下的方法：

```go
func main() {
	i := 1
	v := reflect.ValueOf(&i)
	v.Elem().SetInt(10)
	fmt.Println(i)
}

$ go run reflect.go
10
```

1. 调用 [`reflect.ValueOf`](https://draveness.me/golang/tree/reflect.ValueOf) 获取变量指针；
2. 调用 [`reflect.Value.Elem`](https://draveness.me/golang/tree/reflect.Value.Elem) 获取指针指向的变量；
3. 调用 [`reflect.Value.SetInt`](https://draveness.me/golang/tree/reflect.Value.SetInt) 更新变量的值；

#### 类型和值

当我们想要将一个变量转换成反射对象时，Go 语言会在编译期间完成类型转换，将变量的类型和值转换成了 `interface{}` 并等待运行期间使用 [`reflect`](https://golang.org/pkg/reflect/) 包获取接口中存储的信息。

#### 更新变量

当我们想要更新 [`reflect.Value`](https://draveness.me/golang/tree/reflect.Value) 时，就需要调用 [`reflect.Value.Set`](https://draveness.me/golang/tree/reflect.Value.Set) 更新反射对象，该方法会调用 [`reflect.flag.mustBeAssignable`](https://draveness.me/golang/tree/reflect.flag.mustBeAssignable) 和 [`reflect.flag.mustBeExported`](https://draveness.me/golang/tree/reflect.flag.mustBeExported) 分别检查当前反射对象是否是可以被设置的以及字段是否是对外公开的：

```go
func (v Value) Set(x Value) {
	v.mustBeAssignable()
	x.mustBeExported()
	var target unsafe.Pointer
	if v.kind() == Interface {
		target = v.ptr
	}
	x = x.assignTo("reflect.Set", v.typ, target)
	typedmemmove(v.typ, v.ptr, x.ptr)
}
```

[`reflect.Value.Set`](https://draveness.me/golang/tree/reflect.Value.Set) 会调用 [`reflect.Value.assignTo`](https://draveness.me/golang/tree/reflect.Value.assignTo) 并返回一个新的反射对象，这个返回的反射对象指针会直接覆盖原反射变量。

[`reflect.Value.assignTo`](https://draveness.me/golang/tree/reflect.Value.assignTo) 会根据当前和被设置的反射对象类型创建一个新的 [`reflect.Value`](https://draveness.me/golang/tree/reflect.Value) 结构体：

- 如果两个反射对象的类型是可以被直接替换，就会直接返回目标反射对象；
- 如果当前反射对象是接口并且目标对象实现了接口，就会把目标对象简单包装成接口值；

在变量更新的过程中，[`reflect.Value.assignTo`](https://draveness.me/golang/tree/reflect.Value.assignTo) 返回的 [`reflect.Value`](https://draveness.me/golang/tree/reflect.Value) 中的指针会覆盖当前反射对象中的指针实现变量的更新。

#### 实现协议

[`reflect`](https://golang.org/pkg/reflect/) 包还为我们提供了 [`reflect.rtype.Implements`](https://draveness.me/golang/tree/reflect.rtype.Implements) 方法可以用于判断某些类型是否遵循特定的接口。在 Go 语言中获取结构体的反射类型 [`reflect.Type`](https://draveness.me/golang/tree/reflect.Type) 还是比较容易的，但是想要获得接口类型需要通过以下方式：

```go
reflect.TypeOf((*<interface>)(nil)).Elem()
```

我们通过一个例子在介绍如何判断一个类型是否实现了某个接口。假设我们需要判断如下代码中的 `CustomError` 是否实现了 Go 语言标准库中的 `error` 接口：

```go
type CustomError struct{}

func (*CustomError) Error() string {
	return ""
}

func main() {
	typeOfError := reflect.TypeOf((*error)(nil)).Elem()
	customErrorPtr := reflect.TypeOf(&CustomError{})
	customError := reflect.TypeOf(CustomError{})

	fmt.Println(customErrorPtr.Implements(typeOfError)) // #=> true
	fmt.Println(customError.Implements(typeOfError)) // #=> false
}
```

### 119. for 与 range

#### 现象

##### 循环永动机

如果我们在遍历数组的同时修改数组的元素，能否得到一个永远都不会停止的循环呢？

```go
func main() {
	arr := []int{1, 2, 3}
	for _, v := range arr {
		arr = append(arr, v)
	}
	fmt.Println(arr)
}

$ go run main.go
1 2 3 1 2 3
```

上述代码的输出意味着循环只遍历了原始切片中的三个元素，我们在遍历切片时追加的元素不会增加循环的执行次数，所以循环最终还是停了下来。

对于所有的 range 循环，Go 语言都会在编译期将原切片或者数组赋值给一个新变量 `ha`，在赋值的过程中就发生了拷贝，而我们又通过 `len` 关键字预先获取了切片的长度，所以在循环中追加新的元素也不会改变循环执行的次数，这也就解释了循环永动机的现象。
##### 神奇的指针

第二个例子是使用 Go 语言经常会犯的错误。当我们在遍历一个数组时，如果获取 `range` 返回变量的地址并保存到另一个数组或者哈希时，会遇到令人困惑的现象，下面的代码会输出 “3 3 3”：

```go
func main() {
	arr := []int{1, 2, 3}
	newArr := []*int{}
	for _, v := range arr {
		newArr = append(newArr, &v)
	}
	for _, v := range newArr {
		fmt.Println(*v)
	}
}

$ go run main.go
3 3 3
```

一些有经验的开发者不经意也会犯这种错误，正确的做法应该是使用 `&arr[i]` 替代 `&v`。

##### 遍历清空数组

当我们想要在 Go 语言中清空一个切片或者哈希时，一般都会使用以下的方法将切片中的元素置零：

```go
func main() {
	arr := []int{1, 2, 3}
	for i, _ := range arr {
		arr[i] = 0
	}
}
```

依次遍历切片和哈希看起来是非常耗费性能的，因为数组、切片和哈希占用的内存空间都是连续的，所以最快的方法是直接清空这片内存中的内容。

[`cmd/compile/internal/gc.arrayClear`](https://draveness.me/golang/tree/cmd/compile/internal/gc.arrayClear) 是一个非常有趣的优化，它会优化 Go 语言遍历数组或者切片并删除全部元素的逻辑：

```go
// 原代码
for i := range a {
	a[i] = zero
}

// 优化后
if len(a) != 0 {
	hp = &a[0]
	hn = len(a)*sizeof(elem(a))
	memclrNoHeapPointers(hp, hn)
	i = len(a) - 1
}
```

相比于依次清除数组或者切片中的数据，Go 语言会直接使用 [`runtime.memclrNoHeapPointers`](https://draveness.me/golang/tree/runtime.memclrNoHeapPointers) 或者 [`runtime.memclrHasPointers`](https://draveness.me/golang/tree/runtime.memclrHasPointers) 清除目标数组内存空间中的全部数据，并在执行完成后更新遍历数组的索引，这也印证了我们在遍历清空数组中观察到的现象。

##### 随机遍历

当我们在 Go 语言中使用 `range` 遍历哈希表时，往往都会使用如下的代码结构，但是这段代码在每次运行时都会打印出不同的结果：

```go
func main() {
	hash := map[string]int{
		"1": 1,
		"2": 2,
		"3": 3,
	}
	for k, v := range hash {
		println(k, v)
	}
}
```

两次运行上述代码可能会得到不同的结果，第一次会打印 `2 3 1`，第二次会打印 `1 2 3`，如果我们运行的次数足够多，最后会得到几种不同的遍历顺序。

```bash
$ go run main.go
2 2
3 3
1 1

$ go run main.go
1 1
2 2
3 3
```

Go 语言在运行时为哈希表的遍历引入了不确定性，也是告诉所有 Go 语言的使用者，程序不要依赖于哈希表的稳定遍历。

#### 经典循环

##### 哈希表

首先会选出一个绿色的正常桶开始遍历，随后遍历所有黄色的溢出桶，最后依次按照索引顺序遍历哈希表中其他的桶，直到所有的桶都被遍历完成。

##### 字符串

遍历字符串的过程与数组、切片和哈希表非常相似，只是在遍历时会获取字符串中索引对应的字节并将字节转换成 `rune`。我们在遍历字符串时拿到的值都是 `rune` 类型的变量，`for i, r := range s {}` 的结构都会被转换成如下所示的形式：

```go
ha := s
for hv1 := 0; hv1 < len(ha); {
    hv1t := hv1
    hv2 := rune(ha[hv1])
    if hv2 < utf8.RuneSelf {
        hv1++
    } else {
        hv2, hv1 = decoderune(ha, hv1)
    }
    v1, v2 = hv1t, hv2
}
```

字符串是一个只读的字节数组切片，所以范围循环在编译期间生成的框架与切片非常类似，只是细节有一些不同。

使用下标访问字符串中的元素时得到的就是字节，但是这段代码会将当前的字节转换成 `rune` 类型。如果当前的 `rune` 是 ASCII 的，那么只会占用一个字节长度，每次循环体运行之后只需要将索引加一，但是如果当前 `rune` 占用了多个字节就会使用 [`runtime.decoderune`](https://draveness.me/golang/tree/runtime.decoderune) 函数解码，具体的过程就不在这里详细介绍了。

##### 通道

使用 range 遍历 Channel 也是比较常见的做法，一个形如 `for v := range ch {}` 的语句最终会被转换成如下的格式：

```go
ha := a
hv1, hb := <-ha
for ; hb != false; hv1, hb = <-ha {
    v1 := hv1
    hv1 = nil
    ...
}
```

这里的代码可能与编译器生成的稍微有一些出入，但是结构和效果是完全相同的。该循环会使用 `<-ch` 从管道中取出等待处理的值，这个操作会调用 [`runtime.chanrecv2`](https://draveness.me/golang/tree/runtime.chanrecv2) 并阻塞当前的协程，当 [`runtime.chanrecv2`](https://draveness.me/golang/tree/runtime.chanrecv2) 返回时会根据布尔值 `hb` 判断当前的值是否存在：

- 如果不存在当前值，意味着当前的管道已经被关闭；
- 如果存在当前值，会为 `v1` 赋值并清除 `hv1` 变量中的数据，然后重新陷入阻塞等待新数据；

### 120. select 设计

C 语言的 `select` 系统调用可以同时监听多个文件描述符的可读或者可写的状态，Go 语言中的 `select` 也能够让 Goroutine 同时等待多个 Channel 可读或者可写，在多个文件或者 Channel 状态改变之前，`select` 会一直阻塞当前线程或者 Goroutine。

`select` 是与 `switch` 相似的控制结构，与 `switch` 不同的是，`select` 中虽然也有多个 `case`，但是这些 `case` 中的表达式必须都是 Channel 的收发操作。下面的代码就展示了一个包含 Channel 收发操作的 `select` 结构：

```go
func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
		select {
		case c <- x:
			x, y = y, x+y
		case <-quit:
			fmt.Println("quit")
			return
		}
	}
}
```

上述控制结构会等待 `c <- x` 或者 `<-quit` 两个表达式中任意一个返回。无论哪一个表达式返回都会立刻执行 `case`中的代码，当 `select` 中的两个 `case` 同时被触发时，会随机执行其中的一个。

#### 现象

当我们在 Go 语言中使用 `select` 控制结构时，会遇到两个有趣的现象：

1. `select` 能在 Channel 上进行非阻塞的收发操作；
2. `select` 在遇到多个 Channel 同时响应时，会随机执行一种情况；

##### 非阻塞的收发

在通常情况下，`select` 语句会阻塞当前 Goroutine 并等待多个 Channel 中的一个达到可以收发的状态。但是如果 `select` 控制结构中包含 `default` 语句，那么这个 `select` 语句在执行时会遇到以下两种情况：

1. 当存在可以收发的 Channel 时，直接处理该 Channel 对应的 `case`；
2. 当不存在可以收发的 Channel 时，执行 `default` 中的语句；

当我们运行下面的代码时就不会阻塞当前的 Goroutine，它会直接执行 `default` 中的代码。

```go
func main() {
	ch := make(chan int)
	select {
	case i := <-ch:
		println(i)

	default:
		println("default")
	}
}

$ go run main.go
default
```

非阻塞的 Channel 发送和接收操作还是很有必要的，在很多场景下我们不希望 Channel 操作阻塞当前 Goroutine，只是想看看 Channel 的可读或者可写状态，如下所示：

```go
errCh := make(chan error, len(tasks))
wg := sync.WaitGroup{}
wg.Add(len(tasks))
for i := range tasks {
    go func() {
        defer wg.Done()
        if err := tasks[i].Run(); err != nil {
            errCh <- err
        }
    }()
}
wg.Wait()

select {
case err := <-errCh:
    return err
default:
    return nil
}
```

在上面这段代码中，我们不关心到底多少个任务执行失败了，只关心是否存在返回错误的任务，最后的 `select` 语句能很好地完成这个任务。

#### 实现原理

##### 常见流程

在默认的情况下，编译器会使用如下的流程处理 `select` 语句：

1. 将所有的 `case` 转换成包含 Channel 以及类型等信息的 [`runtime.scase`](https://draveness.me/golang/tree/runtime.scase) 结构体；
2. 调用运行时函数 [`runtime.selectgo`](https://draveness.me/golang/tree/runtime.selectgo) 从多个准备就绪的 Channel 中选择一个可执行的 [`runtime.scase`](https://draveness.me/golang/tree/runtime.scase) 结构体；
3. 通过 `for` 循环生成一组 `if` 语句，在语句中判断自己是不是被选中的 `case`；

一个包含三个 `case` 的正常 `select` 语句其实会被展开成如下所示的逻辑，我们可以看到其中处理的三个部分：

```go
selv := [3]scase{}
order := [6]uint16
for i, cas := range cases {
    c := scase{}
    c.kind = ...
    c.elem = ...
    c.c = ...
}
chosen, revcOK := selectgo(selv, order, 3)
if chosen == 0 {
    ...
    break
}
if chosen == 1 {
    ...
    break
}
if chosen == 2 {
    ...
    break
}
```

展开后的代码片段中最重要的就是用于选择待执行 `case` 的运行时函数 [`runtime.selectgo`](https://draveness.me/golang/tree/runtime.selectgo)，这也是我们要关注的重点。因为这个函数的实现比较复杂， 所以这里分两部分进行执行过程：

1. 执行一些必要的初始化操作并确定 `case` 的处理顺序；
2. 在循环中根据 `case` 的类型做出不同的处理；

#### 小结

1. 空的 `select` 语句会被转换成调用 [`runtime.block`](https://draveness.me/golang/tree/runtime.block) 直接挂起当前 Goroutine；
2. 如果 `select` 语句中只包含一个 `case`，编译器会将其转换成 `if ch == nil { block }; n;`表达式；
    - 首先判断操作的 Channel 是不是空的；
    - 然后执行 `case` 结构中的内容；
3. 如果 `select` 语句中只包含两个 `case` 并且其中一个是 `default`，那么会使用 [`runtime.selectnbrecv`](https://draveness.me/golang/tree/runtime.selectnbrecv) 和 [`runtime.selectnbsend`](https://draveness.me/golang/tree/runtime.selectnbsend) 非阻塞地执行收发操作；
4. 在默认情况下会通过 [`runtime.selectgo`](https://draveness.me/golang/tree/runtime.selectgo) 获取执行 `case` 的索引，并通过多个 `if` 语句执行对应 `case` 中的代码；

在编译器已经对 `select` 语句进行优化之后，Go 语言会在运行时执行编译期间展开的 [`runtime.selectgo`](https://draveness.me/golang/tree/runtime.selectgo) 函数，该函数会按照以下的流程执行：

1. 随机生成一个遍历的轮询顺序 `pollOrder` 并根据 Channel 地址生成锁定顺序 `lockOrder`；
2. 根据 `pollOrder` 遍历所有的 `case` 查看是否有可以立刻处理的 Channel；
    1. 如果存在，直接获取 `case` 对应的索引并返回；
    2. 如果不存在，创建 [`runtime.sudog`](https://draveness.me/golang/tree/runtime.sudog) 结构体，将当前 Goroutine 加入到所有相关 Channel 的收发队列，并调用 [`runtime.gopark`](https://draveness.me/golang/tree/runtime.gopark) 挂起当前 Goroutine 等待调度器的唤醒；
3. 当调度器唤醒当前 Goroutine 时，会再次按照 `lockOrder` 遍历所有的 `case`，从中查找需要被处理的 [`runtime.sudog`](https://draveness.me/golang/tree/runtime.sudog) 对应的索引；

`select` 关键字是 Go 语言特有的控制结构，它的实现原理比较复杂，需要编译器和运行时函数的通力合作。

### 121. defer

#### 现象

##### 作用域

向 `defer` 关键字传入的函数会在函数返回之前运行。假设我们在 `for` 循环中多次调用 `defer` 关键字：

```go
func main() {
	for i := 0; i < 5; i++ {
		defer fmt.Println(i)
	}
}

$ go run main.go
4
3
2
1
0
```

运行上述代码会倒序执行传入 `defer` 关键字的所有表达式，因为最后一次调用 `defer` 时传入了 `fmt.Println(4)`，所以这段代码会优先打印 4。

```go
func main() {
    {
        defer fmt.Println("defer runs")
        fmt.Println("block ends")
    }
    
    fmt.Println("main ends")
}

$ go run main.go
block ends
main ends
defer runs
```

从上述代码的输出我们会发现，`defer` 传入的函数不是在退出代码块的作用域时执行的，它只会在当前函数和方法返回之前被调用。

##### 预计算参数

```go
func main() {
	startedAt := time.Now()
	defer fmt.Println(time.Since(startedAt))
	
	time.Sleep(time.Second)
}

$ go run main.go
0s
```

我们会发现调用 `defer` 关键字会立刻拷贝函数中引用的外部参数，所以 `time.Since(startedAt)` 的结果不是在 `main` 函数退出之前计算的，而是在 `defer` 关键字调用时计算的，最终导致上述代码输出 0s。

想要解决这个问题的方法非常简单，我们只需要向 `defer` 关键字传入匿名函数：

```go
func main() {
	startedAt := time.Now()
	defer func() { fmt.Println(time.Since(startedAt)) }()
	
	time.Sleep(time.Second)
}

$ go run main.go
1s
```

虽然调用 `defer` 关键字时也使用值传递，但是因为拷贝的是函数指针，所以 `time.Since(startedAt)` 会在 `main` 函数返回前调用并打印出符合预期的结果。

#### 数据结构

[`runtime._defer`](https://draveness.me/golang/tree/runtime._defer) 结构体是延迟调用链表上的一个元素，所有的结构体都会通过 `link` 字段串联成链表。

#### 执行机制

##### 堆上分配

根据 [`cmd/compile/internal/gc.state.stmt`](https://draveness.me/golang/tree/cmd/compile/internal/gc.state.stmt) 方法对 `defer` 的处理我们可以看出，堆上分配的 [`runtime._defer`](https://draveness.me/golang/tree/runtime._defer)结构体是默认的兜底方案，当该方案被启用时，编译器会调用 [`cmd/compile/internal/gc.state.callResult`](https://draveness.me/golang/tree/cmd/compile/internal/gc.state.callResult) 和 [`cmd/compile/internal/gc.state.call`](https://draveness.me/golang/tree/cmd/compile/internal/gc.state.call)，这表示 `defer` 在编译器看来也是函数调用。

##### 栈上分配

在默认情况下，我们可以看到 Go 语言中 [`runtime._defer`](https://draveness.me/golang/tree/runtime._defer) 结构体都会在堆上分配，如果我们能够将部分结构体分配到栈上就可以节约内存分配带来的额外开销。

Go 语言团队在 1.13 中对 `defer` 关键字进行了优化，当该关键字在函数体中最多执行一次时，编译期间的 [`cmd/compile/internal/gc.state.call`](https://draveness.me/golang/tree/cmd/compile/internal/gc.state.call) 会将结构体分配到栈上并调用 [`runtime.deferprocStack`](https://draveness.me/golang/tree/runtime.deferprocStack)。

因为在编译期间我们已经创建了 [`runtime._defer`](https://draveness.me/golang/tree/runtime._defer) 结构体，所以在运行期间 [`runtime.deferprocStack`](https://draveness.me/golang/tree/runtime.deferprocStack) 只需要设置一些未在编译期间初始化的字段，就可以将栈上的 [`runtime._defer`](https://draveness.me/golang/tree/runtime._defer) 追加到函数的链表上。

##### 开放编码

Go 语言在 1.14 中通过开放编码（Open Coded）实现 `defer` 关键字，该设计使用代码内联优化 `defer` 关键的额外开销并引入函数数据 `funcdata` 管理 `panic` 的调用，该优化可以将 `defer` 的调用开销从 1.13 版本的 ~35ns 降低至 ~6ns 左右。

然而开放编码作为一种优化 `defer` 关键字的方法，它不是在所有的场景下都会开启的，开放编码只会在满足以下的条件时启用：

1. 函数的 `defer` 数量少于或者等于 8 个；
2. 函数的 `defer` 关键字不能在循环中执行；
3. 函数的 `return` 语句与 `defer` 语句的乘积小于或者等于 15 个；

### 122. panic 和 recover

- `panic` 能够改变程序的控制流，调用 `panic` 后会立刻停止执行当前函数的剩余代码，并在当前 Goroutine 中递归执行调用方的 `defer`；
- `recover` 可以中止 `panic` 造成的程序崩溃。它是一个只能在 `defer` 中发挥作用的函数，在其他作用域中调用不会发挥作用；

#### 现象

##### 跨协程失效

首先要介绍的现象是 `panic` 只会触发当前 Goroutine 的延迟函数调用，我们可以通过如下所示的代码了解该现象：

```go
func main() {
	defer println("in main")
	go func() {
		defer println("in goroutine")
		panic("")
	}()

	time.Sleep(1 * time.Second)
}

$ go run main.go
in goroutine
panic:
...
```

当我们运行这段代码时会发现 `main` 函数中的 `defer` 语句并没有执行，执行的只有当前 Goroutine 中的 `defer`。

 `defer` 关键字对应的 [`runtime.deferproc`](https://draveness.me/golang/tree/runtime.deferproc) 会将延迟调用函数与调用方所在 Goroutine 进行关联。所以当程序发生崩溃时只会调用当前 Goroutine 的延迟调用函数也是非常合理的。

##### 失效的崩溃恢复

初学 Go 语言的读者可能会写出下面的代码，在主程序中调用 `recover` 试图中止程序的崩溃，但是从运行的结果中我们也能看出，下面的程序没有正常退出。

```go
func main() {
	defer fmt.Println("in main")
	if err := recover(); err != nil {
		fmt.Println(err)
	}

	panic("unknown err")
}

$ go run main.go
in main
panic: unknown err

goroutine 1 [running]:
main.main()
	...
exit status 2
```

仔细分析一下这个过程就能理解这种现象背后的原因，`recover` 只有在发生 `panic` 之后调用才会生效。然而在上面的控制流中，`recover` 是在 `panic` 之前调用的，并不满足生效的条件，所以我们需要在 `defer` 中使用 `recover` 关键字。

##### 嵌套崩溃

Go 语言中的 `panic` 是可以多次嵌套调用的。一些熟悉 Go 语言的读者很可能也不知道这个知识点，如下所示的代码就展示了如何在 `defer` 函数中多次调用 `panic`：

```go
func main() {
	defer fmt.Println("in main")
	defer func() {
		defer func() {
			panic("panic again and again")
		}()
		panic("panic again")
	}()

	panic("panic once")
}

$ go run main.go
in main
panic: panic once
	panic: panic again
	panic: panic again and again

goroutine 1 [running]:
...
exit status 2
```

从上述程序输出的结果，我们可以确定程序多次调用 `panic` 也不会影响 `defer` 函数的正常执行，所以使用 `defer`进行收尾工作一般来说都是安全的。

#### 数据结构

`panic` 关键字在 Go 语言的源代码是由数据结构 [`runtime._panic`](https://draveness.me/golang/tree/runtime._panic) 表示的。每当我们调用 `panic` 都会创建一个如下所示的数据结构存储相关信息：

```go
type _panic struct {
	argp      unsafe.Pointer
	arg       interface{}
	link      *_panic
	recovered bool
	aborted   bool
	pc        uintptr
	sp        unsafe.Pointer
	goexit    bool
}
```

1. `argp` 是指向 `defer` 调用时参数的指针；
2. `arg` 是调用 `panic` 时传入的参数；
3. `link` 指向了更早调用的 [`runtime._panic`](https://draveness.me/golang/tree/runtime._panic) 结构；
4. `recovered` 表示当前 [`runtime._panic`](https://draveness.me/golang/tree/runtime._panic) 是否被 `recover` 恢复；
5. `aborted` 表示当前的 `panic` 是否被强行终止；

从数据结构中的 `link` 字段我们就可以推测出以下的结论：`panic` 函数可以被连续多次调用，它们之间通过 `link` 可以组成链表。

#### 程序崩溃

编译器会将关键字 `panic` 转换成 [`runtime.gopanic`](https://draveness.me/golang/tree/runtime.gopanic)，该函数的执行过程包含以下几个步骤：

1. 创建新的 [`runtime._panic`](https://draveness.me/golang/tree/runtime._panic) 并添加到所在 Goroutine 的 `_panic` 链表的最前面；
2. 在循环中不断从当前 Goroutine 的 `_defer` 中链表获取 [`runtime._defer`](https://draveness.me/golang/tree/runtime._defer) 并调用 [`runtime.reflectcall`](https://draveness.me/golang/tree/runtime.reflectcall) 运行延迟调用函数；
3. 调用 [`runtime.fatalpanic`](https://draveness.me/golang/tree/runtime.fatalpanic) 中止整个程序；

#### 崩溃恢复

```go
func gorecover(argp uintptr) interface{} {
	gp := getg()
	p := gp._panic
	if p != nil && !p.recovered && argp == uintptr(p.argp) {
		p.recovered = true
		return p.arg
	}
	return nil
}
```

该函数的实现很简单，如果当前 Goroutine 没有调用 `panic`，那么该函数会直接返回 `nil`，这也是崩溃恢复在非 `defer` 中调用会失效的原因。在正常情况下，它会修改 [`runtime._panic`](https://draveness.me/golang/tree/runtime._panic) 的 `recovered` 字段。

### 123. make 和 new

- `make` 的作用是初始化内置的数据结构，也就是前面提到的切片、哈希表和 Channel
- `new` 的作用是根据传入的类型分配一片内存空间并返回指向这片内存空间的指针

#### make

在编译期间的类型检查阶段，Go 语言会将代表 `make` 关键字的 `OMAKE` 节点根据参数类型的不同转换成了 `OMAKESLICE`、`OMAKEMAP` 和 `OMAKECHAN` 三种不同类型的节点，这些节点会调用不同的运行时函数来初始化相应的数据结构。

#### new

编译器会在中间代码生成阶段通过以下两个函数处理该关键字：

1. [`cmd/compile/internal/gc.callnew`](https://draveness.me/golang/tree/cmd/compile/internal/gc.callnew) 会将关键字转换成 `ONEWOBJ` 类型的节点
2. [`cmd/compile/internal/gc.state.expr`](https://draveness.me/golang/tree/cmd/compile/internal/gc.state.expr) 会根据申请空间的大小分两种情况处理：
    1. 如果申请的空间为 0，就会返回一个表示空指针的 `zerobase` 变量；
    2. 在遇到其他情况时会将关键字转换成 [`runtime.newobject`](https://draveness.me/golang/tree/runtime.newobject) 函数：

[`runtime.newobject`](https://draveness.me/golang/tree/runtime.newobject) 函数会获取传入类型占用空间的大小，调用 [`runtime.mallocgc`](https://draveness.me/golang/tree/runtime.mallocgc) 在堆上申请一片内存空间并返回指向这片内存空间的指针：

```go
func newobject(typ *_type) unsafe.Pointer {
	return mallocgc(typ.size, typ, true)
}
```

### 124. 上下文 Context

#### 设计原理

在 Goroutine 构成的树形结构中对信号进行同步以减少计算资源的浪费是 [`context.Context`](https://draveness.me/golang/tree/context.Context) 的最大作用。Go 服务的每一个请求都是通过单独的 Goroutine 处理的，HTTP/RPC 请求的处理器会启动新的 Goroutine 访问数据库和其他服务。

我们可能会创建多个 Goroutine 来处理一次请求，而 [`context.Context`](https://draveness.me/golang/tree/context.Context) 的作用是在不同 Goroutine 之间同步请求特定数据、取消信号以及处理请求的截止日期。

每一个 [`context.Context`](https://draveness.me/golang/tree/context.Context) 都会从最顶层的 Goroutine 一层一层传递到最下层。[`context.Context`](https://draveness.me/golang/tree/context.Context) 可以在上层 Goroutine 执行出现错误时，将信号及时同步给下层。

在这段代码中，我们创建了一个过期时间为 1s 的上下文，并向上下文传入 `handle` 函数，该方法会使用 500ms 的时间处理传入的请求：

```go
func main() {
	ctx, cancel := context.WithTimeout(context.Background(), 1*time.Second)
	defer cancel()

	go handle(ctx, 500*time.Millisecond)
	select {
	case <-ctx.Done():
		fmt.Println("main", ctx.Err())
	}
}

func handle(ctx context.Context, duration time.Duration) {
	select {
	case <-ctx.Done():
		fmt.Println("handle", ctx.Err())
	case <-time.After(duration):
		fmt.Println("process request with", duration)
	}
}
```

因为过期时间大于处理时间，所以我们有足够的时间处理该请求，运行上述代码会打印出下面的内容：

```go
$ go run context.go
process request with 500ms
main context deadline exceeded
```

`handle` 函数没有进入超时的 `select` 分支，但是 `main` 函数的 `select` 却会等待 [`context.Context`](https://draveness.me/golang/tree/context.Context) 超时并打印出 `main context deadline exceeded`。

如果我们将处理请求时间增加至 1500ms，整个程序都会因为上下文的过期而被中止。

#### 默认上下文

[`context`](https://github.com/golang/go/tree/master/src/context) 包中最常用的方法还是 [`context.Background`](https://draveness.me/golang/tree/context.Background)、[`context.TODO`](https://draveness.me/golang/tree/context.TODO)，这两个方法都会返回预先初始化好的私有变量 `background` 和 `todo`，它们会在同一个 Go 程序中被复用：

```go
func Background() Context {
	return background
}

func TODO() Context {
	return todo
}
```

从源代码来看，[`context.Background`](https://draveness.me/golang/tree/context.Background) 和 [`context.TODO`](https://draveness.me/golang/tree/context.TODO) 也只是互为别名，没有太大的差别，只是在使用和语义上稍有不同：

- [`context.Background`](https://draveness.me/golang/tree/context.Background) 是上下文的默认值，所有其他的上下文都应该从它衍生出来；
- [`context.TODO`](https://draveness.me/golang/tree/context.TODO) 应该仅在不确定应该使用哪种上下文时使用；

在多数情况下，如果当前函数没有上下文作为入参，我们都会使用 [`context.Background`](https://draveness.me/golang/tree/context.Background) 作为起始的上下文向下传递。

#### 取消信号

[`context.WithCancel`](https://draveness.me/golang/tree/context.WithCancel) 函数能够从 [`context.Context`](https://draveness.me/golang/tree/context.Context) 中衍生出一个新的子上下文并返回用于取消该上下文的函数。一旦我们执行返回的取消函数，当前上下文以及它的子上下文都会被取消，所有的 Goroutine 都会同步收到这一取消信号。

[`context.WithCancel`](https://draveness.me/golang/tree/context.WithCancel) 之外，[`context`](https://github.com/golang/go/tree/master/src/context) 包中的另外两个函数 [`context.WithDeadline`](https://draveness.me/golang/tree/context.WithDeadline) 和 [`context.WithTimeout`](https://draveness.me/golang/tree/context.WithTimeout) 也都能创建可以被取消的计时器上下文 [`context.timerCtx`](https://draveness.me/golang/tree/context.timerCtx)

#### 传值方法

[`context`](https://github.com/golang/go/tree/master/src/context) 包中的 [`context.WithValue`](https://draveness.me/golang/tree/context.WithValue) 能从父上下文中创建一个子上下文，传值的子上下文使用 [`context.valueCtx`](https://draveness.me/golang/tree/context.valueCtx) 类型：

```go
func WithValue(parent Context, key, val interface{}) Context {
	if key == nil {
		panic("nil key")
	}
	if !reflectlite.TypeOf(key).Comparable() {
		panic("key is not comparable")
	}
	return &valueCtx{parent, key, val}
}
```

[`context.valueCtx`](https://draveness.me/golang/tree/context.valueCtx) 结构体会将除了 `Value` 之外的 `Err`、`Deadline` 等方法代理到父上下文中，它只会响应 [`context.valueCtx.Value`](https://draveness.me/golang/tree/context.valueCtx.Value) 方法，该方法的实现也很简单：

```go
type valueCtx struct {
	Context
	key, val interface{}
}

func (c *valueCtx) Value(key interface{}) interface{} {
	if c.key == key {
		return c.val
	}
	return c.Context.Value(key)
}
```

如果 [`context.valueCtx`](https://draveness.me/golang/tree/context.valueCtx) 中存储的键值对与 [`context.valueCtx.Value`](https://draveness.me/golang/tree/context.valueCtx.Value) 方法中传入的参数不匹配，就会从父上下文中查找该键对应的值直到某个父上下文中返回 `nil` 或者查找到对应的值。

### 125. 同步原语与锁

#### 基本原语

基本原语提供了较为基础的同步功能，但是它们是一种相对原始的同步机制，在多数情况下，我们都应该使用抽象层级更高的 Channel 实现同步。
##### Mutex

Go 语言的 [`sync.Mutex`](https://draveness.me/golang/tree/sync.Mutex) 由两个字段 `state` 和 `sema` 组成。其中 `state` 表示当前互斥锁的状态，而 `sema` 是用于控制锁状态的信号量。

```go
type Mutex struct {
	state int32
	sema  uint32
}
```

上述两个加起来只占 8 字节空间的结构体表示了 Go 语言中的互斥锁。

###### 正常模式和饥饿模式

在正常模式下，锁的等待者会按照先进先出的顺序获取锁。但是刚被唤起的 Goroutine 与新创建的 Goroutine 竞争时，大概率会获取不到锁，为了减少这种情况的出现，一旦 Goroutine 超过 1ms 没有获取到锁，它就会将当前互斥锁切换饥饿模式，防止部分 Goroutine 被『饿死』。

在饥饿模式中，互斥锁会直接交给等待队列最前面的 Goroutine。新的 Goroutine 在该状态下不能获取锁、也不会进入自旋状态，它们只会在队列的末尾等待。如果一个 Goroutine 获得了互斥锁并且它在队列的末尾或者它等待的时间少于 1ms，那么当前的互斥锁就会切换回正常模式。

与饥饿模式相比，正常模式下的互斥锁能够提供更好地性能，饥饿模式的能避免 Goroutine 由于陷入等待无法获取锁而造成的高尾延时。

###### 加锁和解锁

互斥锁的加锁是靠 [`sync.Mutex.Lock`](https://draveness.me/golang/tree/sync.Mutex.Lock) 完成的，最新的 Go 语言源代码中已经将 [`sync.Mutex.Lock`](https://draveness.me/golang/tree/sync.Mutex.Lock) 方法进行了简化，方法的主干只保留最常见、简单的情况 — 当锁的状态是 0 时，将 `mutexLocked` 位置成 1：

```go
func (m *Mutex) Lock() {
	if atomic.CompareAndSwapInt32(&m.state, 0, mutexLocked) {
		return
	}
	m.lockSlow()
}
```

如果互斥锁的状态不是 0 时就会调用 [`sync.Mutex.lockSlow`](https://draveness.me/golang/tree/sync.Mutex.lockSlow) 尝试通过自旋（Spinnig）等方式等待锁的释放，该方法的主体是一个非常大 for 循环，这里分成几个过程：

1. 判断当前 Goroutine 能否进入自旋；
	- Goroutine 进入自旋的条件非常苛刻：
		1. 互斥锁只有在普通模式才能进入自旋；
		2. [`runtime.sync_runtime_canSpin`](https://draveness.me/golang/tree/runtime.sync_runtime_canSpin) 需要返回 `true`：
			- 运行在多 CPU 的机器上；
			- 当前 Goroutine 为了获取该锁进入自旋的次数小于四次；
			- 当前机器上至少存在一个正在运行的处理器 P 并且处理的运行队列为空；
1. 通过自旋等待互斥锁的释放；
2. 计算互斥锁的最新状态；
3. 更新互斥锁的状态并获取锁；

互斥锁的解锁过程 [`sync.Mutex.Unlock`](https://draveness.me/golang/tree/sync.Mutex.Unlock) 与加锁过程相比就很简单，该过程会先使用 [`sync/atomic.AddInt32`](https://draveness.me/golang/tree/sync/atomic.AddInt32) 函数快速解锁，这时会发生下面的两种情况：

- 如果该函数返回的新状态等于 0，当前 Goroutine 就成功解锁了互斥锁；
- 如果该函数返回的新状态不等于 0，这段代码会调用 [`sync.Mutex.unlockSlow`](https://draveness.me/golang/tree/sync.Mutex.unlockSlow) 开始慢速解锁：

```go
func (m *Mutex) Unlock() {
	new := atomic.AddInt32(&m.state, -mutexLocked)
	if new != 0 {
		m.unlockSlow(new)
	}
}
```

- 当互斥锁已经被解锁时，调用 [`sync.Mutex.Unlock`](https://draveness.me/golang/tree/sync.Mutex.Unlock) 会直接抛出异常；
- 当互斥锁处于饥饿模式时，将锁的所有权交给队列中的下一个等待者，等待者会负责设置 `mutexLocked` 标志位；
- 当互斥锁处于普通模式时，如果没有 Goroutine 等待锁的释放或者已经有被唤醒的 Goroutine 获得了锁，会直接返回；在其他情况下会通过 [`sync.runtime_Semrelease`](https://draveness.me/golang/tree/sync.runtime_Semrelease) 唤醒对应的 Goroutine；

##### RWMutex

###### 结构体

[`sync.RWMutex`](https://draveness.me/golang/tree/sync.RWMutex) 中总共包含以下 5 个字段：

```go
type RWMutex struct {
	w           Mutex
	writerSem   uint32
	readerSem   uint32
	readerCount int32
	readerWait  int32
}
```

- `w` — 复用互斥锁提供的能力；
- `writerSem` 和 `readerSem` — 分别用于写等待读和读等待写：
- `readerCount` 存储了当前正在执行的读操作数量；
- `readerWait` 表示当写操作被阻塞时等待的读操作个数；

###### 流程

- 调用 [`sync.RWMutex.Lock`](https://draveness.me/golang/tree/sync.RWMutex.Lock) 尝试获取写锁时；
    - 每次 [`sync.RWMutex.RUnlock`](https://draveness.me/golang/tree/sync.RWMutex.RUnlock) 都会将 `readerCount`其减一，当它归零时该 Goroutine 会获得写锁；
    - 将 `readerCount` 减少 `rwmutexMaxReaders` 个数以阻塞后续的读操作；
- 调用 [`sync.RWMutex.Unlock`](https://draveness.me/golang/tree/sync.RWMutex.Unlock) 释放写锁时，会先通知所有的读操作，然后才会释放持有的互斥锁；

读写互斥锁在互斥锁之上提供了额外的更细粒度的控制，能够在读操作远远多于写操作时提升性能。

##### WaitGroup

###### 结构体

[`sync.WaitGroup`](https://draveness.me/golang/tree/sync.WaitGroup) 结构体中只包含两个成员变量：

```go
type WaitGroup struct {
	noCopy noCopy
	state1 [3]uint32
}
```

- `noCopy` — 保证 [`sync.WaitGroup`](https://draveness.me/golang/tree/sync.WaitGroup) 不会被开发者通过再赋值的方式拷贝；
- `state1` — 存储着状态和信号量；

###### 接口

[`sync.WaitGroup`](https://draveness.me/golang/tree/sync.WaitGroup) 对外暴露了三个方法 — [`sync.WaitGroup.Add`](https://draveness.me/golang/tree/sync.WaitGroup.Add)、[`sync.WaitGroup.Wait`](https://draveness.me/golang/tree/sync.WaitGroup.Wait) 和 [`sync.WaitGroup.Done`](https://draveness.me/golang/tree/sync.WaitGroup.Done)。

- [`sync.WaitGroup`](https://draveness.me/golang/tree/sync.WaitGroup) 必须在 [`sync.WaitGroup.Wait`](https://draveness.me/golang/tree/sync.WaitGroup.Wait) 方法返回之后才能被重新使用；
- [`sync.WaitGroup.Done`](https://draveness.me/golang/tree/sync.WaitGroup.Done) 只是对 [`sync.WaitGroup.Add`](https://draveness.me/golang/tree/sync.WaitGroup.Add) 方法的简单封装，我们可以向 [`sync.WaitGroup.Add`](https://draveness.me/golang/tree/sync.WaitGroup.Add) 方法传入任意负数（需要保证计数器非负）快速将计数器归零以唤醒等待的 Goroutine；
- 可以同时有多个 Goroutine 等待当前 [`sync.WaitGroup`](https://draveness.me/golang/tree/sync.WaitGroup) 计数器的归零，这些 Goroutine 会被同时唤醒；

##### Once

Go 语言标准库中 [`sync.Once`](https://draveness.me/golang/tree/sync.Once) 可以保证在 Go 程序运行期间的某段代码只会执行一次。在运行如下所示的代码时，我们会看到如下所示的运行结果：

```go
func main() {
    o := &sync.Once{}
    for i := 0; i < 10; i++ {
        o.Do(func() {
            fmt.Println("only once")
        })
    }
}

$ go run main.go
only once
```

###### 结构体

每一个 [`sync.Once`](https://draveness.me/golang/tree/sync.Once) 结构体中都只包含一个用于标识代码块是否执行过的 `done` 以及一个互斥锁 [`sync.Mutex`](https://draveness.me/golang/tree/sync.Mutex)：

```go
type Once struct {
	done uint32
	m    Mutex
}
```

###### 接口

[`sync.Once.Do`](https://draveness.me/golang/tree/sync.Once.Do) 是 [`sync.Once`](https://draveness.me/golang/tree/sync.Once) 结构体对外唯一暴露的方法，该方法会接收一个入参为空的函数：

- 如果传入的函数已经执行过，会直接返回；
- 如果传入的函数没有执行过，会调用 [`sync.Once.doSlow`](https://draveness.me/golang/tree/sync.Once.doSlow) 执行传入的函数：

```go
func (o *Once) Do(f func()) {
	if atomic.LoadUint32(&o.done) == 0 {
		o.doSlow(f)
	}
}

func (o *Once) doSlow(f func()) {
	o.m.Lock()
	defer o.m.Unlock()
	if o.done == 0 {
		defer atomic.StoreUint32(&o.done, 1)
		f()
	}
}
```

1. 为当前 Goroutine 获取互斥锁；
2. 执行传入的无入参函数；
3. 运行延迟函数调用，将成员变量 `done` 更新成 1；

[`sync.Once`](https://draveness.me/golang/tree/sync.Once) 会通过成员变量 `done` 确保函数不会执行第二次。

Tips:

- [`sync.Once.Do`](https://draveness.me/golang/tree/sync.Once.Do) 方法中传入的函数只会被执行一次，哪怕函数中发生了 `panic`；
- 两次调用 [`sync.Once.Do`](https://draveness.me/golang/tree/sync.Once.Do) 方法传入不同的函数只会执行第一次调传入的函数；

##### Cond

Go 语言标准库中还包含条件变量 [`sync.Cond`](https://draveness.me/golang/tree/sync.Cond)，它可以让一组的 Goroutine 都在满足特定条件时被唤醒。每一个 [`sync.Cond`](https://draveness.me/golang/tree/sync.Cond) 结构体在初始化时都需要传入一个互斥锁，我们可以通过下面的例子了解它的使用方法：

```go
var status int64

func main() {
	c := sync.NewCond(&sync.Mutex{})
	for i := 0; i < 10; i++ {
		go listen(c)
	}
	time.Sleep(1 * time.Second)
	go broadcast(c)

	ch := make(chan os.Signal, 1)
	signal.Notify(ch, os.Interrupt)
	<-ch
}

func broadcast(c *sync.Cond) {
	c.L.Lock()
	atomic.StoreInt64(&status, 1)
	c.Broadcast()
	c.L.Unlock()
}

func listen(c *sync.Cond) {
	c.L.Lock()
	for atomic.LoadInt64(&status) != 1 {
		c.Wait()
	}
	fmt.Println("listen")
	c.L.Unlock()
}

$ go run main.go
listen
...
listen
```

上述代码同时运行了 11 个 Goroutine，这 11 个 Goroutine 分别做了不同事情：

- 10 个 Goroutine 通过 [`sync.Cond.Wait`](https://draveness.me/golang/tree/sync.Cond.Wait) 等待特定条件的满足；
- 1 个 Goroutine 会调用 [`sync.Cond.Broadcast`](https://draveness.me/golang/tree/sync.Cond.Broadcast) 唤醒所有陷入等待的 Goroutine；

调用 [`sync.Cond.Broadcast`](https://draveness.me/golang/tree/sync.Cond.Broadcast) 方法后，上述代码会打印出 10 次 “listen” 并结束调用。

###### 结构体

[`sync.Cond`](https://draveness.me/golang/tree/sync.Cond) 的结构体中包含以下 4 个字段：

```go
type Cond struct {
	noCopy  noCopy
	L       Locker
	notify  notifyList
	checker copyChecker
}
```

- `noCopy` — 用于保证结构体不会在编译期间拷贝；
- `copyChecker` — 用于禁止运行期间发生的拷贝；
- `L` — 用于保护内部的 `notify` 字段，`Locker` 接口类型的变量；
- `notify` — 一个 Goroutine 的链表，它是实现同步机制的核心结构；

```go
type notifyList struct {
	wait uint32
	notify uint32

	lock mutex
	head *sudog
	tail *sudog
}
```

在 [`sync.notifyList`](https://draveness.me/golang/tree/sync.notifyList) 结构体中，`head` 和 `tail` 分别指向的链表的头和尾，`wait` 和 `notify` 分别表示当前正在等待的和已经通知到的 Goroutine 的索引。

###### 接口

[`sync.Cond`](https://draveness.me/golang/tree/sync.Cond) 对外暴露的 [`sync.Cond.Wait`](https://draveness.me/golang/tree/sync.Cond.Wait) 方法会将当前 Goroutine 陷入休眠状态，它的执行过程分成以下两个步骤：

1. 调用 [`runtime.notifyListAdd`](https://draveness.me/golang/tree/runtime.notifyListAdd) 将等待计数器加一并解锁；
2. 调用 [`runtime.notifyListWait`](https://draveness.me/golang/tree/runtime.notifyListWait) 等待其他 Goroutine 的唤醒并加锁：

除了将当前 Goroutine 追加到链表的末端之外，我们还会调用 [`runtime.goparkunlock`](https://draveness.me/golang/tree/runtime.goparkunlock) 将当前 Goroutine 陷入休眠，该函数也是在 Go 语言切换 Goroutine 时经常会使用的方法，它会直接让出当前处理器的使用权并等待调度器的唤醒。

[`sync.Cond.Signal`](https://draveness.me/golang/tree/sync.Cond.Signal) 和 [`sync.Cond.Broadcast`](https://draveness.me/golang/tree/sync.Cond.Broadcast) 就是用来唤醒陷入休眠的 Goroutine 的方法，它们的实现有一些细微的差别：

- [`sync.Cond.Signal`](https://draveness.me/golang/tree/sync.Cond.Signal) 方法会唤醒队列最前面的 Goroutine；
- [`sync.Cond.Broadcast`](https://draveness.me/golang/tree/sync.Cond.Broadcast) 方法会唤醒队列中全部的 Goroutine；

#### ErrGroup

[`golang/sync/errgroup.Group`](https://draveness.me/golang/tree/golang/sync/errgroup.Group) 为我们在一组 Goroutine 中提供了同步、错误传播以及上下文取消的功能，我们可以使用如下所示的方式并行获取网页的数据：

```go
var g errgroup.Group
var urls = []string{
    "http://www.golang.org/",
    "http://www.google.com/",
}
for i := range urls {
    url := urls[i]
    g.Go(func() error {
        resp, err := http.Get(url)
        if err == nil {
            resp.Body.Close()
        }
        return err
    })
}
if err := g.Wait(); err == nil {
    fmt.Println("Successfully fetched all URLs.")
}
```

[`golang/sync/errgroup.Group.Go`](https://draveness.me/golang/tree/golang/sync/errgroup.Group.Go) 方法能够创建一个 Goroutine 并在其中执行传入的函数，而 [`golang/sync/errgroup.Group.Wait`](https://draveness.me/golang/tree/golang/sync/errgroup.Group.Wait) 会等待所有 Goroutine 全部返回，该方法的不同返回结果也有不同的含义：

- 如果返回错误 — 这一组 Goroutine 最少返回一个错误；
- 如果返回空值 — 所有 Goroutine 都成功执行；

##### 结构体

1. `cancel` — 创建 [`context.Context`](https://draveness.me/golang/tree/context.Context) 时返回的取消函数，用于在多个 Goroutine 之间同步取消信号；
2. `wg` — 用于等待一组 Goroutine 完成子任务的同步原语；
3. `errOnce` — 用于保证只接收一个子任务返回的错误；

```go
type Group struct {
	cancel func()

	wg sync.WaitGroup

	errOnce sync.Once
	err     error
}
```

这些字段共同组成了 [`golang/sync/errgroup.Group`](https://draveness.me/golang/tree/golang/sync/errgroup.Group) 结构体并为我们提供同步、错误传播以及上下文取消等功能。

#### Semaphore

信号量是在并发编程中常见的一种同步机制，在需要控制访问资源的进程数量时就会用到信号量，它会保证持有的计数器在 0 到初始化的权重之间波动。

- 每次获取资源时都会将信号量中的计数器减去对应的数值，在释放时重新加回来；
- 当遇到计数器大于信号量大小时，会进入休眠等待其他线程释放信号；

Go 语言的扩展包中就提供了带权重的信号量 [`golang/sync/semaphore.Weighted`](https://draveness.me/golang/tree/golang/sync/semaphore.Weighted)，我们可以按照不同的权重对资源的访问进行管理，这个结构体对外也只暴露了四个方法：

- [`golang/sync/semaphore.NewWeighted`](https://draveness.me/golang/tree/golang/sync/semaphore.NewWeighted) 用于创建新的信号量；
- [`golang/sync/semaphore.Weighted.Acquire`](https://draveness.me/golang/tree/golang/sync/semaphore.Weighted.Acquire) 阻塞地获取指定权重的资源，如果当前没有空闲资源，会陷入休眠等待；
- [`golang/sync/semaphore.Weighted.TryAcquire`](https://draveness.me/golang/tree/golang/sync/semaphore.Weighted.TryAcquire) 非阻塞地获取指定权重的资源，如果当前没有空闲资源，会直接返回 `false`；
- [`golang/sync/semaphore.Weighted.Release`](https://draveness.me/golang/tree/golang/sync/semaphore.Weighted.Release) 用于释放指定权重的资源；

##### SingleFlight

[`golang/sync/singleflight.Group`](https://draveness.me/golang/tree/golang/sync/singleflight.Group) 是 Go 语言扩展包中提供了另一种同步原语，它能够在一个服务中抑制对下游的多次重复请求。一个比较常见的使用场景是：我们在使用 Redis 对数据库中的数据进行缓存，发生缓存击穿时，大量的流量都会打到数据库上进而影响服务的尾延时。

在资源的获取非常昂贵时（例如：访问缓存、数据库），就很适合使用 [`golang/sync/singleflight.Group`](https://draveness.me/golang/tree/golang/sync/singleflight.Group) 优化服务。我们来了解一下它的使用方法：

```go
type service struct {
    requestGroup singleflight.Group
}

func (s *service) handleRequest(ctx context.Context, request Request) (Response, error) {
    v, err, _ := requestGroup.Do(request.Hash(), func() (interface{}, error) {
        rows, err := // select * from tables
        if err != nil {
            return nil, err
        }
        return rows, nil
    })
    if err != nil {
        return nil, err
    }
    return Response{
        rows: rows,
    }, nil
}
```

因为请求的哈希在业务上一般表示相同的请求，所以上述代码使用它作为请求的键。当然，我们也可以选择其他的字段作为 [`golang/sync/singleflight.Group.Do`](https://draveness.me/golang/tree/golang/sync/singleflight.Group.Do) 方法的第一个参数减少重复的请求。

##### 结构体

[`golang/sync/singleflight.Group`](https://draveness.me/golang/tree/golang/sync/singleflight.Group) 结构体由一个互斥锁 [`sync.Mutex`](https://draveness.me/golang/tree/sync.Mutex) 和一个映射表组成，每一个 [`golang/sync/singleflight.call`](https://draveness.me/golang/tree/golang/sync/singleflight.call) 结构体都保存了当前调用对应的信息：

```go
type Group struct {
	mu sync.Mutex
	m  map[string]*call
}

type call struct {
	wg sync.WaitGroup

	val interface{}
	err error

	dups  int
	chans []chan<- Result
}
```

[`golang/sync/singleflight.call`](https://draveness.me/golang/tree/golang/sync/singleflight.call) 结构体中的 `val` 和 `err` 字段都只会在执行传入的函数时赋值一次并在 [`sync.WaitGroup.Wait`](https://draveness.me/golang/tree/sync.WaitGroup.Wait) 返回时被读取；`dups` 和 `chans` 两个字段分别存储了抑制的请求数量以及用于同步结果的 Channel。

##### 接口

- [`golang/sync/singleflight.Group.Do`](https://draveness.me/golang/tree/golang/sync/singleflight.Group.Do) — 同步等待的方法；
- [`golang/sync/singleflight.Group.DoChan`](https://draveness.me/golang/tree/golang/sync/singleflight.Group.DoChan) — 返回 Channel 异步等待的方法；

这两个方法在功能上没有太多的区别，只是在接口的表现上稍有不同。

### 126. 计时器

#### 数据结构

[`runtime.timer`](https://draveness.me/golang/tree/runtime.timer) 是 Go 语言计时器的内部表示，每一个计时器都存储在对应处理器的最小四叉堆中，下面是运行时计时器对应的结构体：

```go
type timer struct {
	pp puintptr

	when     int64
	period   int64
	f        func(interface{}, uintptr)
	arg      interface{}
	seq      uintptr
	nextwhen int64
	status   uint32
}
```

- `when` — 当前计时器被唤醒的时间；
- `period` — 两次被唤醒的间隔；
- `f` — 每当计时器被唤醒时都会调用的函数；
- `arg` — 计时器被唤醒时调用 `f` 传入的参数；
- `nextWhen` — 计时器处于 `timerModifiedXX` 状态时，用于设置 `when`字段；
- `status` — 计时器的状态；

然而这里的 [`runtime.timer`](https://draveness.me/golang/tree/runtime.timer) 只是计时器运行时的私有结构体，对外暴露的计时器使用 [`time.Timer`](https://draveness.me/golang/tree/time.Timer) 结体：

```go
type Timer struct {
	C <-chan Time
	r runtimeTimer
}
```

[`time.Timer`](https://draveness.me/golang/tree/time.Timer) 计时器必须通过 [`time.NewTimer`](https://draveness.me/golang/tree/time.NewTimer)、[`time.AfterFunc`](https://draveness.me/golang/tree/time.AfterFunc) 或者 [`time.After`](https://draveness.me/golang/tree/time.After) 函数创建。 当计时器失效时，订阅计时器 Channel 的 Goroutine 会收到计时器失效的时间。

#### 状态机

运行时使用状态机的方式处理全部的计时器，其中包括 10 种状态和几种操作。由于 Go 语言的计时器需要同时支持增加、删除、修改和重置等操作，所以它的状态非常复杂，目前会包含以下 10 种可能：

|状态|解释|
|---|---|
|timerNoStatus|还没有设置状态|
|timerWaiting|等待触发|
|timerRunning|运行计时器函数|
|timerDeleted|被删除|
|timerRemoving|正在被删除|
|timerRemoved|已经被停止并从堆中删除|
|timerModifying|正在被修改|
|timerModifiedEarlier|被修改到了更早的时间|
|timerModifiedLater|被修改到了更晚的时间|
|timerMoving|已经被修改正在被移动|


上述表格已经展示了不同状态的含义，但是我们还需要展示一些重要的信息，例如状态的存在时间、计时器是否在堆上等：

- `timerRunning`、`timerRemoving`、`timerModifying` 和 `timerMoving` — 停留的时间都比较短；
- `timerWaiting`、`timerRunning`、`timerDeleted`、`timerRemoving`、`timerModifying`、`timerModifiedEarlier`、`timerModifiedLater` 和 `timerMoving` — 计时器在处理器的堆上；
- `timerNoStatus` 和 `timerRemoved` — 计时器不在堆上；
- `timerModifiedEarlier` 和 `timerModifiedLater` — 计时器虽然在堆上，但是可能位于错误的位置上，需要重新排序；

当我们操作计时器时，运行时会根据状态的不同而做出反应，所以在分析计时器时会将状态作为切入点分析其实现原理。计时器的状态机中包含如下所示的 7 种不同操作，它们分别承担了不同的职责：

- [`runtime.addtimer`](https://draveness.me/golang/tree/runtime.addtimer) — 向当前处理器增加新的计时器
- [`runtime.deltimer`](https://draveness.me/golang/tree/runtime.deltimer) — 将计时器标记成 `timerDeleted` 删除处理器中的计时器
- [`runtime.modtimer`](https://draveness.me/golang/tree/runtime.modtimer) — 网络轮询器会调用该函数修改计时器
- [`runtime.cleantimers`](https://draveness.me/golang/tree/runtime.cleantimers) — 清除队列头中的计时器，能够提升程序创建和删除计时器的性能
- [`runtime.adjusttimers`](https://draveness.me/golang/tree/runtime.adjusttimers) — 调整处理器持有的计时器堆，包括移动会稍后触发的计时器、删除标记为 `timerDeleted` 的计时器
- [`runtime.runtimer`](https://draveness.me/golang/tree/runtime.runtimer) — 检查队列头中的计时器，在其准备就绪时运行该计时器[

标准库中的计时器在大多数情况下是能够正常工作并且高效完成任务的，但是在遇到极端情况或者性能敏感场景时，它可能没有办法胜任，而在 10ms 的这个粒度中，作者在社区中也没有找到能够使用的计时器实现，一些使用时间轮算法的开源库也不能很好地完成这个任务。

### 127. Channel

#### 设计原理

##### 先进先出

- 先从 Channel 读取数据的 Goroutine 会先接收到数据；
- 先向 Channel 发送数据的 Goroutine 会得到先发送数据的权利；

##### 无锁管道

乐观并发控制本质上是基于验证的协议，我们使用原子指令 CAS（compare-and-swap 或者 compare-and-set）在多线程中同步数据，无锁队列的实现也依赖这一原子指令。

Channel 在运行时的内部表示是 [`runtime.hchan`](https://draveness.me/golang/tree/runtime.hchan)，该结构体中包含了用于保护成员变量的互斥锁，从某种程度上说，Channel 是一个用于同步和通信的有锁队列，使用互斥锁解决程序中可能存在的线程竞争问题是很常见的，我们能很容易地实现有锁队列。

然而锁导致的休眠和唤醒会带来额外的上下文切换，如果临界区[6](https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-channel/#fn:6)过大，加锁解锁导致的额外开销就会成为性能瓶颈。

因为目前通过 CAS 实现的无锁 Channel 没有提供先进先出的特性，所以该提案暂时也被搁浅了。

#### 数据结构

Go 语言的 Channel 在运行时使用 [`runtime.hchan`](https://draveness.me/golang/tree/runtime.hchan) 结构体表示。我们在 Go 语言中创建新的 Channel 时，实际上创建的都是如下所示的结构：

```go
type hchan struct {
	qcount   uint
	dataqsiz uint
	buf      unsafe.Pointer
	elemsize uint16
	closed   uint32
	elemtype *_type
	sendx    uint
	recvx    uint
	recvq    waitq
	sendq    waitq

	lock mutex
}
```

[`runtime.hchan`](https://draveness.me/golang/tree/runtime.hchan) 结构体中的五个字段 `qcount`、`dataqsiz`、`buf`、`sendx`、`recv` 构建底层的循环队列：

- `qcount` — Channel 中的元素个数；
- `dataqsiz` — Channel 中的循环队列的长度；
- `buf` — Channel 的缓冲区数据指针；
- `sendx` — Channel 的发送操作处理到的位置；
- `recvx` — Channel 的接收操作处理到的位置；

除此之外，`elemsize` 和 `elemtype` 分别表示当前 Channel 能够收发的元素类型和大小；`sendq` 和 `recvq` 存储了当前 Channel 由于缓冲区空间不足而阻塞的 Goroutine 列表，这些等待队列使用双向链表 [`runtime.waitq`](https://draveness.me/golang/tree/runtime.waitq)表示，链表中所有的元素都是 [`runtime.sudog`](https://draveness.me/golang/tree/runtime.sudog) 结构：

```go
type waitq struct {
	first *sudog
	last  *sudog
}
```

[`runtime.sudog`](https://draveness.me/golang/tree/runtime.sudog) 表示一个在等待列表中的 Goroutine，该结构中存储了两个分别指向前后 [`runtime.sudog`](https://draveness.me/golang/tree/runtime.sudog) 的指针以构成链表。

#### 发送数据

1. 如果当前 Channel 的 `recvq` 上存在已经被阻塞的 Goroutine，那么会直接将数据发送给当前 Goroutine 并将其设置成下一个运行的 Goroutine；
2. 如果 Channel 存在缓冲区并且其中还有空闲的容量，我们会直接将数据存储到缓冲区 `sendx` 所在的位置上；
3. 如果不满足上面的两种情况，会创建一个 [`runtime.sudog`](https://draveness.me/golang/tree/runtime.sudog) 结构并将其加入 Channel 的 `sendq` 队列中，当前 Goroutine 也会陷入阻塞等待其他的协程从 Channel 接收数据；

发送数据的过程中包含几个会触发 Goroutine 调度的时机：

1. 发送数据时发现 Channel 上存在等待接收数据的 Goroutine，立刻设置处理器的 `runnext` 属性，但是并不会立刻触发调度；
2. 发送数据时并没有找到接收方并且缓冲区已经满了，这时会将自己加入 Channel 的 `sendq` 队列并调用 [`runtime.goparkunlock`](https://draveness.me/golang/tree/runtime.goparkunlock) 触发 Goroutine 的调度让出处理器的使用权；

#### 接收数据

1. 如果 Channel 为空，那么会直接调用 [`runtime.gopark`](https://draveness.me/golang/tree/runtime.gopark) 挂起当前 Goroutine；
2. 如果 Channel 已经关闭并且缓冲区没有任何数据，[`runtime.chanrecv`](https://draveness.me/golang/tree/runtime.chanrecv) 会直接返回；
3. 如果 Channel 的 `sendq` 队列中存在挂起的 Goroutine，会将 `recvx`索引所在的数据拷贝到接收变量所在的内存空间上并将 `sendq` 队列中 Goroutine 的数据拷贝到缓冲区；
4. 如果 Channel 的缓冲区中包含数据，那么直接读取 `recvx` 索引对应的数据；
5. 在默认情况下会挂起当前的 Goroutine，将 [`runtime.sudog`](https://draveness.me/golang/tree/runtime.sudog) 结构加入 `recvq` 队列并陷入休眠等待调度器的唤醒；

总结一下从 Channel 接收数据时，会触发 Goroutine 调度的两个时机：

1. 当 Channel 为空时；
2. 当缓冲区中不存在数据并且也不存在数据的发送者时；

### 128. 调度器

#### 多线程调度器

多线程调度器的主要问题是调度时的锁竞争会严重浪费资源，[Scalable Go Scheduler Design Doc](http://golang.org/s/go11sched) 中对调度器做的性能测试发现 14% 的时间都花费在 [`runtime.futex:go1.0.1`](https://draveness.me/golang/tree/runtime.futex:go1.0.1) 上，该调度器有以下问题需要解决：

1. 调度器和锁是全局资源，所有的调度状态都是中心化存储的，锁竞争问题严重；
2. 线程需要经常互相传递可运行的 Goroutine，引入了大量的延迟；
3. 每个线程都需要处理内存缓存，导致大量的内存占用并影响数据局部性；
4. 系统调用频繁阻塞和解除阻塞正在运行的线程，增加了额外开销；

#### 任务窃取调度器

1. 在当前的 G-M 模型中引入了处理器 P，增加中间层；
2. 在处理器 P 的基础上实现基于工作窃取的调度器；

当前处理器本地的运行队列中不包含 Goroutine 时，调用 [`runtime.findrunnable:779c45a`](https://draveness.me/golang/tree/runtime.findrunnable:779c45a) 会触发工作窃取，从其它的处理器的队列中随机获取一些 Goroutine。

基于工作窃取的多线程调度器将每一个线程绑定到了独立的 CPU 上，这些线程会被不同处理器管理，不同的处理器通过工作窃取对任务进行再分配实现任务的平衡，也能提升调度器和 Go 语言程序的整体性能，今天所有的 Go 语言服务都受益于这一改动。

#### 抢占式调度器

对 Go 语言并发模型的修改提升了调度器的性能，但是 1.1 版本中的调度器仍然不支持抢占式调度，程序只能依靠 Goroutine 主动让出 CPU 资源才能触发调度。Go 语言的调度器在 1.2 版本中引入基于协作的抢占式调度解决下面的问题

- 某些 Goroutine 可以长时间占用线程，造成其它 Goroutine 的饥饿；
- 垃圾回收需要暂停整个程序（Stop-the-world，STW），最长可能需要几分钟的时间，导致整个程序无法工作；

1.2 版本的抢占式调度虽然能够缓解这个问题，但是它实现的抢占式调度是基于协作的，在之后很长的一段时间里 Go 语言的调度器都有一些无法被抢占的边缘情况，例如：for 循环或者垃圾回收长时间占用线程，这些问题中的一部分直到 1.14 才被基于信号的抢占式调度解决。

#### 数据结构

1. G — 表示 Goroutine，它是一个待执行的任务；
2. M — 表示操作系统的线程，它由操作系统的调度器调度和管理；
3. P — 表示处理器，它可以被看做运行在线程上的本地调度器；

##### G

Goroutine 是 Go 语言调度器中待执行的任务，它在运行时调度器中的地位与线程在操作系统中差不多，但是它占用了更小的内存空间，也降低了上下文切换的开销。

Goroutine 只存在于 Go 语言的运行时，它是 Go 语言在用户态提供的线程，作为一种粒度更细的资源调度单元，如果使用得当能够在高并发的场景下更高效地利用机器的 CPU。

Goroutine 在 Go 语言运行时使用私有结构体 [`runtime.g`](https://draveness.me/golang/tree/runtime.g) 表示。这个私有结构体非常复杂，总共包含 40 多个用于表示各种状态的成员变量，这里也不会介绍所有的字段，仅会挑选其中的一部分。

```go
type g struct {
	preempt       bool // 抢占信号
	preemptStop   bool // 抢占时将状态修改成 `_Gpreempted`
	preemptShrink bool // 在同步安全点收缩栈
}
```

Goroutine 与我们在前面章节提到的 `defer` 和 `panic` 也有千丝万缕的联系，每一个 Goroutine 上都持有两个分别存储 `defer` 和 `panic` 对应结构体的链表：

```go
type g struct {
	_panic       *_panic // 最内侧的 panic 结构体
	_defer       *_defer // 最内侧的延迟函数结构体
}
```

最后，我们再节选一些比较有趣或者重要的字段：

```go
type g struct {
	m              *m
	sched          gobuf
	atomicstatus   uint32
	goid           int64
}
```

- `m` — 当前 Goroutine 占用的线程，可能为空；
- `atomicstatus` — Goroutine 的状态；
- `sched` — 存储 Goroutine 的调度相关的数据；
- `goid` — Goroutine 的 ID，该字段对开发者不可见，Go 团队认为引入 ID 会让部分 Goroutine 变得更特殊，从而限制语言的并发能力；

##### M

Go 语言并发模型中的 M 是操作系统线程。调度器最多可以创建 10000 个线程，但是其中大多数的线程都不会执行用户代码（可能陷入系统调用），最多只会有 `GOMAXPROCS` 个活跃线程能够正常运行。

在默认情况下，运行时会将 `GOMAXPROCS` 设置成当前机器的核数，我们也可以在程序中使用 [`runtime.GOMAXPROCS`](https://draveness.me/golang/tree/runtime.GOMAXPROCS) 来改变最大的活跃线程数。一个四核机器会创建四个活跃的操作系统线程，每一个线程都对应一个运行时中的 [`runtime.m`](https://draveness.me/golang/tree/runtime.m) 结构体。

在大多数情况下，我们都会使用 Go 的默认设置，也就是线程数等于 CPU 数，默认的设置不会频繁触发操作系统的线程调度和上下文切换，所有的调度都会发生在用户态，由 Go 语言调度器触发，能够减少很多额外开销。

Go 语言会使用私有结构体 [`runtime.m`](https://draveness.me/golang/tree/runtime.m) 表示操作系统线程

```go
type m struct {
	g0   *g
	curg *g
	...
}
```

其中 g0 是持有调度栈的 Goroutine，`curg` 是在当前线程上运行的用户 Goroutine，这也是操作系统线程唯一关心的两个 Goroutine。

##### P

调度器中的处理器 P 是线程和 Goroutine 的中间层，它能提供线程需要的上下文环境，也会负责调度线程上的等待队列，通过处理器 P 的调度，每一个内核线程都能够执行多个 Goroutine，它能在 Goroutine 进行一些 I/O 操作时及时让出计算资源，提高线程的利用率。

因为调度器在启动时就会创建 `GOMAXPROCS` 个处理器，所以 Go 语言程序的处理器数量一定会等于 `GOMAXPROCS`，这些处理器会绑定到不同的内核线程上。

[`runtime.p`](https://draveness.me/golang/tree/runtime.p) 是处理器的运行时表示，作为调度器的内部实现，它包含的字段也非常多，其中包括与性能追踪、垃圾回收和计时器相关的字段，我们主要关注处理器中的线程和运行队列：

```go
type p struct {
	m           muintptr

	runqhead uint32
	runqtail uint32
	runq     [256]guintptr
	runnext guintptr
	...
}
```

反向存储的线程维护着线程与处理器之间的关系，而 `runqhead`、`runqtail` 和 `runq` 三个字段表示处理器持有的运行队列，其中存储着待执行的 Goroutine 列表，`runnext` 中是线程下一个需要执行的 Goroutine。

#### 运行队列

Go 语言有两个运行队列，其中一个是处理器本地的运行队列，另一个是调度器持有的全局运行队列，只有在本地运行队列没有剩余空间时才会使用全局队列。

### 129. 网络轮询器

#### 设计原理

##### 堵塞I/O

阻塞 I/O 是最常见的 I/O 模型，在默认情况下，当我们通过 `read` 或者 `write` 等系统调用读写文件或者网络时，应用程序会被阻塞：

```c
ssize_t read(int fd, void *buf, size_t count);
ssize_t write(int fd, const void *buf, size_t nbytes);
```

##### 非阻塞I/O

第一次从文件描述符中读取数据会触发系统调用并返回 `EAGAIN` 错误，`EAGAIN` 意味着该文件描述符还在等待缓冲区中的数据；随后，应用程序会不断轮询调用 `read` 直到它的返回值大于 0，这时应用程序就可以对读取操作系统缓冲区中的数据并进行操作。进程使用非阻塞的 I/O 操作时，可以在等待过程中执行其他任务，提高 CPU 的利用率。

##### I/O多路复用

I/O 多路复用被用来处理同一个事件循环中的多个 I/O 事件。I/O 多路复用需要使用特定的系统调用，最常见的系统调用是 [`select`](https://github.com/torvalds/linux/blob/f757165705e92db62f85a1ad287e9251d1f2cd82/fs/select.c#L722)，该函数可以同时监听最多 1024 个文件描述符的可读或者可写状态：

```c
int select(int nfds, fd_set *restrict readfds, fd_set *restrict writefds, fd_set *restrict errorfds, struct timeval *restrict timeout);
```

除了标准的 [`select`](https://github.com/torvalds/linux/blob/f757165705e92db62f85a1ad287e9251d1f2cd82/fs/select.c#L722) 之外，操作系统中还提供了一个比较相似的 `poll` 函数，它使用链表存储文件描述符，摆脱了 1024 的数量上限。

多路复用函数会阻塞的监听一组文件描述符，当文件描述符的状态转变为可读或者可写时，`select` 会返回可读或者可写事件的个数，应用程序可以在输入的文件描述符中查找哪些可读或者可写，然后执行相应的操作。

##### 多模块

为了提高 I/O 多路复用的性能，不同的操作系统也都实现了自己的 I/O 多路复用函数，例如：`epoll`、`kqueue` 和 `evport` 等。Go 语言为了提高在不同操作系统上的 I/O 操作性能，使用平台特定的函数实现了多个版本的网络轮询模块：

- [`src/runtime/netpoll_epoll.go`](https://github.com/golang/go/blob/master/src/runtime/netpoll_epoll.go)
- [`src/runtime/netpoll_kqueue.go`](https://github.com/golang/go/blob/master/src/runtime/netpoll_kqueue.go)
- [`src/runtime/netpoll_solaris.go`](https://github.com/golang/go/blob/master/src/runtime/netpoll_solaris.go)
- [`src/runtime/netpoll_windows.go`](https://github.com/golang/go/blob/master/src/runtime/netpoll_windows.go)
- [`src/runtime/netpoll_aix.go`](https://github.com/golang/go/blob/master/src/runtime/netpoll_aix.go)
- [`src/runtime/netpoll_fake.go`](https://github.com/golang/go/blob/master/src/runtime/netpoll_fake.go)

这些模块在不同平台上实现了相同的功能，构成了一个常见的树形结构。编译器在编译 Go 语言程序时，会根据目标平台选择树中特定的分支进行编译。

#### 数据结构

操作系统中 I/O 多路复用函数会监控文件描述符的可读或者可写，而 Go 语言网络轮询器会监听 [`runtime.pollDesc`](https://draveness.me/golang/tree/runtime.pollDesc) 结构体的状态，它会封装操作系统的文件描述符：

```go
type pollDesc struct {
	link *pollDesc

	lock    mutex
	fd      uintptr
	...
	rseq    uintptr
	rg      uintptr
	rt      timer
	rd      int64
	wseq    uintptr
	wg      uintptr
	wt      timer
	wd      int64
}
```

该结构体中包含用于监控可读和可写状态的变量，我们按照功能将它们分成以下四组：

- `rseq` 和 `wseq` — 表示文件描述符被重用或者计时器被重置；
- `rg` 和 `wg` — 表示二进制的信号量，可能为 `pdReady`、`pdWait`、等待文件描述符可读或者可写的 Goroutine 以及 `nil`；
- `rd` 和 `wd` — 等待文件描述符可读或者可写的截止日期；
- `rt` 和 `wt` — 用于等待文件描述符的计时器；
