# Rust 编程之道 1 - 6 章

## 新时代的语言

1. 设计哲学：内存安全、零成本抽象、实用性。
2. 常见内存错误：引用空指针、使用未初始化内存、释放后使用、缓冲区溢出、重复释放。
3. 从 Haskell 借鉴的特性：没有空指针、默认不可变、表达式、高阶函数、代数数据类型、模式匹配、泛型、trait 和关联类型、本地类型推导。
4. 还实现了： 仿射类型（Affine Type），表达 Move 语义；借用；生命周期。支持硬实时系统，从 C++ 借鉴确定性析构、RAII 和智能指针。

## 语言精要

### 闭包和函数

CTFE
: Compile-Time Function Execution，编译时函数执行

Rust 中的闭包实际上就是由一个匿名结构体和 trait 来实现的。

使用 `const fn foo() -> Type {}` 定义编译时就能运行并确定的值，用于比如定义数组：`[0; foo()]`。由 miri（一个 MIR 解释器）来执行的。

bool 和整数类型间的转换： as 操作符支持将 bool 类型转成 0 和 1，但是并不支持数转为 bool 类型。

### 4 种指针类型：

- 引用指针；
- 原生指针（主要用于 unsafe 中，分为不可变原生指针 *const T，和可变原生指针 *mut T）；
- 函数指针；
- 智能指针。

### 3 种枚举体：

- 无参数枚举体（Color::Red）；
- 类 C 枚举体（Color::Red = 0xff0000 带值）；
- 携带参数类型枚举体（IpAddr::V6(String)，本质上还是 fn(String) -> IpAddr 的函数指针。

### match 引用类型

`match &Some("hello".to_string())` 这样的枚举体，Rust 2015 的版本需要使用 &Some(ref s) 这样的语法来解构其中 s 的引用，之后只需要 Some(s) 即可。

## 类型系统

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

对智能指针 a: Box<String> 解引用 _a 等价于 _(a.deref())，Box<T> 的 deref 的实现为 `fn deref(&self) -> &T { &**self }` ，即 `*&**&a`，同时会引起所有权的转移（解引用移动），目前该行为只支持 Box<T>，使用了 #[lang="owned_box"] 进行标识。

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

高阶生命周期（for<> 语法）：`Box<for<'f> DoSomething<&'f usize>>`

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

IterMut 迭代器示意：`struct IterMut<T> { ptr: *mut T, end: *mut T };`

迭代器适配器，又叫包装器，可以将一个接口转换成所需要的另一个接口。常用适配器：Map, Chain（连接两个迭代器）, Clone, Cycle, Enumerate, Filter, FlatMap, FilterMap, Fuse（快速结束遍历）, Rev。

## 结构化编程

结构体**更新语法**：`let obj2 = Obj { name: 2, ..obj1 }`，obj1 的所有权并未被转移。Rust 中的结构体属于代数数据类型中的**积类型**。

枚举体属于代数类型中的**和类型**。如果说积类型表示逻辑与（合取），那和类型就表示逻辑或（析取）。

`match *self` 不会获取 self 的所有权。

Rust 中的 Newtype 模式：`struct PrintDrop(&'static str);`，为了实现：

1. 隐藏实际类型，限制功能；
2. 明确语义。这样的语义提升是零成本的，没有多余的性能开销；
3. 是复制语义的类型具有移动语义。

Rust 满足的 4 种设计模式：

1. 针对接口编程：trait；
2. 组合优于继承：泛型和 trait 限定；
3. 分离变和不变：类型系统；
4. 委托代替继承：常用的迭代器。

## 字符串与集合类型

### 字符串和排序 trait

#### 字符串

UTF-8 编码规则大致如下：

1. 字符在 ASCII 范围内时，就用一个字节表示，第一个 bit 为 0.
2. 当一个字符占用了 n 个字节时，第一个字节的前 n 位设置为 1，第 n+1 位设置为 0，后面字节的前两位设置为 10.

Rust 中 char 类型使用整数值与 Unicode 标量一一对应。每个 char 类型都代表一个有效的 u32 类型整数。

&str 类型的字符串可以存储在任意地方：

1. 静态存储器。如 &'static str 类型的字符串存储到已编译的可执行文件中，随着程序一起记载启动。
2. 堆分配。如果是通过 &String 生成的，则是在堆上分配。
3. 栈分配。使用 str::from::utf8 方法，既可以将栈分配的 [u8; N] 数组转换为一个 &str。

正则表达式用 `(?P<name>exp)` 这种格式来定义命名捕获组。

#### 与排序和比较相关的 trait

1. PartialEq 表示部分等价，Eq 代表等价，继承与 PartialEq；
2. PartialOrd 偏序，满足：自反性，反对称性，传递性；
3. Ord 全序，区别于偏序， 还要满足完全性（即任意两个元素可比）。

### HashMap 的底层实现原理

实现一个 HashMap 的过程可以分为三大步骤：

1. 实现一个 Hash 函数。
2. 合理地解决 Hash 冲突。
3. 实现 HashMap 的操作方法。

一个好的 Hash 函数不仅性能优越，而且会让存储与底层数组中的值分布得更加均匀，减少冲突的发生。

**Hash 碰撞**，指两个元素得到相同的索引地址。除了 Hash 函数的好坏外，Hash 冲突还取决于**负载因子**，比如容量 100，存储 90 个键值对，负载因子就是 0.9。

Rust 中的默认 Hash 算法是 SipHash13，可以防止 Hash 碰撞拒绝服务攻击。Rust 还提供了可插拔机制，实现了 Hash 算法的更换。

#### 实现 Hash 函数

Rust 中实现了 **Hash 和 Eq** 两个 trait 的类型，都可以作为 HashMap 的键，**Hash** trait 需要实现 write 方法（得到映射结果），finish 方法（写入结果）。

#### 解决 Hash 冲突

业界一共有四类解决 Hash 冲突的方法：外部拉链法（基于数组和链表的组合来解决冲突，冲突放入链表中）、开放定址法、公共溢出法（把冲突的键放在一个地方）、再 Hash 法。

**开放定址法**则是寻找一个空地址存放。寻找的行为叫做**探测**，一个一个寻找叫做线性探测。

Rust 使用罗宾汉算法（开放定址法 + 线性探测），空桶正常插入，被占用时，如果该键值对的探测次数比当前插入的键值对的探测次数少（它属于「富翁」），则插入这个位置，接着找下一个位置来安置被替换的「富翁」。

## 构建健壮的程序

非正常情况大概可以分为三类：失败、错误、异常。

Rust 提供了分层式错误处理方案：

1. Option<T>
2. Result<T, E>
3. 线程恐慌（Panic）
4. 程序中止（Abort）

高效处理 Option：`self.map(f: FnOnce) => Some(f(x)) / None` 方法，或者 `self.and_then(f: FnOnce) => f(x) / None`。

## 模块化编程

可见性和私有性：

1. pub(self)：self 内可见。
2. pub(in outer_mod)：在 outer_mod 中可见。
3. pub(crate)：整个 crate 可见。
4. pub(super)：父模块可见。
5. ::outer_mod：统一路径，代表从根模块开始寻找相应的模块路径。

## 安全并发

### 基本概念

使用并发的两个主要原因：性能和容错。

事件驱动、异步回调（Promise, Future）、协程。

当计算的正确性取决于多个线程交替执行的顺序时，就会产生**竞态条件**。产生竞态条件的区域，叫做**临界区**。当一个线程写一个变量而另一个线程在读一个变量时，就会引起**数据竞争**。并非所有的竞态条件都是数据竞争，也并非所有的数据竞争都是竞态条件。

消除竞态条件的关键在于判断出正确的临界区。

通常可以使用**锁、信号量、屏障、条件变量**机制来实现线程同步。锁又分为互斥锁（Mutex）、读写锁（RwLock）和自旋锁。

### Send 和 Sync

实现了 Send 的类型，可以安全地在线程间传递**所有权**；实现了 Sync 的类型，可以在安全地在线程间传递**不可变借用**。

**'static** 限定表示类型 T 只能是**非引用类型**，除了 &'static。Send 和 Sync 这两个 trait 是 unsafe 的，就是说 Rust 为所有类型设定好了**默认的线程安全规则**。

### 锁

如果线程在获得锁之后发生恐慌，则称这种情况为**中毒**。

RwLock 锁产生的**读锁**和**写锁**必须使用**显式作用域块**隔离开，因为读写锁不能同时存在。

### 生成器、协程和 Future

#### 生成器和协程

协程分为**有栈协程**和**无栈协程**。无栈协程一般基于**状态机**来实现，具体的应用形式叫**生成器**。

通过 `let mut gen = || { yield 1; return 2; };` 的语法，类似于闭包的方式，生成一个匿名结构体 `Generator<Yield = T, Return = U>` 来表示一个生成器。

生成器的性能比迭代器更高。因为生成器是一种**延迟计算或惰性计算**。如果生成器只保留 Return 的类型，生成器就能化身为 Future: `Generator<Yield = (), Return = Result<T, E>>`。

生成器只属于一种**半协程**，因为只能在生成器和调用者之间进行跳转，不能在生成器之间进行跳转。

#### Future

Future trait 定义：

```rust
pub trait Future {
   type Output;
   fn poll(self: Pin<&mut Self>, lw: &LocalWaker) -> Poll<Self::Output>;
}
```

poll 是对**轮询**的一种抽象，Poll 是一个枚举类型：`Read(T) / Pending`。

Future 还需要 Executor 和 Task 共同完成。

几个关键的复合类型：

1. ThreadPool：包含字段 state，设置为 `Arc<PoolState>` 类型，为了共享线程内的线程信息。
2. PoolState：包含 tx 和 rx 两个字段，用于 Channel 通信。
3. Message：枚举类型，包括运行任务 Run(Tssk) 和关闭 Close。
4. Task：包含 future: FutureObj 和 wake_handle: `Arc<WeakHandle>`（唤醒任务的句柄）。

#### Pin 和 Unpin

生成器会由编译器生成相应的结构体来记录状态，当生成器包含对本地变量的引用时，该结构体会生成一种**自引用结构体**。

生成器中 resume() 方法中会用到 std::mem::replace()，会造成自引用类型形成悬垂指针。

Pin<T> 代表将数据的内存位置牢牢地**钉在**原地。Unpin 代表数据可以安全地移动。大多数类型都实现了 Unpin。

## 元编程

### 概况

元编程技术大概分为以下几类：简单文本替换（c 的 define），类型模板，反射，语法扩展，代码自动生成（如 GO 的 go generate）。

反射：自我访问、检测和修改其自身状态或行为的能力。Rust 使用 `std::any::Any` 来实现。

### 宏

Rust 编译的六个阶段：分词阶段、解析阶段、提炼 HIR、提炼 MIR、转译为 LLVM IR、生成机器码。

在宏内部定义依赖宏：`macro_rules! xxx { (@count $(xxx) => (xxx)) }`

生命宏在展开后，不会污染原来的词法作用域，具有这种特性的宏叫做卫生宏。Rust 的卫生宏并不完整，像生命周期、类型等无法保证卫生性。

宏中使用 $crate 关键字表示当前 crate。

Rust 在 libsyntax 基础上，抽象出通用的接口，即**过程宏**，被定义于内置的 libproc_mac 中，过程宏建立在**词条流（TokenSteam）**的基础上。将修改后的词条流交给语法解析器来处理。

## 超越安全的边界

### Unsafe

Unsafe Rust 在进行以下五种操作时，不会提供任何安全检查：

1. 解引用裸指针。
2. 调用 unsafe 的函数或方法。
3. 访问或修改可变静态变量。
4. 读写 Union 联合体中的字段。

Rust 提供了 *const T 和 *mut T 两种指针类型。

replace/swap 是一种内存不安全的方法，不过 std::mem 模块也提供安全的方法。

Vec 的 insert 方法中的 unsafe 块，通过断言判断指定插入的 index 无法越界操作，以及通过判断长度是否达到容量极限来决定是否进行扩容。

型变的三种形式：协变、逆变、不变。

### PhantomData 类型

`PhantomData<T>` 是一个**零大小类型的标记结构体**，也叫做「幻影类型」，还扮演以下三种其他角色：型变、标记拥有关系、自动 trait 实现。

标准库中 `Vec<T>` 的实现，拥有 `buf: RawVec<T>` 类型，RawVec 类型拥有 `ptr: Unique<T>` 类型，Unique 类型拥有 `_marker: PhantomData<T>` 类型，来保证 Vec 对 T 是拥有关系，这样 drop 检查就会严格要求开发者保证析构顺序，同时在析构函数中注意不要使用拥有的数据。

### NonNull 指针

NonNull 旨在成为 Unsafe Rust **默认的原生指针**。因为 \*const T 和 \*mut T 基本上是等价，只是不能从 \*const T 直接得到 &mut T。

使用 `NonNull<T>` 就不需要进行转换，等价于一个**协变版本的 \*mut T**，但还需要 PhantomData 在必要时提供**不变**或**加强 drop 检查**。

类型 `NonNull<T: ?Sized>` 拥有一个字段 `pointer: NonZero<*const T>`，NonZero 的作用就是协变和非零。

使用 NonNull::dangling 创建一个悬垂 NonNull 指针。
