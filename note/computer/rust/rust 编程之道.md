# Rust 编程之道 1 - 6 章

## 新时代的语言

设计哲学
: 内存安全、零成本抽象、实用性。

常见内存错误
: 引用空指针、使用未初始化内存、释放后使用、缓冲区溢出、重复释放。

从 Haskell 借鉴的特性
: 没有空指针、默认不可变、表达式、高阶函数、代数数据类型、模式匹配、泛型、trait 和关联类型、本地类型推导。

还实现了
: 仿射类型（Affine Type），表达 Move 语义；借用；生命周期。支持硬实时系统，从 C++ 借鉴确定性析构、RAII 和智能指针。

## 语言精要

### 闭包和函数

CTFE
: Compile-Time Function Execution，编译时函数执行

闭包实现
: Rust 中的闭包实际上就是由一个匿名结构体和 trait 来实现的。

使用 `const fn foo() -> Type {}` 定义编译时就能运行并确定的值，用于比如定义数组：`[0; foo()]`。由 miri（一个 MIR 解释器）来执行的。

bool 和整数类型间的转换： as 操作符支持将 bool 类型转成 0 和 1，但是并不支持数转为 bool 类型。

### 4 种指针类型：

- 引用指针；
- 原生指针（主要用于 unsafe 中，分为不可变原生指针 *const T，和可变原生指针 *mut T）；
- 函数指针；
- 智能指针。

### 3 种枚举体：

- 无参数枚举体（Color::Red）；
- 类 C 枚举体（Color::Red = 0xff0000带值）；
- 携带参数类型枚举体（IpAddr::V6(String)，本质上还是 fn(String) -> IpAddr 的函数指针。

### match 引用类型

`match &Some("hello".to_string())` 这样的枚举体，Rust 2015 的版本需要使用 &Some(ref s) 这样的语法来解构其中 s 的引用，之后只需要 Some(s) 即可。

##  类型系统

### 多态

现代编程语言包含了三种多态类型：

1. 参数化多态
2. Ad-hoc 多态
3. 子类型多态。

前两个属于静多态，子类型属于动多态（静动多态 Rust 都支持）。参数化多态指泛型，Ad-hoc 多态（又叫特定多态）在 Rust 中指 trait。

### ZST 类型

- 除了 ST 和 DST，Rust 还支持 ZST（Zero Sized Type，零大小类型），比如单元结构体、单元类型、由单元类型组成的数组，它们的值就是它们本身。
- 使用 vec![(); n] 迭代类型循环 n 次，获得更高的性能。
- 标准库中的 HashSet<T> 和 BTreeSet<T> 其实是利用了 HashMap<K, ()>。

### 深入 trait

Rust 编程的哲学：组合优于继承

- Self 是每个 trait 都带有的隐式类型参数，代表当前 trait 的具体类型。
- Add<RHS=Self> 语法表示 RHS 的默认类型是 Self。
- Rust 遵循一条孤儿规则（Orphan Rule）：如果要实现某个 trait，那么该 trait 和要实现该 trait 的那个类型至少要有一个在当前 crate 中定义。（Box<T> 除外，使用了一个 #[fundamental] 的属性标识）
- 为一个类型实现 trait 时，必须为关联类型（比如 type Output;）指定具体类型。
抽象类型

也叫做存在类型，相对于具体类型，抽象类型无法直接实例化，它的每个实例都是具体类型的实例。编译器无法确定抽象类型的确切的功能和所占空间的大小，Rust 提供两种方法来处理抽象类型：trait 对象和 impl Trait。

### trait 对象

TraitObject 包括了两个指针：data 指针和 vtable 指针，该名称来源于 C++，可以称之为虚表。

对象安全的 trait 需要满足：

1. trait 中不包含关联常量
2. trait 的 Self 类型不能被限定为 Sized
3. 所有的方法必须是对象安全的，需要满足一下两点之一
    1. 方法受 Self:Sized 约束；
    2. 或者同时满足：
        1. 方法不包含泛型参数；
        2. 第一个参数为 Self 类型或者可以解引用为 Self 的类型（没有接收者的方法对于 trait 对象来说毫无意义）；
        3. Self 不能出现在除第一个参数之外的地方，包括返回值。

### impl Trait

- 在 Rust 2018 中引入，如果说 trait 对象是装箱抽象类型，那么 impl Trait 就是拆箱抽象类型。目前 impl Trait 只可以在输入的参数和返回值这两个位置使用。用于返回值时，实际上等价于给返回类型增加了一种 trait 限制范围。
- 为了在语义上与 impl Trait 对应，Rust 2018 增加了一种新的语法 dyn Trait 来对应动态分发的 trait 对象。

### 标签 trait

- 5 个重要的标签 trait：Sized, Unsize, Copy, Send（跨线程安全通信）, Sync（跨线程安全共享引用）
- Rust 语言项：#[lang="add"]
- 动态大小使用时，需要遵循如下限制规则：
- 只能通过胖指针来操作 Unsize 类型，比如 &[T]、&Trait；
- 变量、参数和枚举变量不能使用动态大小类型；
- 结构体只有最后一个字段可以用动态大小类型。

### Deref 解引用

Rust 中的隐式类型转换基本只有自动解引用。

String + &String 能够运行的原因就在于 String 类型实现了 `Deref<Target=str>` 这个 Trait。除此之外，Box、Rc、Arc、Vec 等都实现了 Deref trait。

match String 字符串时，如 x: String; 需要 `match &*x { "xxx" => /* do sth */ }` 才能手动解引用为 &str。

### 当前 trait 系统的不足

- 孤儿规则的局限性
- 复用率不高（由于重叠规则，目前在 nightly 中提供了 impl 特化来解决）
- 抽象表达能力有待改进

抽象能力有待改进的例子：迭代器。只能按值迭代，必须重新分配数据，而不能通过引用来复用原始数据。比如 std::io::Lines 不能重用内部缓存区，影响了性能。这是由于迭代器的实现基于关联类型：type Item; 只支持具体类型，不支持泛型。而生命周期函数就是一种泛型，无生命周期函数也就无引用类型。

所以需要实现泛型关联类型（Generic Associated Type，GAT），例如 type Item<'a>; 目前还未实现。

## 内存管理

RAII（Resouce Acquisition Is Initialization），资源获取即初始化。离开作用域时进行析构，同时被叫做作用域界定的资源管理。

利用 `Option<Rc<RefCell<Node<T>>>>` 来演示 Rust 并不能完全保证内存不会泄露。不过循环引用也可以通过使用 Rc::downgrade 得到 Weak<T> 类型来解决这个问题。

## 所有权系统

### 概要

为所有成员都是复制语义类型的结构体实现 Copy trait：#[derive(Copy, Clone)]

借用所有权会让所有者受到如下限制（相当于读写锁）：

1. 不可变借用期间：不能修改资源，不能再借出可变借用；
2. 可变借用期间：不能访问资源，不能再借出所有权。

### 生命周期

标注生命周期并不能改变生命周期长短，只用于编译器的借用检查。

生命周期省略规则：

1. 每个省略的生命周期都是不同的生命周期 'a, 'b, 'c ...；
2. 如果只有一个输入带生命周期，且生命周期为 'life，则所有的都是 'life；
3. 如果输入中包含 &self 或 &mut self，则 self 的生命周期分配给输出。

trait 对象的生命周期：Box<Foo<'a> + 'a>

### 智能指针解引用

对智能指针 a: Box<String> 解引用 *a 等价于 *(a.deref())，Box<T> 的 deref 的实现为 `fn deref(&self) -> &T { &**self }` ，即 `*&**&a`，同时会引起所有权的转移（解引用移动），目前该行为只支持 Box<T>，使用了 #[lang="owned_box"] 进行标识。

### 其他

#### 内部可变性

Cell<T> 可以实现 immut 结构体中成员的可变，方法：Point.y.set() / Point.y.get()。

RefCell<T> 没有 Copy 限制的实现内部可变，不过会有一个运行时借用检查器，检查是否违反借用规则。

#### 写时复制

写时复制 Cow<T>，枚举体智能指针（Borrowed，Owned，分别用于包裹引用和所有者）。to_mut 获取可变借用，引用克隆，所有者不克隆。into_owned 获取所有权对象，引用克隆，所有者转移所有权。

#### 非词法作用域

NLL（Non-Lexical Lifetime，非词法作用域生命周期），利用控制流程图 CFG（一个 DAG）表示程序执行过程中所有可能的流转。

## 函数、闭包与迭代器

### 函数

fn r#match()，使用 r#（原生标识符操作）作为前缀，可使用关键字为函数命名。

### 闭包

在 Rust 中，闭包是编译器临时制造的 struct + trait 的语法糖。

高阶生命周期（for<> 语法）：Box<for<'f> DoSomething<&'f usize>>

闭包 Trait 实现规则：

1. 如果没有捕获任何环境变量，实现 Fn；
2. 如果捕获复制语义类型变量：
    1. 如果不需要修改，无论是否使用 move，自动实现 Fn；
    2. 需要修改，实现 FnMut；
3. 如果捕获移动语义类型变量：
    1. 不修改 + 无 move：FnOnce；
    2. 不修改 + move：Fn；
    3. 修改：FnMut
4. move + 复制语义类型：自动实现 Copy/Clone

### 迭代器

String.reserve(Integer)，确保扩展的字节长度，避免平凡分配。

IterMut 迭代器示意：struct IterMut<T> { ptr: *mut T, end: *mut T };

迭代器适配器，又叫包装器，可以将一个接口转换成所需要的另一个接口。常用适配器：Map, Chain（连接两个迭代器）, Clone, Cycle, Enumerate, Filter, FlatMap, FilterMap, Fuse（快速结束遍历）, Rev。