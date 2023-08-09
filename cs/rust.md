# Rust

## 基础

### 1. 为何要手动设置变量的可变性？

- 灵活性
- 安全性
- 除了以上两个优点，还有一个很大的优点，那就是运行性能上的提升，因为将本身无需改变的变量声明为不可变在运行期会避免一些多余的 `runtime` 检查。

### 2. 变量绑定

为何不用赋值而用绑定呢（其实你也可以称之为赋值，但是绑定的含义更清晰准确）？这里就涉及 Rust 最核心的原则——**所有权**，简单来讲，任何内存对象都是有主人的，而且一般情况下完全属于它的主人，绑定就是把这个对象绑定给一个变量，让这个变量成为它的主人。

### 3. 变量可变性

Rust 的变量在默认情况下是**不可变的**。这让我们编写的代码更安全，性能也更好。当然你可以通过 `mut` 关键字让变量变为**可变的**，让设计更灵活。

### 4. 使用下划线开头忽略未使用的变量

有时创建一个不会被使用的变量是有用的，比如你正在设计原型或刚刚开始一个项目。这时**你希望告诉 Rust 不要警告未使用的变量，为此可以用下划线作为变量名的开头**。

### 5. 常量与变量的差异

- 常量不允许使用 `mut`。**常量不仅仅默认不可变，而且自始至终不可变**，因为常量在编译完成后，已经确定它的值。
- 常量使用 `const` 关键字而不是 `let` 关键字来声明，并且值的类型**必须**标注。

```rust
const MAX_POINTS: u32 = 100_000;
```

### 6. 变量遮蔽

```rust
fn main() {
    let x = 5;
    // 在main函数的作用域内对之前的x进行遮蔽
    let x = x + 1;

    {
        // 在当前的花括号作用域内，对之前的x进行遮蔽
        let x = x * 2;
        println!("The value of x in the inner scope is: {}", x);
    }

    println!("The value of x is: {}", x);
}

// The value of x in the inner scope is: 12 
// The value of x is: 6
```

### 7. 整形溢出

- 使用 `wrapping_*` 方法在所有模式下都按照补码循环溢出规则处理，例如 `wrapping_add`
- 如果使用 `checked_*` 方法时发生溢出，则返回 `None` 值
- 使用 `overflowing_*` 方法返回该值和一个指示是否存在溢出的布尔值
- 使用 `saturating_*` 方法使值达到最小值或最大值

```rust
fn main() {
    let a : u8 = 255;
    let b = a.wrapping_add(20);
    println!("{}", b);  // 19
}
```

### 8. 浮点数陷阱

- 避免在浮点数上测试相等性
- 当结果在数学上可能存在未定义时，需要格外的小心

```rust
fn main() {
  // 断言0.1 + 0.2与0.3相等
  assert!(0.1 + 0.2 == 0.3);
}

// panic
```

因为二进制精度问题，导致了 0.1 + 0.2 并不严格等于 0.3，它们可能在小数点 N 位后存在误差。

### 9. NaN

对于数学上未定义的结果，例如对负数取平方根 `-42.1.sqrt()` ，会产生一个特殊的结果：Rust 的浮点数类型使用 `NaN` (not a number)来处理这些情况。**所有跟 `NaN` 交互的操作，都会返回一个 `NaN`**，而且 `NaN` 不能用来比较。

>出于防御性编程的考虑，可以使用 `is_nan()` 等方法，可以用来判断一个数值是否是 `NaN` 。

### 10. 语句与表达式

语句会执行一些操作但是不会返回一个值，而表达式会在求值后返回一个值，因此在上述函数体的三行代码中，前两行是语句，最后一行是表达式。

```rust
fn add_with_extra(x: i32, y: i32) -> i32 {
    let x = x + 1; // 语句
    let y = y + 5; // 语句
    x + y // 表达式
}
```

**表达式不能包含分号**。这一点非常重要，一旦你在表达式后加上分号，它就会变成一条语句，再也**不会**返回一个值，请牢记！

### 11. 发散函数

当用 `!` 作函数返回类型的时候，表示该函数永不返回( diverge function )，特别的，这种语法往往用做会导致程序崩溃的函数：

```rust
fn forever() -> ! {
  loop {
    //...
  };
}
```

### 12. 所有权原则

1. Rust 中每一个值都被一个变量所拥有，该变量被称为值的所有者
2. 一个值同时只能被一个变量所拥有，或者说一个值只能拥有一个所有者
3. 当所有者 (变量)离开作用域范围时，这个值将被丢弃 (drop)

```rust
let s1 = String::from("hello");
let s2 = s1;

println!("{}, world!", s1);

// error[E0382]: use of moved value: `s1`
```

### 13. 深拷贝（克隆）

如果我们**确实**需要深度复制 `String` 中堆上的数据，而不仅仅是栈上的数据，可以使用一个叫做 `clone` 的方法。

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2);
```

### 14. Copy 特征

**任何基本类型的组合可以 `Copy` ，不需要分配内存或某种形式资源的类型是可以 `Copy` 的**。如下是一些 `Copy` 的类型：

- 所有整数类型，比如 `u32`
- 布尔类型，`bool`，它的值是 `true` 和 `false`
- 所有浮点数类型，比如 `f64`
- 字符类型，`char`
- 元组，当且仅当其包含的类型也都是 `Copy` 的时候。比如，`(i32, i32)` 是 `Copy` 的，但 `(i32, String)` 就不是
- 不可变引用 `&T` ，例如转移所有权中的最后一个例子，**但是注意: 可变引用 `&mut T` 是不可以 Copy 的**

### 15. 可变引用与不可变引用

```rust
fn main() {
    let s = String::from("hello");

    change(&s);
}

fn change(some_string: &String) {
    some_string.push_str(", world");
}

// error[E0596]: cannot borrow `*some_string` as mutable, as it is behind a `&` reference
```

声明 `s` 是可变类型，其次创建一个可变的引用 `&mut s` 和接受可变引用参数 `some_string: &mut String` 的函数。

```rust
fn main() {
    let mut s = String::from("hello");

    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

***可变引用同时只能存在一个***

```rust
let mut s = String::from("hello");

let r1 = &mut s;
let r2 = &mut s;

println!("{}, {}", r1, r2);

// error[E0499]: cannot borrow `s` as mutable more than once at a time
```

***可变引用与不可变引用不能同时存在***

```rust
let mut s = String::from("hello");

let r1 = &s;
let r2 = &s;
let r3 = &mut s;

println!("{}, {}, and {}", r1, r2, r3);

// error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
```

>注意，引用的作用域 `s` 从创建开始，一直持续到它最后一次使用的地方，这个跟变量的作用域有所不同，变量的作用域从创建持续到某一个花括号 `}`

### 16. 悬垂引用

悬垂引用也叫做悬垂指针，意思为指针指向某个值后，这个值被释放掉了，而指针仍然存在，其指向的内存可能不存在任何值或已被其它变量重新使用。

```rust
fn main() {
    let reference_to_nothing = dangle();
}

fn dangle() -> &String {
    let s = String::from("hello");

    &s
} // 这里 s 离开作用域并被丢弃。其内存被释放。
```

因为 `s` 是在 `dangle` 函数内创建的，当 `dangle` 的代码执行完毕后，`s` 将被释放，但是此时我们又尝试去返回它的引用。这意味着这个引用会指向一个无效的 `String`。

其中一个很好的解决方法是直接返回 `String`：

```rust
fn no_dangle() -> String {
    let s = String::from("hello");

    s
}
```

这样就没有任何错误了，最终 `String` 的 **所有权被转移给外面的调用者**。

### 17. 字符串切片

字符串切片是非常危险的操作，因为切片的索引是通过字节来进行，但是字符串又是 UTF-8 编码，因此你无法保证索引的字节刚好落在字符的边界上。

```rust
let hello = "中国人";

let s = &hello[0..2];
```

我们索引的字节落在了 `中` 字符的内部，这种返回没有任何意义。

### 18. 字符串深度剖析

- 首先向操作系统请求内存来存放 `String` 对象
- 在使用完成后，将内存释放，归还给操作系统

对于第二点，就是百家齐放的环节，在有**垃圾回收 GC** 的语言中，GC 来负责标记并清除这些不再使用的内存对象，这个过程都是自动完成，无需开发者关心，非常简单好用；但是在无 GC 的语言中，需要开发者手动去释放这些内存对象，就像创建对象需要通过编写代码来完成一样，未能正确释放对象造成的后果简直不可估量。

对于 Rust 而言，如果使用 GC，那么会牺牲性能；如果使用手动管理内存，那么会牺牲安全，这该怎么办？为此，Rust 的开发者想出了一个无比惊艳的办法：变量在离开作用域后，就自动释放其占用的内存。

### 19. 结构体的所有权

把结构体中具有所有权的字段转移出去后，将无法再访问该字段，但是可以正常访问其它的字段。

### 20. Option 枚举用于处理空值

在对 `Option<T>` 进行 `T` 的运算之前必须将其转换为 `T`。通常这能帮助我们捕获到空值最常见的问题之一：期望某值不为空但实际上为空的情况。

为了拥有一个可能为空的值，你必须要显式的将其放入对应类型的 `Option<T>` 中。接着，当使用这个值时，必须明确的处理值为空的情况。只要一个值不是 `Option<T>` 类型，你就 **可以** 安全的认定它的值不为空。

### 21. 想在循环中获取元素的索引

```rust
fn main() {
    let a = [4, 3, 2, 1];
    // `.iter()` 方法把 `a` 数组变成一个迭代器
    for (i, v) in a.iter().enumerate() {
        println!("第{}个元素是{}", i + 1, v);
    }
}
```

### 22. for ... in 的使用方法
|使用方法|等价使用方式|所有权|
|---|---|---|
|`for item in collection`|`for item in IntoIterator::into_iter(collection)`|转移所有权|
|`for item in &collection`|`for item in collection.iter()`|不可变借用|
| `for item in &mut collection` | `for item in collection.iter_mut()` |可变借用|

### 23. Match 模式匹配

- `match` 的匹配必须要穷举出所有可能，因此这里用 `_` 来代表未列出的所有可能性
- `match` 的每一个分支都必须是一个表达式，且所有分支的表达式最终返回值的类型必须相同
- **X | Y**，类似逻辑运算符 `或`，代表该分支可以匹配 `X` 也可以匹配 `Y`，只要满足一个即可

其实 `match` 跟其他语言中的 `switch` 非常像，`_` 类似于 `switch` 中的 `default`。

### 24. If let 匹配

```rust
if let Some(3) = v {
    println!("three");
}
```

**当你只要匹配一个条件，且忽略其他条件时就用 `if let` ，否则都用 `match`**。

### 25. maches! 宏

```rust
let foo = 'f';
assert!(matches!(foo, 'A'..='Z' | 'a'..='z'));

let bar = Some(4);
assert!(matches!(bar, Some(x) if x > 2));
```

### 26. 匹配守卫

**匹配守卫**（_match guard_）是一个位于 `match` 分支模式之后的额外 `if` 条件，它能为分支模式提供更进一步的匹配条件。

这个条件可以使用模式中创建的变量：

```rust
let num = Some(4);

match num {
    Some(x) if x < 5 => println!("less than five: {}", x),
    Some(x) => println!("{}", x),
    None => (),
}
```

### 27. @绑定

`@`（读作 at）运算符允许为一个字段绑定另外一个变量。下面例子中，我们希望测试 `Message::Hello` 的 `id` 字段是否位于 `3..=7` 范围内，同时也希望能将其值绑定到 `id_variable` 变量中以便此分支中相关的代码可以使用它。

```rust
enum Message {
    Hello { id: i32 },
}

let msg = Message::Hello { id: 5 };

match msg {
    Message::Hello { id: id_variable @ 3..=7 } => {
        println!("Found an id in range: {}", id_variable)
    },
    Message::Hello { id: 10..=12 } => {
        println!("Found an id in another range")
    },
    Message::Hello { id } => {
        println!("Found some other id: {}", id)
    },
}
```

当你既想要限定分支范围，又想要使用分支的变量时，就可以用 `@` 来绑定到一个新的变量上，实现想要的功能。

#### @前绑定后解构

使用 `@` 还可以在绑定新变量的同时，对目标进行解构：

```rust
#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    // 绑定新变量 `p`，同时对 `Point` 进行解构
    let p @ Point {x: px, y: py } = Point {x: 10, y: 23};
    println!("x: {}, y: {}", px, py);
    println!("{:?}", p);


    let point = Point {x: 10, y: 5};
    if let p @ Point {x: 10, y} = point {
        println!("x is 10 and y is {} in {:?}", y, p);
    } else {
        println!("x was not 10 :(");
    }
}
```

### 28. 为具体的泛型类型实现方法

对于 `Point<T>` 类型，你不仅能定义基于 `T` 的方法，还能针对特定的具体类型，进行方法定义：

```rust
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

这段代码意味着 `Point<f32>` 类型会有一个方法 `distance_from_origin`，而其他 `T` 不是 `f32` 类型的 `Point<T>` 实例则没有定义此方法。这个方法计算点实例与坐标 `(0.0, 0.0)` 之间的距离，并使用了只能用于浮点型的数学运算符。

### 29. 泛型的性能

在 Rust 中泛型是零成本的抽象，意味着你在使用泛型时，完全不用担心性能上的问题。

但是任何选择都是权衡得失的，Rust 是在编译期为泛型对应的多个类型，生成各自的代码，因此损失了编译速度和增大了最终生成文件的大小。

### 30. const 泛型

针对类型实现的泛型，所有的泛型都是为了抽象不同的类型，const 泛型则是针对值的泛型。

```rust
fn display_array<T: std::fmt::Debug, const N: usize>(arr: [T; N]) {
    println!("{:?}", arr);
}
fn main() {
    let arr: [i32; 3] = [1, 2, 3];
    display_array(arr);

    let arr: [i32; 2] = [1, 2];
    display_array(arr);
}
```

如上所示，我们定义了一个类型为 `[T; N]` 的数组，其中 `T` 是一个基于类型的泛型参数，这个和之前讲的泛型没有区别，而重点在于 `N` 这个泛型参数，它是一个基于值的泛型参数！因为它用来替代的是数组的长度。

### 31. 特征孤儿规则

**如果你想要为类型** `A` **实现特征** `T`**，那么** `A` **或者** `T` **至少有一个是在当前作用域中定义的**。

### 32. 使用特征作为函数参数

```rust
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

`impl Summary`，顾名思义，它的意思是 **实现了 `Summary` 特征** 的 `item` 参数。

你可以使用任何实现了 `Summary` 特征的类型作为该函数的参数，同时在函数体内，还可以调用该特征的方法，例如 `summarize` 方法。

### 33. 多重约束

除了单个约束条件，我们还可以指定多个约束条件，例如除了让参数实现 `Summary` 特征外，还可以让参数实现 `Display` 特征以控制它的格式化输出：

```rust
pub fn notify(item: &(impl Summary + Display)) {}
```

除了上述的语法糖形式，还能使用特征约束的形式：

```rust
pub fn notify<T: Summary + Display>(item: &T) {}
```

通过这两个特征，就可以使用 `item.summarize` 方法，以及通过 `println!("{}", item)` 来格式化输出 `item`。

### 34. Where 约束

```rust
fn some_function<T, U>(t: &T, u: &U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{}
```

### 35. 使用条件约束有条件地实现方法或特征

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self {
            x,
            y,
        }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

`cmp_display` 方法，并不是所有的 `Pair<T>` 结构体对象都可以拥有，只有 `T` 同时实现了 `Display + PartialOrd` 的 `Pair<T>` 才可以拥有此方法。该函数可读性会更好，因为泛型参数、参数、返回值都在一起，可以快速的阅读，同时每个泛型参数的特征也在新的代码行中通过**特征约束**进行了约束。

**也可以有条件地实现特征**, 例如，标准库为任何实现了 `Display` 特征的类型实现了 `ToString` 特征：

```rust
impl<T: Display> ToString for T {
    // --snip--
}
```

### 36. 通过 derive 派生特征

形如 `#[derive(Debug)]` 的代码，这种是一种特征派生语法，被 `derive` 标记的对象会自动实现对应的默认特征代码，继承相应的功能。

例如 `Debug` 特征，它有一套自动实现的默认代码，当你给一个结构体标记后，就可以使用 `println!("{:?}", s)` 的形式打印该结构体的对象。

### 37. 特征对象

可以通过 `&` 引用或者 `Box<T>` 智能指针的方式来创建特征对象。

```rust
trait Draw {
    fn draw(&self) -> String;
}

impl Draw for u8 {
    fn draw(&self) -> String {
        format!("u8: {}", *self)
    }
}

impl Draw for f64 {
    fn draw(&self) -> String {
        format!("f64: {}", *self)
    }
}

// 若 T 实现了 Draw 特征， 则调用该函数时传入的 Box<T> 可以被隐式转换成函数参数签名中的 Box<dyn Draw>
fn draw1(x: Box<dyn Draw>) {
    // 由于实现了 Deref 特征，Box 智能指针会自动解引用为它所包裹的值，然后调用该值对应的类型上定义的 `draw` 方法
    x.draw();
}

fn draw2(x: &dyn Draw) {
    x.draw();
}

fn main() {
    let x = 1.1f64;
    // do_something(&x);
    let y = 8u8;

    // x 和 y 的类型 T 都实现了 `Draw` 特征，因为 Box<T> 可以在函数调用时隐式地被转换为特征对象 Box<dyn Draw> 
    // 基于 x 的值创建一个 Box<f64> 类型的智能指针，指针指向的数据被放置在了堆上
    draw1(Box::new(x));
    // 基于 y 的值创建一个 Box<u8> 类型的智能指针
    draw1(Box::new(y));
    draw2(&x);
    draw2(&y);
}
```

上面代码，有几个非常重要的点：

- `draw1` 函数的参数是 `Box<dyn Draw>` 形式的特征对象，该特征对象是通过 `Box::new(x)` 的方式创建的
- `draw2` 函数的参数是 `&dyn Draw` 形式的特征对象，该特征对象是通过 `&x` 的方式创建的
- `dyn` 关键字只用在特征对象的类型声明上，在创建时无需使用 `dyn`

因此，可以使用特征对象来代表泛型或具体的类型。

### 38. 特征对象的动态分发

泛型是在编译期完成处理的：编译器会为每一个泛型参数对应的具体类型生成一份代码，这种方式是**静态分发(static dispatch)**，因为是在编译期完成的，对于运行期性能完全没有任何影响。

与静态分发相对应的是**动态分发(dynamic dispatch)**，在这种情况下，直到运行时，才能确定需要调用什么方法。之前代码中的关键字 `dyn` 正是在强调这一“动态”的特点。

### 39. Self 和 self

在 Rust 中，有两个 `self`，一个指代当前的实例对象，一个指代特征或者方法类型的别名：

```rust
trait Draw {
    fn draw(&self) -> Self;
}

#[derive(Clone)]
struct Button;
impl Draw for Button {
    fn draw(&self) -> Self {
        return self.clone()
    }
}

fn main() {
    let button = Button;
    let newb = button.draw();
}
```

上述代码中，`self` 指代的就是当前的实例对象，也就是 `button.draw()` 中的 `button` 实例，`Self` 则指代的是 `Button` 类型。

### 40. 特征对象的限制

不是所有特征都能拥有特征对象，只有对象安全的特征才行。当一个特征的所有方法都有如下属性时，它的对象才是安全的：

- 方法的返回类型不能是 `Self`
- 方法没有任何泛型参数

### 41. 关联类型

使用泛型，你将得到以下的代码：

```rust
trait Container<A,B> {
    fn contains(&self,a: A,b: B) -> bool;
}

fn difference<A,B,C>(container: &C) -> i32
  where
    C : Container<A,B> {...}
```

可以看到，由于使用了泛型，导致函数头部也必须增加泛型的声明，而使用关联类型，将得到可读性好得多的代码：

```rust
trait Container{
    type A;
    type B;
    fn contains(&self, a: &Self::A, b: &Self::B) -> bool;
}

fn difference<C: Container>(container: &C) {}
```

### 42. 完全限定语法

```rust
fn main() {
    println!("A baby dog is called a {}", <Dog as Animal>::baby_name());
}
```

在尖括号中，通过 `as` 关键字，我们向 Rust 编译器提供了类型注解，也就是 `Animal` 就是 `Dog`，而不是其他动物，因此最终会调用 `impl Animal for Dog` 中的方法

### 43. 特征定义中的特征约束

有时，我们会需要让某个特征 A 能使用另一个特征 B 的功能(另一种形式的特征约束)，这种情况下，不仅仅要为类型实现特征 A，还要为类型实现特征 B 才行，这就是 `supertrait` (实在不知道该如何翻译，有大佬指导下嘛？)

例如有一个特征 `OutlinePrint`，它有一个方法，能够对当前的实现类型进行格式化输出：

```rust
use std::fmt::Display;

trait OutlinePrint: Display {
    fn outline_print(&self) {
        let output = self.to_string();
        let len = output.len();
        println!("{}", "*".repeat(len + 4));
        println!("*{}*", " ".repeat(len + 2));
        println!("* {} *", output);
        println!("*{}*", " ".repeat(len + 2));
        println!("{}", "*".repeat(len + 4));
    }
}
```

### 44. NewType

在特征章节中，有提到孤儿规则，简单来说，就是特征或者类型必需至少有一个是本地的，才能在此类型上定义特征。

这里提供一个办法来绕过孤儿规则，那就是使用 **newtype 模式**。就是为一个元组结构体创建新类型。该元组结构体封装有一个字段，该字段就是希望实现特征的具体类型。

下面来看一个例子，我们有一个动态数组类型： `Vec<T>`，它定义在标准库中，还有一个特征 `Display`，它也定义在标准库中，如果没有 `newtype`，我们是无法为 `Vec<T>` 实现 `Display` 的：

```rust
use std::fmt;

struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec![String::from("hello"), String::from("world")]);
    println!("w = {}", w);
}
```

注意到我们怎么访问里面的数组吗？`self.0.join(", ")`，是的，很啰嗦，因为需要先从 `Wrapper` 中取出数组: `self.0`，然后才能执行 `join` 方法。

类似的，任何数组上的方法，你都无法直接调用，需要先用 `self.0` 取出数组，然后再进行调用。

Rust 提供了一个特征叫 `Deref`，实现该特征后，可以自动做一层类似类型转换的操作，可以将 `Wrapper` 变成 `Vec<String>` 来使用。这样就会像直接使用数组那样去使用 `Wrapper`，而无需为每一个操作都添加上 `self.0`。

同时，如果不想 `Wrapper` 暴露底层数组的所有方法，我们还可以为 `Wrapper` 去重载这些方法，实现隐藏的目的。

### 45. 存储不同类型的元素

数组的元素必须类型相同，但是也提到了解决方案：那就是通过使用枚举类型和特征对象来实现不同类型元素的存储。先来看看通过枚举如何实现：

```rust
#[derive(Debug)]
enum IpAddr {
    V4(String),
    V6(String)
}
fn main() {
    let v = vec![
        IpAddr::V4("127.0.0.1".to_string()),
        IpAddr::V6("::1".to_string())
    ];

    for ip in v {
        show_addr(ip)
    }
}

fn show_addr(ip: IpAddr) {
    println!("{:?}",ip);
}
```

数组 `v` 中存储了两种不同的 `ip` 地址，但是这两种都属于 `IpAddr` 枚举类型的成员，因此可以存储在数组中。

再来看看特征对象的实现：

```rust
trait IpAddr {
    fn display(&self);
}

struct V4(String);
impl IpAddr for V4 {
    fn display(&self) {
        println!("ipv4: {:?}",self.0)
    }
}
struct V6(String);
impl IpAddr for V6 {
    fn display(&self) {
        println!("ipv6: {:?}",self.0)
    }
}

fn main() {
    let v: Vec<Box<dyn IpAddr>> = vec![
        Box::new(V4("127.0.0.1".to_string())),
        Box::new(V6("::1".to_string())),
    ];

    for ip in v {
        ip.display();
    }
}
```

比枚举实现要稍微复杂一些，我们为 `V4` 和 `V6` 都实现了特征 `IpAddr`，然后将它俩的实例用 `Box::new` 包裹后，存在了数组 `v` 中，需要注意的是，这里必须手动地指定类型：`Vec<Box<dyn IpAddr>>`，表示数组 `v` 存储的是特征 `IpAddr` 的对象，这样就实现了在数组中存储不同的类型。

在实际使用场景中，**特征对象数组要比枚举数组常见很多**，主要原因在于特征对象非常灵活，而编译器对枚举的限制较多，且无法动态增加类型。
### 46. 函数签名中的生命周期标注

从两个字符串切片中返回较长的那个

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

- 和泛型一样，使用生命周期参数，需要先声明 `<'a>`
- `x`、`y` 和返回值至少活得和 `'a` 一样久 (因为返回值要么是 `x`，要么是 `y`)

### 47. 结构体中的生命周期

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");
    let i = ImportantExcerpt {
        part: first_sentence,
    };
}
```

`ImportantExcerpt` 结构体中有一个引用类型的字段 `part`，因此需要为它标注上生命周期。结构体的生命周期标注语法跟泛型参数语法很像，需要对生命周期参数进行声明 `<'a>`。该生命周期标注说明，**结构体 `ImportantExcerpt` 所引用的字符串 `str` 必须比该结构体活得更久**。

### 48. 生命周期消除

1. **每一个引用参数都会获得独自的生命周期**
    
    例如一个引用参数的函数就有一个生命周期标注: `fn foo<'a>(x: &'a i32)`，两个引用参数的有两个生命周期标注:`fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`, 依此类推。
    
2. **若只有一个输入生命周期(函数参数中只有一个引用类型)，那么该生命周期会被赋给所有的输出生命周期**，也就是所有返回值的生命周期都等于该输入生命周期
    
    例如函数 `fn foo(x: &i32) -> &i32`，`x` 参数的生命周期会被自动赋给返回值 `&i32`，因此该函数等同于 `fn foo<'a>(x: &'a i32) -> &'a i32`
    
3. **若存在多个输入生命周期，且其中一个是 `&self` 或 `&mut self`，则 `&self` 的生命周期被赋给所有的输出生命周期**
    
    拥有 `&self` 形式的参数，说明该函数是一个 `方法`，该规则让方法的使用便利度大幅提升。

### 49. 生命周期约束

- `'a: 'b`，是生命周期约束语法，跟泛型约束非常相似，用于说明 `'a` 必须比 `'b` 活得久
- 可以把 `'a` 和 `'b` 都在同一个地方声明（如上），或者分开声明但通过 `where 'a: 'b` 约束生命周期关系，如下：

```rust
impl<'a> ImportantExcerpt<'a> {
    fn announce_and_return_part<'b>(&'a self, announcement: &'b str) -> &'b str
    where
        'a: 'b,
    {
        println!("Attention please: {}", announcement);
        self.part
    }
}
```

### 50. 静态生命周期

在 Rust 中有一个非常特殊的生命周期，那就是 `'static`，拥有该生命周期的引用可以和整个程序活得一样久。

```rust
let s: &'static str = "我没啥优点，就是活得久，嘿嘿";
```

### 51. 线程 panic 后，程序是否会终止

如果是 `main` 线程，则程序会终止，如果是其它子线程，该线程会终止，但是不会影响 `main` 线程。因此，尽量不要在 `main` 线程中做太多任务，将这些任务交由子线程去做，就算子线程 `panic` 也不会导致整个程序的结束。

### 52. 对返回的错误进行处理

直接 `panic` 还是过于粗暴，因为实际上 IO 的错误有很多种，我们需要对部分错误进行特殊处理，而不是所有错误都直接崩溃：

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => panic!("Problem opening the file: {:?}", other_error),
        },
    };
}
```

上面代码在匹配出 `error` 后，又对 `error` 进行了详细的匹配解析，最终结果：

- 如果是文件不存在错误 `ErrorKind::NotFound`，就创建文件，这里创建文件`File::create` 也是返回 `Result`，因此继续用 `match` 对其结果进行处理：创建成功，将新的文件句柄赋值给 `f`，如果失败，则 `panic`
- 剩下的错误，一律 `panic`

### 53. 失败就 panic: unwrap 和 except

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap();
}
```

如果调用这段代码时 _hello.txt_ 文件不存在，那么 `unwrap` 就将直接 `panic`
`expect` 跟 `unwrap` 很像，也是遇到错误直接 `panic`, 但是会带上自定义的错误提示信息，相当于重载了错误打印的函数：

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}
```

### 54. ? 宏

```rust
use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```

其实 `?` 就是一个宏，它的作用跟下面的 `match`

```rust
let mut f = match f {
    // 打开文件成功，将file句柄赋值给f
    Ok(file) => file,
    // 打开文件失败，将错误返回(向上传播)
    Err(e) => return Err(e),
};
```

### 55. ? 用于 Option 的返回

```rust
fn first(arr: &[i32]) -> Option<&i32> {
   let v = arr.get(0)?;
   Some(v)
}
```

上面的函数中，`arr.get` 返回一个 `Option<&i32>` 类型，因为 `?` 的使用，如果 `get` 的结果是 `None`，则直接返回 `None`，如果是 `Some(&i32)`，则把里面的值赋给 `v`。

### 56. 易混淆的 Package 和包

牢记 `Package` 是一个项目工程，而包只是一个编译单元，基本上也就不会混淆这个两个概念了：`src/main.rs` 和 `src/lib.rs` 都是编译单元，因此它们都是包。

### 57. 结构体和枚举的可见性

- 将结构体设置为 `pub`，但它的所有字段依然是私有的
- 将枚举设置为 `pub`，它的所有字段也将对外可见

