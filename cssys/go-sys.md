## Go 语言底层设计

### 数组与编译

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

### 切片与编译

从切片的定义我们能推测出，切片在编译期间的生成的类型只会包含切片中的元素类型，即 `int` 或者 `interface{}` 等。

使用 `append` 关键字向切片中追加元素也是常见的切片操作，中间代码生成阶段的 [`cmd/compile/internal/gc.state.append`](https://draveness.me/golang/tree/cmd/compile/internal/gc.state.append) 方法会根据返回值是否会覆盖原变量，选择进入两种流程，最大的区别在于得到的新切片是否会赋值回原变量。如果我们选择覆盖原有的变量，就不需要担心切片发生拷贝影响性能，因为 Go 语言编译器已经对这种常见的情况做出了优化。

相比于依次拷贝元素，[`runtime.memmove`](https://draveness.me/golang/tree/runtime.memmove) 能够提供更好的性能。需要注意的是，整块拷贝内存仍然会占用非常多的资源，在大切片上执行拷贝操作时一定要注意对性能的影响。

### 理解 Go 中哈希表的原理

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

### 字符串原理设计

只读只意味着字符串会分配到只读的内存空间，但是 Go 语言只是不支持直接修改 `string` 类型变量的内存空间，我们仍然可以通过在 `string` 和 `[]byte` 类型之间反复转换实现修改这一目的：

1. 先将这段内存拷贝到堆或者栈上；
2. 将变量的类型转换成 `[]byte` 后并修改字节数据；
3. 将修改后的字节数组转换回 `string`；

与切片的结构体相比，字符串只少了一个表示容量的 `Cap` 字段，而正是因为切片在 Go 语言的运行时表示与字符串高度相似，所以我们经常会说字符串是一个只读的切片类型。

### 函数调用

Go 通过栈传递函数的参数和返回值，在调用函数之前会在栈上为返回值分配合适的内存空间，随后将入参从右到左按顺序压栈并拷贝参数，返回值会被存储到调用方预留好的栈空间上，我们可以简单总结出以下几条规则：

1. 通过堆栈传递参数，入栈的顺序是从右到左，而参数的计算是从左到右；
2. 函数返回值通过堆栈传递并由调用者预先分配内存空间；
3. 调用函数时都是传值，接收方会对入参进行复制再计算；
#### 参数传递

不同语言会选择不同的方式传递参数，Go 语言选择了传值的方式，**无论是传递基本类型、结构体还是指针，都会对传递的参数进行拷贝**。

### 接口 itab 结构体设计

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

### 反射法则

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

### for 与 range

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

### select 设计

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

### defer

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

### panic 和 recover

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

### make 和 new

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

### 上下文 Context

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

### 同步原语与锁

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

### 计时器

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

### Channel

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

### 调度器

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

### 网络轮询器

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

### 系统监控

#### 设计原理

守护进程是很有效的设计，它在整个系统的生命周期中都会存在，会随着系统的启动而启动，系统的结束而结束。在操作系统和 Kubernetes 中，我们经常会将数据库服务、日志服务以及监控服务等进程作为守护进程运行。

Go 语言的系统监控也起到了很重要的作用，它在内部启动了一个不会中止的循环，在循环的内部会轮询网络、抢占长期运行或者处于系统调用的 Goroutine 以及触发垃圾回收，通过这些行为，它能够让系统的运行状态变得更健康。

#### 监控循环

系统监控在每次循环开始时都会通过 `usleep` 挂起当前线程，该函数的参数是微秒，运行时会遵循以下的规则决定休眠时间：

- 初始的休眠时间是 20μs；
- 最长的休眠时间是 10ms；
- 当系统监控在 50 个循环中都没有唤醒 Goroutine 时，休眠时间在每个循环都会倍增；

当程序趋于稳定之后，系统监控的触发时间就会稳定在 10ms。它除了会检查死锁之外，还会在循环中完成以下的工作：

- 运行计时器 — 获取下一个需要被触发的计时器；
- 轮询网络 — 获取需要处理的到期文件描述符；
- 抢占处理器 — 抢占运行时间较长的或者处于系统调用的 Goroutine；
- 垃圾回收 — 在满足条件时触发垃圾收集回收内存；

### 内存分配器

内存管理一般包含三个不同的组件，分别是用户程序（Mutator）、分配器（Allocator）和收集器（Collector），当用户程序申请内存时，它会通过内存分配器申请新内存，而分配器会负责从堆中初始化相应的内存区域。

内存分配是 Go 语言运行时内存管理的核心逻辑，运行时的内存分配器使用类似 TCMalloc 的分配策略将对象根据大小分类，并设计多层级的组件提高内存分配器的性能。

