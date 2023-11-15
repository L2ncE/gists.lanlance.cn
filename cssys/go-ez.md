## Go 高手技法

### 多分支映射的声明

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

### 刀法灵活的切片

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

### unsafe.Pointer 最佳实践

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

### 格式化优化技巧

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

### 编译后执行效率优化

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

### 相辅相成的协程

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

### 精打细算的锁

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

### 追求极致的 No GC

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