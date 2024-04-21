参考文档：[Rust 程序设计语言 - Rust 程序设计语言 简体中文版 (kaisery.github.io)](https://kaisery.github.io/trpl-zh-cn/)
ide：vscode，安装 rust 插件

- rust-analyzer
- Even Better TOML

crate：[crates.io: Rust Package Registry](https://crates.io/)


# ----- 入门指南

## hello world

编写 Rust 程序

- 程序文件后缀名：rs
- 文件命名规范：`hello_world.rs`

编译与运行 Rust 程序

- 编译：`rustc main.rs`
- 运行：
	- Windows：`.\main.exe`
	- Linux/mac：`./main`

编译和运行是单独的两步

- 运行 Rust 程序之前必须先编译，命令为：`rustc 源件名`
- 编译成功后，会生成一个二进制文件（windows 下的 `.exe` 文件）
	- 在 Windows 上还会生成一个 `.pdb` 文件，里面包含调试信息 
- Rust 是 ahead-of-time 编译的语言
	- 可以先编译程序，然后把可执行文件交给别人运行（无需安装 Rust）
- `rustc` 只适合简单的 Rust 程序...

## Cargo

Cargo 是 Rust 的构建系统和包管理工具，构建代码、下载依赖的库、构建这些库...

安装 Rust 的时候会安装 Cargo，`cargo --version`

创建项目 `cargo new hello_cargo`

- 项目名称也是 `hello_cargo`
- 会创建一个新的目录 `hello_cargo`，包含
	- Cargo.toml
	- src 目录
		- main.rs
	- 初始化了一个新的 Git 仓库，`.gitignore`
		- 可以使用其它的 VCS
		- 或不使用 VCS：`cargo new` 的时候使用 `--vcs` 这个 flag

Cargo.toml

- TOML (Tom's Obvious, Minimal Language) 格式，是 Cargo 的配置格式
- `[package]`，是一个区域标题，表示下方内容是用来配置包的
	- name，项目名
	- version，项目版本
	- authors，项目作者
	- edition，使用的 Rust 版本
- `[dependencies]`，另一个区域的开始，它会列出项目的依赖项
	- 在 Rust 里面，代码的包称作 **crate**

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2021"

[dependencies]
```

src/main.rs

- cargo 生成的 main.rs 在 src 目录下，源代码都应该在 src 目录下
- 顶层目录可以放置：README、许可信息、配置文件和其它与程序源码无关的文件
- 如果创建项目时没有使用 cargo，也可以把项目转化为使用 cargo
	- 把源代码文件移动到 src 下
	- 创建 Cargo.toml 并填写相应的配置

构建 Cargo 项目 `cargo build`

> 在 `hello_cargo` 目录下，输入 `cargo build` 构建项目

- 这个命令会创建一个**可执行文件** `target/debug/hello_cargo`（Windows 上是 `hello_cargo.exe`）
	- 由于默认的构建方法是调试构建（debug build），Cargo 会将可执行文件放在名为 debug 的目录中
- 第一次运行 `cargo build` 会在顶层目录生成 cargo.lock 文件
	- 该文件负责追踪项目依赖的精确版本
	- 不需要（也不要）手动修改该文件

构建和运行 Cargo 项目 `cargo run`

- 编译代码+执行结果
- 如果之前编译成功过，并且源码没有改变，那么就会直接运行二进制文件

cargo check

- 检查代码，确保能通过编译，但是==不产生任何可执行文件== 
- cargo check 要比 cargo build **快得多**
	- 编写代码的时候可以连续反复的使用 cargo check 检查代码，提高效率

**发布构建**

- `cargo build --release`
	- 编译时会进行优化
		- 代码会运行的更快，但是编译时间更长
	- 会在 target/release 而不是 target/debug 生成可执行文件
- 两种配置:
	- 一个开发
	- 一个正式发布

# ----- 猜数游戏

猜数游戏-目标

- 生成一个 1 到 100 间的随机数
- 提示玩家输入一个猜测
- 猜完之后，程序会提示猜测是太小了还是太大了
- 如果猜测正确，那么打印出一个庆祝信息，程序退出

## 处理一次猜测

```rust
use std::io;

fn main() {
    println!("猜数!");

    println!("猜测一个数");

    let mut guess = String::new();

    io::stdin().read_line(&mut guess).expect("无法读取行");

    println!("你猜测的数是：{}", guess);
}
```

### 引入 IO 标准库

```rust
use std::io;
```

将 `io` 输入/输出库引入当前作用域。`io` 库来自于**标准库**（也被称为 `std`）

- 默认情况下，Rust 设定了若干个会**自动导入**到每个程序作用域中的**标准库内容**，这组内容被称为 _预导入（preclude）_ 内容。
- 如果需要的类型不在**预导入内容**中，就必须使用 `use` 语句显式地将其引入作用域

> 可以在[标准库文档](https://doc.rust-lang.org/std/prelude/index.html)中查看预导入的所有内容

### 使用变量存储值

```rust
let mut guess = String::new();
```

Rust 中，变量默认是不可变的，一旦给变量赋值，这个值就不再可以修改了

- 在变量名前使用 `mut` 来使一个变量可变

`guess` 所绑定的值是 `String::new` 的结果，这个函数会返回一个 `String` 的新实例。[`String`](https://doc.rust-lang.org/std/string/struct.String.html) 是一个标准库提供的**字符串类型**

- `::` 语法表明 `new` 是 `String` 类型的一个 **关联函数**（_associated function_）。
- 关联函数是针对**类型**实现的，在这个例子中是 `String`，而不是 `String` 的某个特定实例。一些语言中把它称为 **静态方法**（_static method_）

> #todo 类型 `==` 类？

### 接受用户输入

```rust
io::stdin().read_line(&mut guess)
```

如果程序的开头没有使用 `use std::io;` 引入 `io` 库，我们仍可以通过把函数调用写成 `std::io::stdin` 来使用函数

- `stdin` 函数返回一个 [`std::io::Stdin`](https://doc.rust-lang.org/std/io/struct.Stdin.html) 的实例，这代表终端标准输入句柄的类型。

将 `&mut guess` 作为参数传递给 `read_line()` 函数，让其将用户输入储存到这个字符串中。

- `read_line` 的工作是，无论用户在标准输入中键入什么内容，都将其追加（不会覆盖其原有内容）到一个字符串中，因此它需要字符串作为参数。这个字符串参数应该是**可变的**
- `&` 表示这个参数是一个 **引用**（_reference_），它允许多处代码访问同一处数据，而无需在内存中多次拷贝。
	- 引用是一个复杂的特性，Rust 的一个主要优势就是安全而简单的操纵引用。
	- 它像变量一样，**默认是不可变的**。因此，需要写成 `&mut guess` 来使其可变，而不是 `&guess`

### 使用 Result 类型来处理潜在的错误

```rust
io::stdin().read_line(&mut guess).expect("Failed to read line");
```

`read_line` 会将用户输入附加到传递给它的字符串中，返回一个类型为 `Result` 的值

- [`Result`](https://doc.rust-lang.org/std/result/enum.Result.html) 是一种 [_枚举类型_](https://kaisery.github.io/trpl-zh-cn/ch06-00-enums.html)，通常也写作 _enum_。枚举类型变量的值可以是多种可能状态中的一个。我们把每种可能的状态称为一种 _枚举成员（variant）_
- `Result` 的成员是 `Ok` 和 `Err`，`Ok` 成员表示操作成功，内部包含==成功时产生的值==。`Err` 成员则意味着操作失败，并且包含失败的前因后果。

`Result` 类型的作用是编码错误处理信息。`Result` 的实例拥有 [`expect` 方法](https://doc.rust-lang.org/std/result/enum.Result.html#method.expect)

- 如果 `io::Result` 实例的值是 `Err`，`expect` 会导致程序崩溃，并显示当做参数传递给 `expect` 的信息。如果 `read_line` 方法返回 `Err`，则可能是来源于底层操作系统错误的结果。
- 如果 `Result` 实例的值是 `Ok`，`expect` 会获取 `Ok` 中的值并原样返回，在本例中，这个值是==用户输入到标准输入中的字节数==。

如果不调用 `expect`，程序也能编译，不过会出现一个警告

```bash
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
warning: unused `Result` that must be used
  --> src/main.rs:10:5
   |
10 |     io::stdin().read_line(&mut guess);
   |     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   |
   = note: `#[warn(unused_must_use)]` on by default
   = note: this `Result` may be an `Err` variant, which should be handled

warning: `guessing_game` (bin "guessing_game") generated 1 warning
    Finished dev [unoptimized + debuginfo] target(s) in 0.59s

```

- Rust 警告我们没有使用 `read_line` 的返回值 `Result`，说明有一个可能的错误没有处理。


### 使用 println! 占位符打印值

`println!` 是一个在屏幕上打印字符串的宏，里面的 `{}` 是预留在特定位置的占位符

当打印变量的值时，变量名可以写进大括号中

```rust
println!("你猜测的数是：{}", guess);
println!("你猜测的数是：{guess}");
```

## 生成一个神秘数字

Rust 标准库中尚未包含随机数功能。然而，Rust 团队还是提供了一个包含上述功能的 rand crate

### 使用 crate 来增加更多功能

_crate_ 是一个 Rust 代码包。我们正在构建的项目是一个 _二进制 crate_，它生成一个可执行文件。 `rand` crate 是一个 _库 crate_，库 crate 可以包含任意能被其他程序使用的代码，但是不能自执行

在 _Cargo.toml_ 文件中，`[dependencies]` 片段告诉 Cargo 本项目依赖了哪些外部 crate 及其版本。

- 本例中使用语义化版本 `0.8.5` 来指定 `rand` crate。`0.8.5` 事实上是 `^0.8.5` 的**简写**，它表示任何至少是 `0.8.5` 但小于 `0.9.0` 的版本。
- Cargo 认为这些版本与 `0.8.5` 版本的公有 API 相兼容，这样的版本指定确保了我们可以获取能使本章代码编译的最新的补丁（patch）版本。
	- 任何大于等于 `0.9.0` 的版本不能保证和接下来的示例采用了相同的 API

> Cargo 理解 [语义化版本（Semantic Versioning）](http://semver.org/)（有时也称为 _SemVer_），这是一种==定义版本号的标准==。


#### Cargo.lock 文件确保构建是可重现的

Cargo 有一个机制来确保任何人在任何时候重新构建代码，都会产生相同的结果：Cargo 只会使用你指定的依赖版本，除非你又手动指定了别的。

> 例如，如果下周 `rand` crate 的 `0.8.6` 版本出来了，它修复了一个重要的 bug，同时也含有一个会破坏代码运行的缺陷。为了处理这个问题，Rust 在你第一次运行 `cargo build` 时建立了 _Cargo.lock_ 文件

- 当**第一次**构建项目时，Cargo 计算出所有符合要求的依赖版本并写入 _Cargo.lock_ 文件。
- 当将来构建项目时，Cargo 会发现 _Cargo.lock_ 已存在并使用其中指定的版本，而不是再次计算所有的版本。这使得你拥有了一个自动化的可重现的构建。
- 换句话说，项目会持续使用 `0.8.5` 直到你**显式升级**，多亏有了 _Cargo.lock_ 文件。
- 由于 _Cargo.lock_ 文件对于“可重复构建”非常重要，因此它通常会和项目中的其余代码一样纳入到版本控制系统中。

####  更新 crate 到新版本

当你 **确实** 需要升级 crate 时，Cargo 提供了这样一个命令 `update`，

- 它会==忽略 _Cargo.lock_ 文件==，
- 计算出所有符合 _Cargo.toml_ 声明的最新版本，并写入 _Cargo.lock_ 文件。
- 不过，Cargo 默认只会寻找大于 `0.8.5` 而小于 `0.9.0` 的版本

如果想要使用 `0.9.0` 版本的 `rand` 或是任何 `0.9.x` 系列的版本，必须像这样更新 _Cargo.toml_ 文件：

```toml
[dependencies]  
rand = "0.9.0"
```

### 生成一个随机数

```rust
use rand::Rng;
use std::io;

fn main() {
    println!("猜数游戏开始!");

    println!("请猜测一个数!");
    let mut guess = String::new();
    io::stdin().read_line(&mut guess).expect("无法读取行");
    println!("你猜测的数是：{guess}");

    // 生成一个随机数
    let secret_number = rand::thread_rng().gen_range(1..=100);
    println!("生成的随机数是：{secret_number}")
}
```

```rust
use rand::Rng;
```

- `Rng` 是一个 *trait*，它定义了随机数生成器应实现的方法，想使用这些方法的话，此 trait 必须在作用域中

```rust
let secret_number = rand::thread_rng().gen_range(1..=100);
```

- `rand::thread_rng` 函数提供实际使用的**随机数生成器**：它位于当前执行线程的本地环境中，并从操作系统获取 seed
- 接着调用随机数生成器的 `gen_range` 方法。
	- 这个方法由 `use rand::Rng` 语句引入到作用域的 `Rng` trait 定义。
	- `gen_range` 方法获取一个**范围表达式**（range expression）作为参数，并生成一个在此范围之间的随机数。
- 这里使用的这类范围表达式使用了 `start..=end` 这样的形式，也就是说包含了上下端点，
	- 所以需要指定 `1..=100` 来请求一个 1 和 100 之间的数。

> 你不可能凭空就知道应该 use 哪个 trait 以及该从 crate 中调用哪个方法，因此每个 crate 有使用说明文档。
> 
> Cargo 有一个很棒的功能是：运行 `cargo doc --open` 命令来构建所有本地依赖提供的文档，并在浏览器中打开

## 比较猜测的数字和秘密数字

```rust
use rand::Rng;
use std::cmp::Ordering;
use std::io;

fn main() {
    // --snip--

    println!("You guessed: {guess}");

    match guess.cmp(&secret_number) {
        Ordering::Less => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal => println!("You win!"),
    }
}
```

首先从标准库引入了一个叫做 `std::cmp::Ordering` 的类型到作用域中。 

- `Ordering` 也是一个枚举，不过它的成员是 `Less`、`Greater` 和 `Equal`。这是比较两个值时可能出现的三种结果。

`cmp` 方法用来比较两个值并可以在任何可比较的值上调用。

- 它获取一个被比较值的引用：这里是把 `guess` 与 `secret_number` 做比较。
- 然后它会返回一个刚才通过 `use` 引入作用域的 `Ordering` 枚举的成员。

使用一个 [`match`](https://kaisery.github.io/trpl-zh-cn/ch06-02-match.html) 表达式，根据对 `guess` 和 `secret_number` 调用 `cmp` 返回的 `Ordering` 成员来决定接下来做什么

- 一个 `match` 表达式由 **分支（arms）** 构成。一个分支包含
	- **模式**（_pattern_）
	- 表达式开头的值与分支模式相匹配时应该执行的代码
- Rust 获取提供给 `match` 的值并挨个检查每个分支的模式

> `match` 结构和模式是 Rust 中强大的功能，它体现了代码可能遇到的多种情形，并帮助你确保没有遗漏处理。

此时编译代码会报错，

```shell
  --> src/main.rs:22:21
   |
22 |     match guess.cmp(&secret_number) {
   |                 --- ^^^^^^^^^^^^^^ expected struct `String`, found integer
   |                 |
   |                 arguments to this function are incorrect
```

错误的核心表明这里有 **不匹配的类型**（_mismatched types_）。Rust 有一个**静态强类型系统**，同时也有**类型推断**。

- 当我们写出 `let guess = String::new()` 时，Rust 推断出 `guess` 应该是 `String` 类型，并不需要我们写出类型。
- 另一方面，`secret_number`，是数字类型。
	- 几个数字类型拥有 1 到 100 之间的值：32 位数字 `i32`；32 位无符号数字 `u32`；64 位数字 `i64` 等等。Rust 默认使用 `i32`，
- 所以 `secret_number` 是 `i32` 类型。这里错误的原因在于 Rust ==不会比较字符串类型和数字类型==。

所以我们必须把 guess 转换为一个数字类型，才好与秘密数字进行比较

```rust
let mut guess = String::new();
......
let guess: u32 = guess.trim().parse().expect("Please type a number!");
```

这里新创建了一个叫做 `guess` 的变量。Rust 允许用一个新值来 **隐藏** （_Shadowing_） `guess` 之前的值。

- 这个功能常用在需要**转换值类型**之类的场景。它允许我们**复用** `guess` 变量的名字，而不是被迫创建两个不同变量，
- 诸如 `guess_str` 和 `guess` 之类

这个新变量绑定到 `guess.trim().parse()` 表达式上。

- `String` 实例的 *trim 方法* 会==去除字符串开头和结尾的空白字符==，我们必须执行此方法才能将字符串与 `u32` 比较，因为 `u32` 只能包含数值型数据。
	- 用户必须输入 enter 键才能让 `read_line` 返回并输入他们的猜想，这将会在字符串中增加一个换行（newline）符。
		- 例如，用户输入 5 并按下 enter（在 Windows 上，按下 enter 键会得到一个回车符和一个换行符，`\r\n`），
	- `guess` 看起来像这样：`5\n` 或者 `5\r\n`。`trim` 方法会消除 `\n` 或者 `\r\n`，只留下 `5`
- [字符串的 `parse` 方法](https://doc.rust-lang.org/std/primitive.str.html#method.parse) 将字符串转换成其他类型。这里用它来把字符串转换为数值。
	- 我们需要告诉 Rust 具体的数字类型，这里通过 `let guess: u32` 指定，`u32` 是一个无符号的 32 位整型。对于不大的正整数来说，它是不错的默认类型

程序中的 `u32` 注解以及与 `secret_number` 的比较，意味着 ==Rust 会推断出 secret_number 也是 u32 类型==。现在可以使用相同类型比较两个值了！

`parse` 方法只有在字符逻辑上可以转换为数字的时候才能工作所以非常容易出错，因此，`parse` 方法返回一个 `Result` 类型，再次按部就班的用 `expect` 方法处理即可

完整代码：

```rust
use rand::Rng;
use std::cmp::Ordering;
use std::io;

fn main() {
    println!("猜数游戏开始!");

    println!("请猜测一个数!");
    let mut guess = String::new();
    io::stdin().read_line(&mut guess).expect("无法读取行");
    println!("你猜测的数是：{guess}");

    // 生成一个随机数
    let secret_number = rand::thread_rng().gen_range(1..=100);
    println!("生成的随机数是：{secret_number}");

    let guess: u32 = guess.trim().parse().expect("请输入一个数字");
    match guess.cmp(&secret_number) {
        Ordering::Less => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal => println!("You win!"),
    }
}
```

## 使用循环允许多次猜测

`loop` 关键字创建了一个无限循环。我们会增加循环来给用户更多机会猜数字：

```rust
// --snip--

println!("The secret number is: {secret_number}");

loop {
	println!("Please input your guess.");

	// --snip--

	match guess.cmp(&secret_number) {
		Ordering::Less => println!("Too small!"),
		Ordering::Greater => println!("Too big!"),
		Ordering::Equal => println!("You win!"),
	}
}
```

### 猜测正确后退出

增加一个 `break` 语句，在用户猜对时退出游戏

```rust
        // --snip--

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("You win!");
                break;
            }
        }
    }
}
```

### 处理无效输入

为了进一步改善游戏性，不要在用户输入非数字时崩溃，需要忽略非数字，让用户可以继续猜测。可以通过修改 `guess` 将 `String` 转化为 `u32` 那部分代码来实现

```rust
// --snip--

// let guess: u32 = guess.trim().parse().expect("请输入一个数字");
let guess: u32 = match guess.trim().parse() {
	Ok(num) => num,
	Err(_) => continue,
};

// --snip--
```

将 `expect` 调用换成 `match` 语句，以从遇到错误就崩溃转换为**处理错误**。

- `parse` 返回一个 `Result` 类型，而 `Result` 是一个拥有 `Ok` 或 `Err` 成员的枚举
- 如果 `parse` 能够成功的将字符串转换为一个数字，它会返回一个包含结果数字的 `Ok`。这个 `Ok` 值与 `match` 第一个分支的模式相匹配，该分支对应的动作返回 `Ok` 值中的数字 `num`，最后如愿变成新创建的 `guess` 变量
- 如果 `parse` **不**能将字符串转换为一个数字，它会返回一个包含更多错误信息的 `Err`。`Err` 值会匹配第二个分支的 `Err(_)` 模式：
	- `_` 是一个通配符值，本例中用来匹配所有 `Err` 值，不管其中有何种信息。所以程序会执行第二个分支的动作，
	- `continue` 意味着进入 `loop` 的下一次循环，请求另一个猜测。

## 最终代码

```rust
use rand::Rng;
use std::cmp::Ordering;
use std::io;

fn main() {
    println!("猜数游戏开始!");
    
     // 生成一个随机数
     let secret_number = rand::thread_rng().gen_range(1..=100);
     // println!("生成的随机数是：{secret_number}");

    loop {
        println!("请猜测一个数!");
        let mut guess = String::new();
        io::stdin().read_line(&mut guess).expect("无法读取行");
        // 类型转换、处理无效输入
        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => {
                print!("不要输入数字以外的字符，");
                continue;
            }
        };
        println!("你猜测的数是：{guess}");

        match guess.cmp(&secret_number) {
            Ordering::Less => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal => {
                println!("You win!");
                break;
            }
        }
    }
}
```

# ----- 常见编程概念

## 变量与可变性

变量默认是**不可改变的（immutable）**。这是 Rust 提供给你的众多优势之一，让你得以充分利用 Rust 提供的安全性和简单并发性来编写代码

当变量不可变时，一旦值被绑定一个名称上，你就不能改变这个值

```rust
fn main() {
    let x = 5;
    println!("The value of x is: {x}");
    x = 6;
    println!("The value of x is: {x}");
}
```

以上代码运行后会报错：

```shell
 --> src\main.rs:4:5
  |
2 |     let x = 5;
  |         -
  |         |
  |         first assignment to `x`
  |         help: consider making this binding mutable: `mut x`
3 |     println!("The value of x is: {x}");
4 |     x = 6;
  |     ^^^^^ cannot assign twice to immutable variable
```

**可以在变量名前添加 `mut` 来使其可变**

```rust
fn main() {
    let mut x = 5;
    println!("The value of x is: {x}");
    x = 6;
    println!("The value of x is: {x}");
}
```

### 常量

类似于不可变变量，_常量 (constants)_ 是绑定到一个名称的不允许改变的值，不过常量与变量还是有一些区别。

- 不允许对常量使用 `mut`。
- 常量不光默认不可变，它**总是不可变**。
- 声明常量使用 `const` 关键字而不是 `let`，并且 _必须_ 注明值的类型
- 常量可以在**任何作用域**中声明，包括全局作用域，这在一个值需要被很多部分的代码用到时很有用。
- 常量只能被设置为**常量表达式**，而不可以是其他任何只能在运行时计算出的值。

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

常量的命名约定是在单词之间使用**全大写加下划线**。

> 有关声明常量时可以使用哪些操作的详细信息，请参阅 [Rust Reference 的常量求值部分](https://doc.rust-lang.org/reference/const_eval.html)。

### 隐藏

我们可以定义一个与之前变量**同名的新变量**。Rustacean 们称之为第一个变量被第二个 **隐藏（Shadowing）** 了，这意味着当您使用变量的名称时，编译器将看到第二个变量，直到第二个变量自己也被隐藏或第二个变量的作用域结束。

可以用相同变量名称来隐藏一个变量，以及**重复使用** `let` 关键字来多次隐藏，如下所示：

```rust
fn main() {
    let x = 5;

    let x = x + 1;

    {
        let x = x * 2; 
        println!("The value of x in the inner scope is: {x}"); // 12
    }

    println!("The value of x is: {x}"); // 6
}
```

> 这个程序首先将 `x` 绑定到值 `5` 上。接着通过 `let x =` 创建了一个新变量 `x`，获取初始值并加 `1`，这样 `x` 的值就变成 `6` 了。
> 
> 然后，在使用花括号创建的内部作用域内，第三个 `let` 语句也隐藏了 `x` 并创建了一个新的变量，将之前的值乘以 `2`，`x` 得到的值是 `12`。
> 
> 当该作用域结束时，内部 shadowing 的作用域也结束了，`x` 又返回到 `6`

隐藏与将变量标记为 `mut` 是有区别的。

- 当不小心尝试对变量重新赋值时，如果**没有使用 `let` 关键字**，就会导致编译时错误。
- 通过使用 `let`，我们可以用这个值进行一些计算，不过计算完之后变量仍然是不可变的。
- 当再次使用 `let` 时，
	- 实际上**创建了一个新变量**，
	- 我们可以**改变值的类型**，并且**复用**这个名字。

例如，假设程序请求用户输入空格字符来说明希望在文本之间显示多少个空格，接下来我们想将输入存储成数字（多少个空格）：

```rust
let spaces = "   ";     
let spaces = spaces.len();
```

> 第一个 `spaces` 变量是字符串类型，第二个 `spaces` 变量是数字类型。隐藏使我们不必使用不同的名字，如 `spaces_str` 和 `spaces_num`；相反，我们可以复用 `spaces` 这个更简单的名字

如果尝试使用 `mut`，将会得到一个编译时错误

```rust
let mut spaces = "   ";
spaces = spaces.len();
```

这个错误说明，我们不能改变变量的类型：

```shell
 --> src/main.rs:3:14
  |
2 |     let mut spaces = "   ";
  |                      ----- expected due to this value
3 |     spaces = spaces.len();
  |              ^^^^^^^^^^^^ expected `&str`, found `usize`
```

## 数据类型

在 Rust 中，每一个值都属于某一个 **数据类型**（_data type_），这告诉 Rust 它被指定为何种数据，以便明确数据处理方式。

我们将看到两类数据类型子集：*标量（scalar）* 和 *复合（compound）*

Rust 是 **静态类型**（_statically typed_）语言，也就是说在==编译时就必须知道所有变量的类型==。

- 根据值及其使用方式，编译器通常可以推断出我们想要用的类型。
- **当多种类型均有可能时，必须增加类型注解**

> 比如第二章的 [“比较猜测的数字和秘密数字”](https://kaisery.github.io/trpl-zh-cn/ch02-00-guessing-game-tutorial.html#%E6%AF%94%E8%BE%83%E7%8C%9C%E6%B5%8B%E7%9A%84%E6%95%B0%E5%AD%97%E5%92%8C%E7%A7%98%E5%AF%86%E6%95%B0%E5%AD%97) 使用 `parse()` 将 `String` 转换为数字时

```rust
let guess: u32 = "42".parse().expect("Not a number!");
```

如果不像上面的代码这样添加类型注解 `: u32`，Rust 会显示如下错误，这说明编译器需要我们提供更多信息，来了解我们想要的类型：

```shell
 --> src/main.rs:2:9
  |
2 |     let guess = "42".parse().expect("Not a number!");
  |         ^^^^^
  |
help: consider giving `guess` an explicit type
  |
2 |     let guess: _ = "42".parse().expect("Not a number!");
  |              +++
```

### 标量类型

**标量**（_scalar_）类型代表一个单独的值。

Rust 有四种基本的标量类型：*整型*、*浮点型*、*布尔类型*和*字符类型*

#### 整型

**整数** 是一个没有小数部分的数字

Rust 内建的整数类型如下：

|长度|有符号|无符号|
|---|---|---|
|8-bit| `i8` | `u8` |
|16-bit| `i16` | `u16` |
|32-bit| `i32` | `u32` |
|64-bit| `i64` | `u64` |
|128-bit| `i128` | `u128` |
|arch| `isize` | `usize` |

> `isize` 和 `usize` 类型依赖运行程序的计算机架构：64 位架构上它们是 64 位的，32 位架构上它们是 32 位的。

整型字面值：

- 除了 byte 类型外，所有的数字字面值都允许使用类型后缀，例如 `57u8`
- 如果不确定使用哪种类型，可以使用 Rust 相应的默认类型，数字类型默认是 `i32`
- 允许使用 `_` 做为分隔符以方便读数，例如 `1_000`，它的值与你指定的 `1000` 相同。

|数字字面值|例子|
|---|---|
|Decimal (十进制)|`98_222`|
|Hex (十六进制)|`0xff`|
|Octal (八进制)|`0o77`|
|Binary (二进制)|`0b1111_0000`|
|Byte (单字节字符)(仅限于`u8`)|`b'A'`|

整数溢出：

- 比方说有一个 `u8` ，它可以存放 `0~255` 的值。那么当你将其修改为 `256` 时会发生什么呢？这被称为 “整型溢出”（“integer overflow” ），这会导致以下两种行为之一的发生。
	- 当在 debug 模式编译时，Rust 检查这类问题并使程序 _panic_
	- 发布模式下编译（`--release`），Rust **不会**检测可能导致 panic 的整型溢出。
		- 发生整型溢出时，Rust 会进行一种被称为二进制补码 *wrapping*（two's complement wrapping）的操作。
			- 简而言之，比此类型能容纳最大值还大的值会回绕到最小值，值 `256` 变成 `0`，值 `257` 变成 `1`，依此类推。
		- 程序不会 panic，不过变量可能也不会是你所期望的值。
		- 依赖整型溢出 wrapping 的行为被认为是一种错误。

#### 浮点型

Rust 也有两个原生的 **浮点数**（_floating-point numbers_）类型，它们是带小数点的数字。

- Rust 的浮点数类型是 `f32` 和 `f64`，分别占 32 位和 64 位
- 默认类型是 `f64`，因为在现代 CPU 中，它与 `f32` 速度几乎一样，不过精度更高。
- 所有的浮点型都是**有符号**的。
- 浮点数采用 IEEE-754 标准表示。`f32` 是单精度浮点数，`f64` 是双精度浮点数。

```rust
fn main() {     
	let x = 2.0; // f64      
	let y: f32 = 3.0; // f32 
}
```

#### 数值计算

Rust 中的所有数字类型都支持基本数学运算：加法、减法、乘法、除法和取余。

- 整数除法会**向零**舍入到最接近的整数。

```rust
fn main() {
    // addition
    let sum = 5 + 10;

    // subtraction
    let difference = 95.5 - 4.3;

    // multiplication
    let product = 4 * 30;

    // division
    let quotient = 56.7 / 32.2;
    let truncated = -5 / 3; // 结果为 -1

    // remainder
    let remainder = 43 % 5;
}
```

#### 布尔型

- Rust 中的布尔类型有两个可能的值：`true` 和 `false`。
- Rust 中的布尔类型使用 `bool` 表示
- 使用布尔值的主要场景是条件表达式，例如 `if` 表达式

```rust
fn main() {
    let t = true;
    let f: bool = false; // with explicit type annotation
}
```

#### 字符类型

Rust 的 `char` 类型是语言中最原生的字母类型

- 用单引号声明 `char` 字面量，双引号声明字符串字面量
- `char` 类型的大小为四个字节，并代表了一个 Unicode 标量值（Unicode Scalar Value），这意味着它可以比 ASCII 表示更多内容

```rust
fn main() {
    let c = 'z';
    let z: char = 'ℤ'; // with explicit type annotation
    let heart_eyed_cat = '😻';
}
```

> 在 Rust 中，带变音符号的字母（Accented letters），中文、日文、韩文等字符，emoji（绘文字）以及零长度的空白字符都是有效的 `char` 值。Unicode 标量值包含从 `U+0000` 到 `U+D7FF` 和 `U+E000` 到 `U+10FFFF` 在内的值。不过，“字符” 并不是一个 Unicode 中的概念，所以人直觉上的 “字符” 可能与 Rust 中的 `char` 并不符合。

### 复合类型

**复合类型**（_Compound types_）可以将多个值组合成一个类型。

- Rust 有两个原生的复合类型：*元组*（tuple）和 *数组*（array）。

#### 元组类型

元组是一个将多个**其他类型**的值组合进一个复合类型的主要方式。

- 元组**长度固定**，且一旦声明，其长度不会增大或缩小。
- 元组中的==每一个位置都有一个类型==，而且这些不同值的类型也**不必是**相同的
- 不带任何值的元组有个特殊的名称，叫做 **单元（unit）** 元组。这种**值**以及**对应的类型**都写作 `()`，表示空值或空的返回类型。
	- 如果表达式不返回任何其他值，则会隐式返回单元值。

```rust
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1);
    
	let tuple2: () = ();
}
```

从元组中获取单个值，

- 可以使用模式匹配（pattern matching）来**解构**（destructure）元组值
- 也可以使用点号（`.`）后跟值的**索引**来直接访问它们

```rust
fn main() {
    let tup = (500, 6.4, 1);

    let (x, y, z) = tup;

    println!("The value of y is: {y}");
}
```

```rust
fn main() {
    let x: (i32, f64, u8) = (500, 6.4, 1);

    let five_hundred = x.0;

    let six_point_four = x.1;

    let one = x.2;
}
```

#### 数组类型

- 数组中的每个元素的**类型**必须相同
- 数组**长度固定**
- 应用场景：
	- 当你想要在**栈**（stack）而不是在堆（heap）上为数据分配空间
	- 想要确保总是有固定数量的元素时

```rust
fn main() {
    let a = [1, 2, 3, 4, 5];
}
```

> 数组并不如 vector 类型灵活。vector 类型是标准库提供的一个 **允许** 增长和缩小长度的类似数组的集合类型。当不确定是应该使用数组还是 vector 的时候，那么很可能应该使用 vector。

编写数组的**类型**

- 在方括号中包含：
	- 每个元素的类型，
	- 后跟分号，
	- 再后跟数组元素的数量。

```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
```

创建一个每个元素都为相同值的数组

- 在方括号中包含
	- 指定初始值
	- 分号
	- 元素个数

```rust
// let a = [3, 3, 3, 3, 3]
let a = [3; 5];
```

使用索引来访问数组的元素

- 程序在索引操作中使用一个无效的值时导致 **运行时** 错误。如果索引超出了数组长度，Rust 会 _panic_

> _panic_，这是 Rust 术语，它用于程序因为错误而退出的情况。这种检查必须在运行时进行

```rust
fn main() {
    let a = [1, 2, 3, 4, 5];

    let first = a[0];
    let second = a[1];

    let err = a[5];
}
```

```rust
error: this operation will panic at runtime
 --> src\main.rs:7:10
  |
7 |     let err=a[5];
  |             ^^^^ index out of bounds: the length is 5 but the index is 5
  |
  = note: `#[deny(unconditional_panic)]` on by default
```

## 函数

定义函数：

- Rust 代码中的函数和变量名使用 _snake case_ 规范风格。
- 通过输入 `fn` 后面跟着函数名和一对圆括号来定义函数
- Rust 不关心函数定义所在的位置，只要函数被调用时出现在调用之处可见的作用域内就行
	- another_function 定义在 main 函数前后都可以

> 在 snake case 中，所有字母都是小写并使用下划线分隔单词。

```rust
// fn another_function() {
//     println!("Another function.");
// }

fn main() {
    println!("Hello, world!");

    another_function();
}

fn another_function() {
    println!("Another function.");
}
```

#### 参数

我们可以定义为拥有 **参数**（_parameters_）的函数，参数是**特殊变量**，是**函数签名**的一部分。

- **必须** 声明每个参数的类型

```rust
fn main() {
    print_labeled_measurement(5, 'h');
}

fn print_labeled_measurement(value: i32, unit_label: char) {
    println!("The measurement is: {value}{unit_label}");
}
```

#### 语句和表达式

> Rust 是一门**基于表达式**（expression-based）的语言，这是一个需要理解的（不同于其他语言）重要区别。

函数体由一系列的**语句**和一个**可选的结尾表达式**构成。

- **语句**（_Statements_）是执行一些操作但不返回值的指令。 
	- 函数定义也是语句
	- 语句不返回值。因此，不能把 `let` 语句赋值给另一个变量
- **表达式**（_Expressions_）计算并产生一个值
	- 比如 `5 + 6`，这是一个表达式并计算出值 `11`
	- 表达式可以是语句的一部分
		- 如语句 `let y = 6;` 中的 `6` 是一个表达式
	- 函数调用是一个表达式
	- 宏调用是一个表达式
	- 用大括号创建的一个新的块作用域也是一个表达式

```rust
 --> src/main.rs:2:14
  |
2 |     let x = (let y = 6);
  |              ^^^

error: expected expression, found statement (`let`)
 --> src/main.rs:2:14
  |
2 |     let x = (let y = 6);
  |              ^^^^^^^^^
  |
  = note: variable declaration using `let` is a statement
```

> `let y = 6` 语句并不返回值，所以没有可以绑定到 `x` 上的值。这与其他语言不同，例如 C 和 Ruby，它们的赋值语句会返回所赋的值。在这些语言中，可以这么写 `x = y = 6`，这样 `x` 和 `y` 的值都是 `6`；Rust 中不能这样写

==表达式的结尾没有分号==。

- 如果在表达式的结尾加上分号，它就变成了语句，而语句不会返回值

```rust
fn main() {
    let y = {
        let x = 3;
        x + 1
    };

    println!("The value of y is: {y}");
}
```

表达式：

```rust
{
    let x = 3;
    x + 1
}
```

### 具有返回值的函数

函数可以向调用它的代码返回值。

- 我们并不对返回值命名，但要在箭头（`->`）后声明它的**类型**。
- 在 Rust 中，函数的返回值等同于函数体**最后一个表达式的值**。
- 使用 `return` 关键字和指定值，可从函数中提前返回；但大部分函数**隐式**的返回最后的表达式。

```rust
fn five() -> i32 {
    5
}

fn main() {
    let x = five();

    println!("The value of x is: {x}");
}
```

## 控制流

根据条件是否为真来决定是否执行某些代码，以及根据条件是否为真来重复运行一段代码的能力是大部分编程语言的基本组成部分。Rust 代码中最常见的用来控制执行流的结构是 `if` 表达式和循环。

### if 表达式

`if` 表达式允许根据条件执行不同的代码分支。你提供一个条件并表示 “如果条件满足，运行这段代码；如果条件不满足，不运行这段代码。”

- 条件 **必须** 是 `bool` 值，Rust 并不会尝试自动地将非布尔值转换为布尔值

```rust
fn main() {
    let number = 3;

    if number < 5 {
        println!("condition was true");
    } else {
        println!("condition was false");
    }
}
```

#### 使用 `else if` 处理多重条件

可以将 `else if` 表达式与 `if` 和 `else` 组合来实现多重条件

```rust
fn main() {
    let number = 6;

    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
}
```

#### 在 `let` 语句中使用 `if`

因为 `if` 是一个表达式，我们可以在 `let` 语句的右侧使用它

- `if` 的每个分支的可能的返回值都必须是**相同类型**

```rust
fn main() {
    let condition = true;
    let number = if condition { 5 } else { 6 };

    println!("The value of number is: {number}");
}
```

Rust 需要在编译时就确切的知道 `number` 变量的类型，这样它就可以在编译时验证在每处使用的 `number` 变量的类型是有效的

```rust
error[E0308]: `if` and `else` have incompatible types
 --> src/main.rs:4:44
  |
4 |     let number = if condition { 5 } else { "six" };
  |                                 -          ^^^^^ expected integer, found `&str`
  |                                 |
  |                                 expected because of this
```

### 使用循环重复执行

Rust 有三种循环：`loop`、`while` 和 `for`

#### 使用 `loop` 重复执行代码

`loop` 关键字告诉 Rust 一遍又一遍地执行一段代码直到你明确要求停止。

```rust
fn main() {
    loop {
        println!("again!");
    }
}
```

#### 从循环返回值

`loop` 的一个用例是重试可能会失败的操作，

- 比如检查线程是否完成了任务。然而你可能会需要将操作的结果传递给其它的代码。
- 如果将返回值加入你用来停止循环的 `break` 表达式，它会被停止的循环返回

```rust
fn main() {
    let mut counter = 0;

    let result = loop {
        counter += 1;

        if counter == 10 {
            break counter * 2;
        }
    };

    println!("The result is {result}"); //20
}
```

#### 循环标签：在多个循环之间消除歧义

如果存在嵌套循环，`break` 和 `continue` 应用于此时**最内层**的循环。

你可以选择在一个循环上指定一个 **循环标签**（_loop label_），然后将标签与 `break` 或 `continue` 一起使用，使这些关键字应用于**已标记的循环**而不是最内层的循环

```rust
fn main() {
    let mut count = 0;
    'counting_up: loop {
        println!("count = {count}");
        let mut remaining = 10;

        loop {
            println!("remaining = {remaining}");
            if remaining == 9 {
                break;
            }
            if count == 2 {
                break 'counting_up;
            }
            remaining -= 1;
        }

        count += 1;
    }
    println!("End count = {count}");
}

count = 0
remaining = 10
remaining = 9
count = 1
remaining = 10
remaining = 9
count = 2
remaining = 10
End count = 2
```

> 外层循环有一个标签 `counting_up`，它将从 0 数到 2。没有标签的内部循环从 10 向下数到 9。第一个没有指定标签的 `break` 将只退出内层循环。`break 'counting_up;` 语句将退出外层循环

#### `while` 条件循环

在程序中计算循环的条件也很常见。当条件为 `true`，执行循环。当条件不再为 `true`，调用 `break` 停止循环。这个循环类型可以通过组合 `loop`、`if`、`else` 和 `break` 来实现；如果你喜欢的话，现在就可以在程序中试试。

然而，这个模式太常用了，Rust 为此内置了一个语言结构，它被称为 `while` 循环。这种结构消除了很多使用 `loop`、`if`、`else` 和 `break` 时所必须的嵌套，这样更加清晰

```rust
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{number}!");

        number -= 1;
    }

    println!("LIFTOFF!!!");
}
```

#### 使用 for 遍历集合

使用 `for` 循环来对一个集合的每个元素执行一些代码

- `for` 循环的安全性和简洁性使得它成为 Rust 中使用最多的循环结构

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];

    for element in a {
        println!("the value is: {element}");
    }
}
```

大部分 Rustacean 也会使用 `for` 循环。这么做的方式是使用 `Range`

- 它是标准库提供的类型，用来生成从一个数字开始到另一个数字之前结束的所有数字的序列
- `rev`，用来反转 range

```rust
fn main() {
    for number in (1..4).rev() {
        println!("{number}!");
    }
    println!("LIFTOFF!!!");
}

3!
2!
1!
LIFTOFF!!!
```

# ----- 所有权

## 什么是所有权？

Rust 的核心功能（之一）是 **所有权**（_ownership_）。虽然该功能很容易解释，但它对语言的其他部分有着深刻的影响。

所有程序都必须管理其运行时使用计算机内存的方式。一些语言中具有垃圾回收机制，在程序运行时有规律地寻找不再使用的内存；在另一些语言中，程序员必须亲自分配和释放内存。

Rust 则选择了第三种方式：==通过所有权系统管理内存==，

- 编译器在编译时会根据一系列的规则进行检查。如果违反了任何这些规则，程序都不能编译。
- 在运行时，所有权系统的任何功能都不会减慢程序。

### 栈（Stack）与堆（Heap）

栈和堆都是代码在运行时可供使用的内存，但是它们的结构不同。

栈以放入值的顺序存储值并以相反顺序取出值。这也被称作 **后进先出**（_last in, first out_）。栈中的所有数据都必须占用已知且固定的大小。在编译时大小未知或大小可能变化的数据，要改为存储在堆上。 

堆是缺乏组织的：当向堆放入数据时，你要请求一定大小的空间。内存分配器（memory allocator）在堆的某处找到一块足够大的空位，把它标记为已使用，并返回一个表示该位置地址的 **指针**（_pointer_）。这个过程称作 **在堆上分配内存**（_allocating on the heap_），有时简称为 “分配”（allocating）。

（将数据推入栈中并不被认为是分配）。因为指向放入堆中数据的指针是已知的并且大小是固定的，==你可以将该指针存储在栈上==，不过当需要实际数据时，必须访问指针。

==入栈比在堆上分配内存要快==，因为（入栈时）分配器无需为存储新数据去搜索内存空间；其位置总是在栈顶。相比之下，在堆上分配内存则需要更多的工作，这是因为分配器必须首先找到一块足够存放数据的内存空间，并接着做一些记录为下一次分配做准备。

==访问堆上的数据比访问栈上的数据慢==，因为必须通过指针来访问。现代处理器在内存中跳转越少就越快（缓存）。

当你的代码调用一个函数时，传递给函数的值（包括可能指向堆上数据的指针）和函数的局部变量被压入栈中。当函数结束时，这些值被移出栈。

跟踪哪部分代码正在使用堆上的哪些数据，最大限度的减少堆上的重复数据的数量，以及清理堆上不再使用的数据确保不会耗尽空间，这些问题正是所有权系统要处理的。一旦理解了所有权，你就不需要经常考虑栈和堆了，不过明白了所有权的主要目的就是==管理堆数据==，能够帮助解释为什么所有权要以这种方式工作。

### 所有权规则（重要）

1. Rust 中的每一个【值】都有一个 **所有者**（_owner_）。
2. 【值】在任一时刻**有且只有一个**所有者。
3. 当所有者（变量）**离开作用域**，这个值将被丢弃。

变量的所有权总是遵循相同的模式：将值赋给另一个变量时移动它。

- 当持有**堆**中数据值的变量**离开作用域**时，其值将通过 `drop` 被**清理**掉，
- 除非数据被**移动**为另一个变量所有（赋值、函数参数、函数返回值）

> 以下内容都围绕所有权规则展开

### 变量作用域

作用域是一个项（item）在程序中有效的范围。

```rust
{                      // s 在这里无效，它尚未声明
	let s = "hello";   // 从此处起，s 是有效的

	// 使用 s
}                      // 此作用域已结束，s 不再有效
```

### String 类型

> 我们会专注于 `String` 与所有权相关的部分。这些方面也**同样适用于**标准库提供的或你自己创建的其他复杂数据类型

我们已经见过字符串字面值，即被硬编码进程序里的字符串值。字符串字面值是很方便的，不过它们并不适合使用文本的每一种场景。

- 原因之一就是它们是**不可变的**。
- 另一个原因是并非所有字符串的值都能在编写代码时就知道：
	- 例如，要是想获取用户输入并存储该怎么办呢？为此，

Rust 有另一种字符串类型，`String`。这个类型管理被分配到**堆**上的数据，所以能够存储在编译时未知大小的文本。
- 可以使用 `from` 函数基于字符串字面值来创建 `String`

```rust
let s = String::from("hello");
```

> 这两个冒号 `::` 是运算符，允许将特定的 `from` 函数置于 `String` 类型的命名空间（namespace）下，而不需要使用类似 `string_from` 这样的名字

**可以** 修改此类字符串

```rust
let mut s = String::from("hello");

s.push_str(", world!"); // push_str() 在字符串后追加字面值

println!("{}", s); // 将打印 `hello, world!`

```

> 为什么 `String` 可变而字面值却不行呢？区别在于两个类型对内存的处理上。

### 内存与分配

> 就字符串字面值来说，我们在编译时就知道其内容，所以文本被直接硬编码进最终的可执行文件中。这使得字符串字面值快速且高效。不过这些特性都只得益于字符串字面值的**不可变性**。
> 
> 不幸的是，我们不能为了每一个在编译时大小未知的文本而将一块内存放入二进制文件中，并且它的大小还可能随着程序运行而改变。

对于 `String` 类型，为了支持一个**可变，可增长**的文本片段，需要在**堆**上分配一块在编译时未知大小的内存来存放内容。这意味着：

- 必须在**运行时**向内存分配器（memory allocator）请求内存。
- 需要一个当我们处理完 `String` 时将内存**返回**给分配器的方法。

第一部分由我们完成：当调用 `String::from` 时，它的实现 (_implementation_) 请求其所需的内存

Rust 采取了一个不同的策略：==内存在拥有它的变量离开作用域后就被自动释放==。

 - 这是一个将 `String` 需要的内存返回给分配器的很自然的位置：当 `s` 离开作用域的时候。
 - 当变量离开作用域，Rust 为我们调用一个特殊的函数 [`drop`](https://doc.rust-lang.org/std/ops/trait.Drop.html#tymethod.drop)，在这里 `String` 的作者可以放置释放内存的代码。
 - Rust 在结尾的 `}` 处自动调用 `drop`

```rust
{
	let s = String::from("hello"); // 从此处起，s 是有效的

	// 使用 s
}                                  // 此作用域已结束，
								   // s 不再有效
```

#### 变量与数据交互的方式（一）：移动

在 Rust 中，多个变量可以采取不同的方式与**同一数据**进行交互。

```rust
    let x = 5;
    let y = x;
```

> 两个 `5` 被放入了栈中

```rust
let s1 = String::from("hello"); 
let s2 = s1;
```

当我们将 `s1` 赋值给 `s2`，`String` 的数据被复制了，这意味着我们从栈上拷贝了它的指针、长度和容量。我们==并没有复制指针指向的堆上数据==

![300](assets/Pasted%20image%2020240420112724.png)

这就有了一个问题：当 `s2` 和 `s1` 离开作用域，它们都会尝试释放相同的内存。这是一个叫做 **二次释放**（_double free_）的错误

为了确保内存安全，在 `let s2 = s1;` 之后，Rust 认为 `s1` 不再有效，因此 Rust 不需要在 `s1` 离开作用域后清理任何东西，这个操作被称为 **移动**（_move_），`s1` 被 **移动** 到了 `s2` 中

```rust
error[E0382]: borrow of moved value: `s1`
 --> src/main.rs:5:28
  |
2 |     let s1 = String::from("hello");
  |         -- move occurs because `s1` has type `String`, which does not implement the `Copy` trait
3 |     let s2 = s1;
  |              -- value moved here
4 |
5 |     println!("{}, world!", s1);
  |                            ^^ value borrowed here after move
```

![300](assets/Pasted%20image%2020240420113631.png)

#### 变量与数据交互的方式（二）：克隆

如果我们 确实 需要**深度复制** `String` 中堆上的数据，而不仅仅是栈上的数据，可以使用一个叫做 `clone` 的通用函数

```rust
    let s1 = String::from("hello");
    let s2 = s1.clone();

    println!("s1 = {}, s2 = {}", s1, s2);
```

#### 只在栈上的数据：拷贝

```rust
    let x = 5;
    let y = x;

    println!("x = {}, y = {}", x, y);
```

没有调用 `clone`，不过 `x` 依然有效且没有被移动到 `y` 中。原因是像整型这样的在编译时已知大小的类型被整个存储在栈上

	Rust 有一个叫做 `Copy` trait 的特殊注解，可以用在类似整型这样的存储在栈上的类型上，如果一个类型实现了 `Copy` trait，那么一个旧的变量在将其赋值给其他变量后仍然可用。

实现了 `Copy` trait 的类型

- 所有整数类型，比如 `u32`。
- 布尔类型，`bool`，它的值是 `true` 和 `false`。
- 所有浮点数类型，比如 `f64`。
- 字符类型，`char`。
- 元组，当且仅当其包含的类型也**都实现** `Copy` 的时候。比如，`(i32, i32)` 实现了 `Copy`，但 `(i32, String)` 就没有。

### 所有权与函数

#### 向函数传递值

将值传递给函数与给变量赋值的原理相似。向函数传递值可能会**移动**或者**复制**，就像赋值语句一样

```rust
fn main() {
    let s = String::from("hello"); // s 进入作用域

    takes_ownership(s); // s 的值移动到函数里 ...
                        // ... 所以s到这里不再有效
                        // println!("s: {s}");

    let x = 5; // x 进入作用域

    makes_copy(x); // x 应该移动函数里，
                   // 但 i32 是 Copy 的，
                   // 所以在后面可继续使用 x
    println!("x: {x}");
}   // 这里，x 先移出了作用域，然后是 s。但因为 s 的值已被移走，
    // 没有特殊之处

fn takes_ownership(some_string: String) {
    // some_string 进入作用域
    println!("{}", some_string);
}   // 这里，some_string 移出作用域并调用 `drop` 方法。
    // 占用的内存被释放

fn makes_copy(some_integer: i32) {
    // some_integer 进入作用域
    println!("{}", some_integer);
}   // 这里，some_integer 移出作用域。没有特殊之处

```

#### 返回值与作用域

返回值也可以**转移**所有权

```rust
fn main() {
    let s1 = gives_ownership(); // gives_ownership 将返回值
                                // 转移给 s1

    let s2 = String::from("hello"); // s2 进入作用域

    let s3 = takes_and_gives_back(s2); // s2 被移动到
                                       // takes_and_gives_back 中，
                                       // 它也将返回值移给 s3
} // 这里，s3 移出作用域并被丢弃。
  // s2 也移出作用域，但已被移动，所以什么也不会发生。
  // s1 离开作用域并被丢弃

fn gives_ownership() -> String {
    // gives_ownership 会将
    // 返回值移动给
    // 调用它的函数

    let some_string = String::from("yours"); // some_string 进入作用域。

    some_string // 返回 some_string
                // 并移出给调用的函数

}

// takes_and_gives_back 将传入字符串并返回该值
fn takes_and_gives_back(a_string: String) -> String {
    // a_string 进入作用域

    a_string // 返回 a_string 并移出给调用的函数
}
```

每一个函数中都获取所有权并接着返回所有权有些啰嗦。

- 如果我们想要函数使用一个值但**不获取所有权**该怎么办呢？
- 如果我们还要**接着使用**它的话，每次都传进去再返回来就有点烦人
- 除此之外，我们也可能想返回函数体中产生的一些数据。

我们可以使用元组来返回多个值

```rust
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    println!("The length of '{s2}' is {len}.");
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() 返回字符串的长度

    (s, length)
}
``` 

但是这未免有些形式主义，而且这种场景应该很常见。幸运的是，Rust 对此提供了一个不用获取所有权就可以使用值的功能，叫做 **引用**（_references_）

## 引用与借用

**引用**（_reference_）

- 像一个指针，因为它是一个**地址**，我们可以由此访问储存于该地址的属于其他变量的数据。 
- 与指针不同，引用确保指向某个**特定类型**的有效值。
- **不用获取所有权**就可以使用值

> 改造上面用元组接受返回值的代码，以一个对象的引用作为参数而不是获取值的所有权

```rust
fn main() {
    let s1 = String::from("hello");

    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

// 获取 `&String` 而不是 `String`
fn calculate_length(s: &String) -> usize {
    s.len()
}  // 这里，s 离开了作用域。但因为它并不拥有引用值的所有权， 
   // 所以什么也不会发生
```

`&String s` 指向 `String s1` 如下

![500](assets/Pasted%20image%2020240420163458.png)

`&s1` 语法让我们创建一个 **指向** 值 `s1` 的引用，但是并不拥有它。因为并不拥有这个值，所以当引用停止使用时，它所指向的值也**不会被丢弃**。

---

将创建一个引用的行为称为 **借用**（_borrowing_）

- 正如变量默认是不可变的，引用也一样。（默认）**不允许**修改引用的值。

```rust
fn main() {
    let s = String::from("hello");

    change(&s);
}

fn change(some_string: &String) {
    some_string.push_str(", world");
}
```

以上代码编译会出错

```shell
error[E0596]: cannot borrow `*some_string` as mutable, as it is behind a `&` reference
 --> src\main.rs:8:5
  |
8 |     some_string.push_str(", world");
  |     ^^^^^^^^^^^ `some_string` is a `&` reference, so the data it refers to cannot be borrowed as mutable
  |
help: consider changing this to be a mutable reference
  |
7 | fn change(some_string: &mut String) {
  |                         +++
```

### 可变引用

```rust
fn main() {
	//将 `s` 改为 `mut`
    let mut s = String::from("hello");
	
    change(&mut s);
}

fn change(some_string: &mut String) { //可变引用 `&mut s`
    some_string.push_str(", world");
}
```

### 引用的规则

1. 在任意给定时间，**要么** 只能有一个“可变引用”，**要么** 只能有一个或多个“不可变引用”
2. 引用必须总是有效的。（悬垂引用、引用作用域）

> #Bo 这里的时间可以按作用域理解

好处：是 Rust 可以在编译时就避免数据竞争

> **数据竞争**类似于竞态条件，它可由这三个行为造成：
> - 两个或更多指针同时访问同一数据。
> - 至少有一个指针被用来写入数据。
> - 没有同步数据访问的机制。

> 限制1演示

```rust
    let mut s = String::from("hello");

	// 不能在同一时间多次将 `s` 作为可变变量借用
    let r1 = &mut s;
    let r2 = &mut s;

    println!("{}, {}", r1, r2);
```

```shell
error[E0499]: cannot borrow `s` as mutable more than once at a time
 --> src/main.rs:5:14
  |
4 |     let r1 = &mut s;
  |              ------ first mutable borrow occurs here
5 |     let r2 = &mut s;
  |              ^^^^^^ second mutable borrow occurs here
6 |
7 |     println!("{}, {}", r1, r2);
  |                        -- first borrow later used here
```

可以使用大括号来创建一个新的作用域，以允许拥有多个可变引用，只是不能 **同时** 拥有：

```rust
    let mut s = String::from("hello");

    {
        let r1 = &mut s;
    } // r1 在这里离开了作用域，所以我们完全可以创建一个新的引用

    let r2 = &mut s;
```

> 限制 2 演示

```rust
    let mut s = String::from("hello");

    let r1 = &s; // 多个不可变引用是可以的
    let r2 = &s; // 多个不可变引用是可以的
    let r3 = &mut s; // 大问题

    println!("{}, {}, and {}", r1, r2, r3);
```

### 引用的作用域

一个引用的作用域从声明的地方开始一直持续到**最后一次使用**为止

```rust
    let mut s = String::from("hello");

    let r1 = &s; // 没问题
    let r2 = &s; // 没问题
    println!("{} and {}", r1, r2);
    // 此位置之后 r1 和 r2 不再使用

    let r3 = &mut s; // 没问题
    println!("{}", r3);
```

> 不可变引用 `r1` 和 `r2` 的作用域在 `println!` 最后一次使用之后结束，这也是创建可变引用 `r3` 的地方。它们的作用域没有重叠，所以代码是可以编译的

### 悬垂引用（Dangling References）

> 在具有指针的语言中，很容易通过释放内存时保留指向它的指针而错误地生成一个 **悬垂指针**（_dangling pointer_），所谓悬垂指针是==其指向的内存可能已经被分配给其它持有者==。

Rust 中编译器确保引用**永远也不会**变成悬垂状态：当你拥有一些数据的引用，编译器确保数据**不会**在其引用之前离开作用域。

> 尝试创建一个悬垂引用

```rust
fn main() {
    let reference_to_nothing = dangle();
}

fn dangle() -> &String {
    let s = String::from("hello");

    &s
}
```

```shell
error[E0106]: missing lifetime specifier
 --> src/main.rs:5:16
  |
5 | fn dangle() -> &String {
  |                ^ expected named lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but there is no value for it to be borrowed from
help: consider using the `'static` lifetime
  |
5 | fn dangle() -> &'static String {
  |                 +++++++
```

> 错误信息引用了一个我们还未介绍的功能：生命周期（lifetimes）

## slice（切片）类型

_slice_ 允许你引用**集合**中一段连续的元素序列，而不用引用整个集合。

- slice 是一种**引用**，没有所有权

> 设计一个函数 first_word，功能是返回在字符串中找到的第一个单词

```rust
fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s); // word 的值为 5

    s.clear(); // 这清空了字符串，使其等于 ""

    // word 在此处的值仍然是 5，
    // 但是没有更多的字符串让我们可以有效地应用数值 5。word 的值现在完全无效！
}

// 返回在该字符串中找到的第一个单词
fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }

    s.len()
}
```

这个程序==编译时没有任何错误==，而且在调用 `s.clear()` 之后使用 `word` 也不会出错。因为 `word` 与 `s` 状态完全没有联系

但我们不得不时刻担心 `word` 的索引与 `s` 中的数据不再同步，这很啰嗦且易出错！

如果编写这么一个 `second_word` 函数的话，管理索引这件事将更加容易出问题

```rust
fn second_word(s: &String) -> (usize, usize) {
```

现在我们要跟踪一个开始索引 **和** 一个结尾索引，同时有了更多从数据的某个特定状态计算而来的值，但都完全没有与这个状态相关联。现在有三个飘忽不定的不相关变量需要保持同步。

幸运的是，Rust 为这个问题提供了一个解决方法：字符串 slice。

### 字符串 slice（切片）

**字符串 slice**（_string slice_）是 `String` 中一部分值的**引用**

- 变量的类型：`&str`
- 形式：`[开始索引..结束索引]`
	- 开始索引就是切片起始位置的索引值
	- 结束索引是切片终止位置的**下一个**索引值
- slice 的数据结构存储了 slice 的开始位置和长度
	- 长度对应于 开始索引 减去 开始索引 的值。

```rust
    let s = String::from("hello world");

    let hello = &s[0..5];
    let world = &s[6..11];
```

> 对于 `let world = &s[6..11];` 的情况，`world` 是一个包含指向 `s` 索引 6 的指针和长度值 5 的 slice。

![300](assets/Pasted%20image%2020240420215124.png)

简略语法：

- 对于 Rust 的 `..` range 语法，如果想要从索引 0 开始，可以不写两个点号之前的值
- 如果 slice 包含 `String` 的最后一个字节，也可以舍弃尾部的数字
- 也可以同时舍弃这两个值来获取整个字符串的 slice

```rust
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];

let slice = &s[3..len]; 
let slice = &s[3..];

let slice = &s[0..len]; 
let slice = &s[..];
```

> 重写 `first_word` 来返回一个 slice

```rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

---

**编译器会确保指向 `String` 的引用持续有效。**

> 如下使用 slice 版本的 `first_word` 会抛出一个编译时错误：

```rust
fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s);

    s.clear(); // 错误！

    println!("the first word is: {}", word);
}
```

当拥有某值的不可变引用时，就不能再获取一个可变引用。

- `clear` 需要清空 `String`，它尝试获取一个可变引用。
- `println!` 使用了 `word` 中的引用，所以这个不可变的引用在此时必须仍然有效。
- Rust 不允许 `clear` 中的可变引用和 `word` 中的不可变引用同时存在，因此编译失败。

```shell
error[E0502]: cannot borrow `s` as mutable because it is also borrowed as immutable
  --> src/main.rs:18:5
   |
16 |     let word = first_word(&s);
   |                           -- immutable borrow occurs here
17 |
18 |     s.clear(); // error!
   |     ^^^^^^^^^ mutable borrow occurs here
19 |
20 |     println!("the first word is: {}", word);
   |                                       ---- immutable borrow later used here
```

#### 字符串字面值就是 slice

字符串字面值被直接存储在二进制程序中

**`&str` 是一个不可变引用**

```rust
let s = "Hello, world!";
```

> 这里 `s` 的类型是 `&str`：它是一个指向==二进制程序特定位置==的 slice。这也就是为什么字符串字面值是不可变的；

#### 字符串 slice 作为参数

```rust
fn first_word(s: &String) -> &str {
```

有经验的 Rustacean 会编写出：

```rust
// 将 `s` 参数的类型改为字符串 slice
fn first_word(s: &str) -> &str {
```

- 因为它使得可以对 `&String` 值和 `&str` 值使用相同的函数，使得我们的 API 更加**通用**并且**不会丢失**任何功能
	- 如果有一个**字符串 slice**，可以直接传递它。
	- 如果有一个**字符串字面值**，可以直接传递它
	- 如果有一个 `String`，则可以传递
		- 整个 `String` 的 slice 
		- 或==对 `String` 的引用==

> 这种灵活性利用了 _deref coercions_ 的优势，这个特性我们将在[“函数和方法的隐式 Deref 强制转换”](https://kaisery.github.io/trpl-zh-cn/ch15-02-deref.html#%E5%87%BD%E6%95%B0%E5%92%8C%E6%96%B9%E6%B3%95%E7%9A%84%E9%9A%90%E5%BC%8F-deref-%E5%BC%BA%E5%88%B6%E8%BD%AC%E6%8D%A2)章节中介绍。

```rust
fn main() {
    let my_string = String::from("hello world");

    // `first_word` 适用于 `String`（的 slice），部分或全部
    let word = first_word(&my_string[0..6]);
    let word = first_word(&my_string[..]);
    // `first_word` 也适用于 `String` 的引用，
    // 这等价于整个 `String` 的 slice
    let word = first_word(&my_string);

    let my_string_literal = "hello world";

    // `first_word` 适用于字符串字面值，部分或全部
    let word = first_word(&my_string_literal[0..6]);
    let word = first_word(&my_string_literal[..]);

    // 因为字符串字面值已经是 字符串 slice 了，
    // 这也是适用的，无需 slice 语法！
    let word = first_word(my_string_literal);
}

fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}
```

### 其他类型的 slice

字符串 slice 是针对字符串的，也有更通用的 slice 类型，比如数组：

```rust
let a = [1, 2, 3, 4, 5];  
let slice = &a[1..3];  
assert_eq!(slice, &[2, 3]);`
```

这个 slice 的类型是 `&[i32]`。它跟字符串 slice 的工作方式一样，通过存储第一个集合元素的引用和一个集合总长度。你可以对其他所有集合使用这类 slice。

# ----- 结构体

_struct_，或者 _structure_，是一个**自定义数据类型**，允许你包装和命名多个相关的值，从而形成一个有意义的组合

## 定义和实例化

### 基本使用

和元组的区别

- 结构体的每一部分可以是不同类型（和元组一样）
- 结构体需要命名各部分数据（不同于元组）
- 结构体比元组更灵活：不需要依赖顺序来指定或访问实例中的值。

定义结构体，

- 需要使用 `struct` 关键字并为整个结构体提供一个名字。
- 在大括号中，定义每一部分数据的**名字**和**类型**，我们称为 **字段**（_field_）

```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```

创建结构体的 **实例**
- 以结构体的名字开头
-  `key: value` 键 - 值对的形式提供字段

```rust
fn main() {
    let user1 = User {
        active: true,
        username: String::from("someusername123"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
    };
}
```

从结构体中获取某个特定的值
- 使用点号

如果结构体的**实例是可变的**，可以使用点号并为对应的字段**赋值**

- Rust 并**不允许**只将某个字段标记为可变

```rust
fn main() {
	// user1是可变的
    let mut user1 = User {
        active: true,
        username: String::from("someusername123"),
        email: String::from("someone@example.com"),
        sign_in_count: 1,
    };

    user1.email = String::from("anotheremail@example.com");
}
```

以在函数体的最后一个表达式中构造一个结构体的新实例，来隐式地返回这个实例

```rust
fn build_user(email: String, username: String) -> User {
	// User实例
    User {
        active: true,
        username: username,
        email: email,
        sign_in_count: 1,
    }
}
```

### 字段初始化简写语法

无需重复 `username` 和 `email` 了

```rust
fn build_user(email: String, username: String) -> User {
    User {
        active: true,
        username, //字段初始化简写语法
        email, //字段初始化简写语法
        sign_in_count: 1,
    }
}
```

### 结构体更新语法

使用**旧实例**的大部分值但**改变其部分值**来创建一个新的结构体实例通常是很有用的。这可以通过 **结构体更新语法**（_struct update syntax_）实现。

> 不使用更新语法时的写法

```rust
fn main() {
    // --snip--

    let user2 = User {
        active: user1.active,
        username: user1.username,
        email: String::from("another@example.com"),
        sign_in_count: user1.sign_in_count,
    };
}
```

使用结构体更新语法
- `..` 语法指定了剩余未显式设置值的字段，应有与给定实例对应字段相同的值
- `..` 必须放最后

```rust
fn main() {
    // --snip--

    let user2 = User {
        email: String::from("another@example.com"),
        ..user1
    };
}
```

结构更新语法就像带有 `=` 的赋值，因为它**移动**了数据
- `user1` 的 `username` 字段中的 `String` 被移到 `user2` 中

```rust
fn main() {
    let user1 = User {
        active: true,
        username: String::from("user1"),
        email: String::from("116@qq.com"),
        sign_in_count: 32,
    };

    let user2 = User {
        email: String::from("another@example.com"),
        ..user1
    };

    println!("{}",user1.active);
    println!("{}",user1.email);
    println!("{}",user1.username); // 出错
    println!("{}",user1.sign_in_count);
}
```

### 元组结构体

元组结构体有着结构体名称提供的含义，但**没有**具体的字段名，**只有**字段的类型。
- 适用场景：当你想给整个元组取一个名字，并使元组成为与其他元组不同的类型时

定义元组结构体
- 以 `struct` 关键字和结构体名开头并后跟元组中的类型

```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
}
```

注意 `black` 和 `origin` 值的**类型不同**：定义的每一个结构体有其自己的类型，即使结构体中的字段可能有着相同的类型

在其他方面，元组结构体实例**类似于**元组：可以将它们**解构**为单独的部分，也可以使用 `.` 后跟**索引**来访问单独的值，等等。

### 类单元结构体

**类单元结构体**（_unit-like structs_）没有任何字段
- 它们类似于 `()`，即[“元组类型”](https://kaisery.github.io/trpl-zh-cn/ch03-02-data-types.html#%E5%85%83%E7%BB%84%E7%B1%BB%E5%9E%8B)中的 unit 类型。
- 使用场景：想要在某个类型上实现 trait，但不需要在类型中存储数据的时候。

```rust
struct AlwaysEqual;

fn main() {
    let subject = AlwaysEqual;
}
```

### 结构体数据的所有权

在以上示例中的 `User` 结构体的定义中，我们使用了自身拥有所有权的 `String` 类型而不是 `&str` 字符串 slice 类型。这是一个有意而为之的选择，因为我们想要这个结构体拥有它所有的数据，为此只要整个结构体是有效的话其数据也是有效的。

```rust
fn main() {
    let str = String::from("hello");

    let user1 = User {
        active: true,
        username: str,
        email: String::from("116@qq.com"),
        sign_in_count: 32,
    };

    println!("{}", user1.username);
    println!("{}", str); // 报错，str被移动了
}
```

可以使结构体存储被其他对象拥有的数据的引用，不过这么做的话需要用上 **生命周期**（_lifetimes_）。生命周期确保结构体引用的数据有效性跟结构体本身保持一致。

- 如果你尝试在结构体中存储一个引用而不指定生命周期将是**无效的**

```rust
struct User {
    active: bool,
    username: &str,
    email: &str,
    sign_in_count: u64,
}

fn main() {
    let user1 = User {
        active: true,
        username: "someusername123",
        email: "someone@example.com",
        sign_in_count: 1,
    };
}
```

```shell
error[E0106]: missing lifetime specifier
 --> src/main.rs:3:15
  |
3 |     username: &str,
  |               ^ expected named lifetime parameter
  |
help: consider introducing a named lifetime parameter
  |
1 ~ struct User<'a> {
2 |     active: bool,
3 ~     username: &'a str,
  |

error[E0106]: missing lifetime specifier
 --> src/main.rs:4:12
  |
4 |     email: &str,
  |            ^ expected named lifetime parameter
  |
help: consider introducing a named lifetime parameter
  |
1 ~ struct User<'a> {
2 |     active: bool,
3 |     username: &str,
4 ~     email: &'a str,
  |
```

> 现在，我们会使用像 `String` 这类拥有所有权的类型来替代 `&str` 这样的引用以修正这个错误。

## 结构体示例程序

需求：获取以像素为单位的长方形的宽度和高度，并计算出长方形的面积

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        area(&rect1)
    );
}

fn area(rectangle: &Rectangle) -> u32 {
    rectangle.width * rectangle.height
}
```

- 使用 `Rectangle` 的 `width` 和 `height` 字段，计算 `Rectangle` 的面积。这表明宽高是相互联系的，并为这些值提供了描述性的名称而不是使用元组的索引值 `0` 和 `1` 。结构体胜在更清晰明了。

### 通过派生 trait 增加实用功能

在调试程序时打印出 `Rectangle` 实例来查看其**所有字段**的值非常有用

1）println! 宏

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!("rect1 is {:#?}", rect1);
}
```

2）`dbg!` 宏接收一个表达式的所有权（`println!` 宏接收的是引用），打印出代码中调用 `dbg!` 宏时所在的文件和行号，以及该表达式的结果值，并**返回**该值的所有权。

#todo 

## 方法语法

**方法**（method）与函数类似：它们使用 `fn` 关键字和名称声明，可以拥有参数和返回值，同时包含在某处调用该方法时会执行的代码。

不过方法与函数是不同的，因为

- 它们在结构体的上下文中被定义（或者是枚举 或 trait 对象的上下文）
- 并且它们**第一个参数**总是 `self`，它代表调用该方法的**结构体实例**

### 定义方法

为了使函数定义于 `Rectangle` 的上下文中，

- 我们开始了一个 `impl` 块（`impl` 是 _implementation_ 的缩写），这个 `impl` 块中的所有内容都将与 `Rectangle` 类型相关联。
- 接着将 `area` 函数移动到 `impl` 大括号中，并将签名中的第一个参数和函数体中其他地方的对应参数改成 `self`。
- 然后在 `main` 中使用 **方法语法**（_method syntax_）在 `Rectangle` 实例上调用 `area` 方法。
	- 方法语法获取一个实例并加上一个点号，后跟方法名、圆括号以及任何参数。

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        rect1.area()
    );
}
```

`&self` 实际上是 `self: &Self` 的缩写。在一个 `impl` 块中，`Self` 类型是 `impl` 块的**类型的别名**

- 方法可以选择获得 `self` 的**所有权**，
	- 很少见，通常用在当方法将 `self` 转换成别的实例的时候，这时我们想要防止调用者在转换之后使用原始的实例。
- 或者像我们这里一样**不可变地借用** `self`（`&self`）
- 或者**可变地借用** `self`（`&mut self`），就跟其他参数一样

使用方法替代函数，

- 除了可使用方法语法和不需要在每个函数签名中重复 `self` 的类型之外，
- 其主要好处在于**组织性**。我们将某个类型实例能做的所有事情都一起放入 `impl` 块中，而不是让将来的用户在我们的库中到处寻找 `Rectangle` 的功能。

getter方法：可以选择将方法的名称与结构中的一个字段相同

- 与字段同名的方法将被定义为==只返回字段中的值，而不做其他事情==。这样的方法被称为 _getters_，
- Rust 并**不像**其他一些语言那样为结构字段自动实现它们
- Getters 很有用，因为你可以把字段变成**私有的**，但方法是**公共的**

```rust
impl Rectangle {
	// 方法名 width，字段名 width
    fn width(&self) -> bool {
        self.width > 0
    }
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    if rect1.width() {
        println!("The rectangle has a nonzero width; it is {}", rect1.width);
    }
}
```

### 自动引用和解引用

> 在 C/C++ 语言中，有两个不同的运算符来调用方法：`.` 直接在对象上调用方法，而 `->` 在一个对象的指针上调用方法，这时需要先解引用（dereference）指针。
> 
> 换句话说，如果 `object` 是一个指针，那么 `object->something()` 就像 `(*object).something()` 一样。

Rust 并没有一个与 `->` 等效的运算符；相反，Rust 有一个叫 **自动引用和解引用**（_automatic referencing and dereferencing_）的功能。方法调用是 Rust 中少数几个拥有这种行为的地方。

它是这样工作的：当使用 `object.something()` 调用方法时，Rust 会**自动**为 `object` 添加 `&`、`&mut` 或 `*` 以便==使 object 与方法签名匹配==。也就是说，这些代码是等价的：

```rust
p1.distance(&p2); 
(&p1).distance(&p2);
```

第一行看起来简洁的多。这种自动引用的行为之所以有效，是因为方法有一个明确的接收者———— `self` 的类型。在给出接收者和方法名的前提下，Rust 可以明确地计算出方法是仅仅读取（`&self`），做出修改（`&mut self`）或者是获取所有权（`self`）。

事实上，Rust 对方法接收者的隐式借用让所有权在实践中更友好。

### 带有更多参数的方法

方法实现的功能：让一个 `Rectangle` 的实例获取另一个 `Rectangle` 实例，如果 `self` （第一个 `Rectangle`）能完全包含第二个长方形则返回 `true`；否则返回 `false`

```rust
fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };
    let rect2 = Rectangle {
        width: 10,
        height: 40,
    };
    let rect3 = Rectangle {
        width: 60,
        height: 45,
    };

    println!("Can rect1 hold rect2? {}", rect1.can_hold(&rect2));
    println!("Can rect1 hold rect3? {}", rect1.can_hold(&rect3));
}
```

获取另一个 `Rectangle` 的不可变引用作为参数，希望 `main` 保持 `rect2` 的所有权，这样就可以在调用这个方法后继续使用它

```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }

    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

### 关联函数

所有在 `impl` 块中定义的函数被称为 **关联函数**（_associated functions_），因为它们与 `impl` 后面命名的类型相关。

- *形式*：不以 `self` 为第一参数的关联函数（因此不是方法），因为它们并不作用于一个结构体的实例。
	- 例如在 `String` 类型上定义的 `String::from` 函数。
- *使用场景*：关联函数经常被用作**返回**一个结构体新实例的**构造函数**。这些函数的名称通常为 `new` ，但 `new` 并不是一个关键字。

> 例如我们可以提供一个叫做 `square` 关联函数，它接受一个维度参数并且同时作为宽和高

```rust
impl Rectangle {
	// 构造一个正方形
    fn square(size: u32) -> Self {
        Self {
            width: size,
            height: size,
        }
    }
}
```

- **关键字** `Self` 在函数的返回类型中代指在 `impl` 关键字后出现的类型

使用结构体名和 `::` 语法来调用这个关联函数

- 这个函数位于结构体的**命名空间**中

> `::` 语法用于关联函数和模块创建的命名空间

```rust
let sq = Rectangle::square(3);
```

### 多个 `impl` 块

每个结构体都允许拥有多个 `impl` 块，但每个方法有其自己的 `impl` 块。

```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}

impl Rectangle {
    fn can_hold(&self, other: &Rectangle) -> bool {
        self.width > other.width && self.height > other.height
    }
}
```

这里没有理由将这些方法分散在多个 `impl` 块中，不过这是有效的语法。

# ----- 枚举和模式匹配

**枚举**（_enumerations_），也被称作 _enums_。

枚举允许你通过列举可能的 **成员**（_variants_）来**定义一个类型**。

## 枚举的定义

> 结构体给予你将字段和数据聚合在一起的方法，像 `Rectangle` 结构体有 `width` 和 `height` 两个字段。
> 
> 而枚举给予你一个途径去声明某个值是一个集合中的一员。比如，我们想让 `Rectangle` 是一些形状的集合，包含 `Circle` 和 `Triangle` 。为了做到这个，Rust 提供了枚举类型。

> 任何一个 IP 地址要么是 IPv4 的要么是 IPv6 的，而且不能两者都是。IP 地址的这个特性使得枚举数据结构非常适合这个场景，因为枚举值只可能是其中一个成员。
> 
> IPv4 和 IPv6 从根本上讲仍是 IP 地址，所以当代码在处理适用于任何类型的 IP 地址的场景时应该把它们当作相同的类型。

定义一个 `IpAddrKind` 枚举，`V4` 和 `V6` 被称为枚举的 **成员**（_variants_）

- `IpAddrKind` 就是一个可以在代码中使用的**自定义数据类型**了

```rust
enum IpAddrKind {
    V4,
    V6,
}
```

创建 `IpAddrKind` 两个不同成员的实例：

```rust
    let four = IpAddrKind::V4;
    let six = IpAddrKind::V6;
```

- 枚举的成员位于其标识符的**命名空间**中，并使用两个冒号分开
	- 益处是现在 `IpAddrKind::V4` 和 `IpAddrKind::V6` 都是 `IpAddrKind` **类型**的
	- 可以定义一个函数来获取任何 `IpAddrKind`，可以使用任一成员来调用这个函数

```rust
fn route(ip_kind: IpAddrKind) {}

fn main() {
    route(IpAddrKind::V4);
    route(IpAddrKind::V6);
}
```

### [枚举值](https://kaisery.github.io/trpl-zh-cn/ch06-01-defining-an-enum.html#%E6%9E%9A%E4%B8%BE%E5%80%BC)

将“枚举成员”与“值”相关联

1）将枚举作为结构体的一部分

```rust
    enum IpAddrKind {
        V4,
        V6,
    }

    struct IpAddr {
        kind: IpAddrKind,
        address: String,
    }

    let home = IpAddr {
        kind: IpAddrKind::V4,
        address: String::from("127.0.0.1"),
    };

    let loopback = IpAddr {
        kind: IpAddrKind::V6,
        address: String::from("::1"),
    };
```

2）将数据**直接**放进每一个枚举**成员**

- 定义的枚举成员的名字也变成了一个构建枚举的**实例**的**函数**
- 作为定义枚举的结果，这些构造函数会**自动**被定义。
- 好处
	- **直接**将数据附加到枚举的每个成员上，这样就不需要一个额外的结构体了
	- 每个成员可以处理**不同类型**和数量的数据

> `IpAddr::V4()` 是一个获取 `String` 参数并返回 `IpAddr` 类型实例的**函数调用**。

```rust
    enum IpAddr {
        V4(String),
        V6(String),
    }

    let home = IpAddr::V4(String::from("127.0.0.1"));

    let loopback = IpAddr::V6(String::from("::1"));
```

```rust
    enum IpAddr {
        V4(u8, u8, u8, u8),
        V6(String),
    }

    let home = IpAddr::V4(127, 0, 0, 1);

    let loopback = IpAddr::V6(String::from("::1"));
```

标准库是如何定义 `IpAddr` 的：将成员中的地址数据嵌入到了两个不同形式的结构体中，对不同的成员的定义是不同的

```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

>另外，标准库中的类型通常并不比你设想出来的要复杂多少。
> 
> 注意虽然标准库中包含一个 `IpAddr` 的定义，仍然可以创建和使用我们自己的定义而不会有冲突，因为我们并没有将标准库中的定义引入作用域

可以将**任意类型**的数据放入枚举成员中，例如字符串、数字类型或者结构体，甚至可以包含另一个枚举！

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

> 这个枚举有四个含有不同类型的成员：
> 
> - `Quit` 没有关联任何数据。
> - `Move` 类似结构体包含命名字段。
> - `Write` 包含单独一个 `String`。
> - `ChangeColor` 包含三个 `i32`。

### 在枚举上定义方法

以使用 `impl` 来为结构体定义方法那样，也可以在枚举上定义方法

```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

impl Message {
    fn call(&self) {
        // 在这里定义方法体
    }
}

fn main() {
    let m = Message::Write(String::from("hello"));
    m.call();
}
```

- 方法体使用了 `self` 来获取调用方法的值。这个例子中，创建了一个值为 `Message::Write(String::from("hello"))` 的变量 `m`，而且这就是当 `m.call()` 运行时 `call` 方法中的 `self` 的值

### [`Option` 枚举和其相对于空值的优势](https://kaisery.github.io/trpl-zh-cn/ch06-01-defining-an-enum.html#option-%E6%9E%9A%E4%B8%BE%E5%92%8C%E5%85%B6%E7%9B%B8%E5%AF%B9%E4%BA%8E%E7%A9%BA%E5%80%BC%E7%9A%84%E4%BC%98%E5%8A%BF)

> 例如，如果请求一个非空列表的第一项，会得到一个值，如果请求一个空的列表，就什么也不会得到。从类型系统的角度来表达这个概念就意味着编译器需要检查是否处理了所有应该处理的情况，这样就可以避免在其他编程语言中非常常见的 bug。
> 
> 编程语言的设计经常要考虑包含哪些功能，但考虑排除哪些功能也很重要。Rust 并没有很多其他语言中有的空值功能。**空值**（_Null_ ）是一个值，它代表没有值。在有空值的语言中，变量总是这两种状态之一：空值和非空值。
> 
> 空值的问题在于当你尝试像一个非空值那样使用一个空值，会出现某种形式的错误。因为空和非空的属性无处不在，非常容易出现这类错误。
> 
> 然而，空值尝试表达的概念仍然是有意义的：空值是一个因为某种原因目前无效或缺失的值。

Rust 并没有空值，不过它确实拥有一个可以编码存在或不存在概念的**枚举**。这个枚举是 `Option<T>`

```rust
enum Option<T> {
    None,
    Some(T),
}
```

- `Option<T>` 枚举已经被包含在了 prelude 之中，**不需要**将其显式引入作用域
- 它的成员也是如此，可以**不需要** `Option::` 前缀来直接使用 `Some` 和 `None`
- `<T>` 意味着 `Option` 枚举的 `Some` 成员可以包含任意类型的数据
- Rust 需要我们指定 `Option` 整体的类型，因为编译器只通过 `None` 值无法推断出 `Some` 成员保存的值的类型。

```rust
    let some_number = Some(5);
    let some_char = Some('e');

	let absent_number = None; //错误
    let absent_number: Option<i32> = None;
```

因为 `Option<T>` 和 `T`（这里 `T` 可以是任何类型）是不同的类型，编译器不允许像一个肯定有效的值那样使用 `Option<T>`

```rust
    let x: i8 = 5;
    let y: Option<i8> = Some(5);

    let sum = x + y;
```

在对 `Option<T>` 进行运算之前**必须**将其转换为 `T`。通常这能帮助我们捕获到空值最常见的问题之一：假设某值不为空但实际上为空的情况

- 为了拥有一个**可能为空**的值，你**必须**要显式的将其放入对应类型的 `Option<T>` 中。
- 当使用这个值时，**必须**明确的处理值为空的情况。
- 只要一个值不是 `Option<T>` 类型，你就 **可以** 安全的认定它的值不为空

## [`match` 控制流结构](https://kaisery.github.io/trpl-zh-cn/ch06-02-match.html#match-%E6%8E%A7%E5%88%B6%E6%B5%81%E7%BB%93%E6%9E%84)

`match` 是一个极为强大的**控制流运算符**，它允许我们将一个值与一系列的**模式**相比较，并根据相匹配的模式执行相应代码。

- “模式”可由字面值、变量、通配符和许多其他内容构成。
- `match` 的力量来源于**模式**的表现力以及**编译器检查**，它确保了所有可能的情况都得到处理。

> 示例 6-3：一个枚举和一个以枚举成员作为模式的 `match` 表达式

```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}

```

- 列出 `match` 关键字后跟一个**表达式**
	- 在这个例子中是 `coin` 的值
- 接下来是 `match` 的分支。一个分支有两个部分：一个模式和一些代码
	- `=>` 运算符将模式和将要运行的代码分开

### [绑定值的模式](https://kaisery.github.io/trpl-zh-cn/ch06-02-match.html#%E7%BB%91%E5%AE%9A%E5%80%BC%E7%9A%84%E6%A8%A1%E5%BC%8F)

匹配分支的另一个有用的功能：可以**绑定**匹配的模式的部分值。
- 也就是==从枚举成员中提取值==。

```rust
#[derive(Debug)] // 这样可以立刻看到州的名称
enum UsState {
    Alabama,
    Alaska,
    // --snip--
}

enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter(UsState), //`Quarter` 成员也存放了一个 `UsState` 值的 `Coin` 枚举
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter(state) => {
            println!("State quarter from {:?}!", state);
            25
        }
    }
}
```

在匹配 `Coin::Quarter` 成员的分支的模式中增加了一个叫做 `state` 的变量。

- 当匹配到 `Coin::Quarter` 时，变量 `state` 将会绑定 25 美分硬币所对应州的值

如果调用 `value_in_cents(Coin::Quarter(UsState::Alaska))`

- `coin` 将是 `Coin::Quarter(UsState::Alaska)`。
- 当将值与每个分支相比较时，没有分支会匹配，直到遇到 `Coin::Quarter(state)`。
- 这时，`state` 绑定的将会是值 `UsState::Alaska`。
- 接着就可以在 `println!` 表达式中使用这个绑定了，像这样就可以获取 `Coin` 枚举的 `Quarter` 成员中内部的州的值。

### [匹配是穷尽的](https://kaisery.github.io/trpl-zh-cn/ch06-02-match.html#%E5%8C%B9%E9%85%8D%E6%98%AF%E7%A9%B7%E5%B0%BD%E7%9A%84)

`match` 中的分支必须覆盖了所有的可能性。

```rust
    fn plus_one(x: Option<i32>) -> Option<i32> {
        match x {
            Some(i) => Some(i + 1),
        }
    }
```

我们没有处理 `None` 的情况，所以这些代码会造成一个 bug

```shell
error[E0004]: non-exhaustive patterns: `None` not covered
 --> src/main.rs:3:15
  |
3 |         match x {
  |               ^ pattern `None` not covered
  |
note: `Option<i32>` defined here
  = note: the matched value is of type `Option<i32>`
help: ensure that all possible cases are being handled by adding a match arm with a wildcard pattern or an explicit pattern as shown
  |
4 ~             Some(i) => Some(i + 1),
5 ~             None => todo!(),
  |
```

在这个 `Option<T>` 的例子中，Rust 防止我们忘记明确的处理 `None` 的情况，这让我们免于假设拥有一个实际上为空的值

```rust
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        Some(i) => Some(i + 1),
        None => todo!(),
    }
}
```

### [通配模式和 `_` 占位符](https://kaisery.github.io/trpl-zh-cn/ch06-02-match.html#%E9%80%9A%E9%85%8D%E6%A8%A1%E5%BC%8F%E5%92%8C-_-%E5%8D%A0%E4%BD%8D%E7%AC%A6)

1）如果我们希望对一些**特定的值**采取特殊操作，而对**其他的值**采取默认操作：

- 可以使用 other

```rust
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        other => move_player(other),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
    fn move_player(num_spaces: u8) {}
```

最后一个分支则涵盖了所有其他可能的值，模式是我们命名为 `other` 的一个**变量**。

- `other` 分支的代码通过将其传递给 `move_player` 函数来**使用这个变量**
- 这种**通配模式**满足了 `match` 必须被穷尽的要求
- 必须将通配分支放在**最后**

2）当我们不想使用通配模式获取的值时

- 请使用 `_` 。这是一个特殊的模式，可以匹配任意值而不绑定到该值。这告诉 Rust 我们**不会使用**这个值

> 当你掷出的值不是 3 或 7 的时候，你必须再次掷出。这种情况下我们不需要使用这个值

```rust
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        _ => reroll(),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
    fn reroll() {}
```

可以明确告诉 Rust 我们不会使用与前面模式不匹配的值，并且这种情况下我们**不想运行任何代码**

- 可以使用**单元值**作为 `_` 分支的代码

```rust
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        _ => (),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
```

## [`if let` 简洁控制流](https://kaisery.github.io/trpl-zh-cn/ch06-03-if-let.html#if-let-%E7%AE%80%E6%B4%81%E6%8E%A7%E5%88%B6%E6%B5%81)

`if let` 语法让我们以一种不那么冗长的方式，来处理**只匹配一个模式**的值而忽略其他模式的情况

> 示例 6-6：`match` 只关心当值为 `Some` 时执行代码

```rust
    let config_max = Some(3u8);
    match config_max {
        Some(max) => println!("The maximum is configured to be {}", max),
        _ => (),
    }
```

可以使用 `if let` 这种更短的方式编写
- 通过等号分隔的一个**模式**（`Some(max)`）和一个**表达式**（`config_max`）

```rust
    let config_max = Some(3u8);
    if let Some(max) = config_max {
        println!("The maximum is configured to be {}", max);
    }
```

可以在 `if let` 中包含一个 `else`。
- `else` 块中的代码与 `match` 表达式中的 `_` 分支块中的代码相同

```rust
let mut count = 0;

match coin {
	Coin::Quarter(state) => println!("State quarter from {:?}!", state),
	_ => count += 1,
}

// 简化版本
if let Coin::Quarter(state) = coin {
	println!("State quarter from {:?}!", state);
} else {
	count += 1;
}
```

# ----- 使用包、Crate 和模块管理不断增长的项目

Rust 有许多功能可以让你管理代码的组织，包括哪些内容可以被公开，哪些内容作为私有部分，以及程序每个作用域中的名字。这些功能，有时被统称为 “模块系统（the module system）”，包括：

- **包**（_Packages_）：Cargo 的一个功能，它允许你构建、测试和分享 crate。
- **Crates** ：一个模块的树形结构，它形成了库或二进制项目。
- **模块**（_Modules_）和 **use**：允许你控制作用域和路径的私有性。
- **路径**（_path_）：一个命名例如结构体、函数或模块等项的方式

## [包和 Crate](https://kaisery.github.io/trpl-zh-cn/ch07-01-packages-and-crates.html#%E5%8C%85%E5%92%8C-crate)

crate 是 Rust 在编译时**最小**的代码单位。

- 如果你用 `rustc` 而不是 `cargo` 来编译一个文件（第一章我们这么做过），编译器还是会将那个文件认作一个 crate。
- crate 可以包含**模块**，模块可以定义在其他文件，然后和 crate 一起编译

crate 有两种形式：二进制项和库。

- _二进制项_ 可以被编译为**可执行程序**，
	- 比如一个命令行程序或者一个服务器。
	- 它们必须有一个 `main` 函数来定义当程序被执行的时候所需要做的事情。
	- 目前我们所创建的 crate 都是二进制项。
- _库_ 并**没有** `main` 函数，它们也**不会**编译为可执行程序，
	- 它们提供一些诸如函数之类的东西，使其他项目也能使用这些东西。
		- 比如 [第二章](https://kaisery.github.io/trpl-zh-cn/ch02-00-guessing-game-tutorial.html#%E7%94%9F%E6%88%90%E4%B8%80%E4%B8%AA%E9%9A%8F%E6%9C%BA%E6%95%B0) 的 `rand` crate 就提供了生成随机数的东西。
	- 大多数时间 `Rustaceans` 说的 crate 指的都是库，这与其他编程语言中 library 概念一致。

_crate root_ 是一个源文件，Rust 编译器以它为起始点，并构成 crate 的**根模块**

_包_（_package_）是提供一系列功能的**一个或者多个** crate

- 一个包会包含一个 _Cargo.toml_ 文件，阐述如何去构建这些 crate。
- Cargo 就是一个包含构建你代码的二进制项的**包**，也包含这些二进制项所依赖的库。
- 其他项目也能用 Cargo 库来实现与 Cargo 命令行程序一样的逻辑。
- 包中可以包含**至多**一个*库 crate(library crate)*。包中可以包含**任意多个***二进制 crate(binary crate)*，但是**必须至少**包含一个 crate（无论是库的还是二进制的）。

创建一个包，输入命令 `cargo new my-project`

- Cargo 会给我们的包创建一个 _Cargo.toml_ 文件
- Cargo 遵循的约定
	- _src/main.rs_ 是一个与**包同名**的二进制 crate，且是 crate 根
	- _src/lib.rs_，是一个与**包同名**的库 crate，且是 crate 根
	- *crate 根文件* 将由 Cargo 传递给 `rustc` 来实际构建库或者二进制项目。
- 在此，我们有了一个只包含 _src/main.rs_ 的包，意味着它只含有一个名为 `my-project` 的二进制 crate。
- 如果一个包同时含有 _src/main.rs_ 和 _src/lib.rs_，则它有两个 crate：一个二进制的和一个库的，且名字都与包相同
- 通过将文件放在 _src/bin_ 目录下，一个包可以拥有多个二进制 crate
	- 每个 _src/bin_ 下的文件都会被编译成一个独立的二进制 crate。

```shell
$ cargo new my-project
     Created binary (application) `my-project` package
$ ls my-project
Cargo.toml
src
$ ls my-project/src
main.rs
```

## [定义模块来控制作用域与私有性](https://kaisery.github.io/trpl-zh-cn/ch07-02-defining-modules-to-control-scope-and-privacy.html#%E5%AE%9A%E4%B9%89%E6%A8%A1%E5%9D%97%E6%9D%A5%E6%8E%A7%E5%88%B6%E4%BD%9C%E7%94%A8%E5%9F%9F%E4%B8%8E%E7%A7%81%E6%9C%89%E6%80%A7)

### [模块小抄](https://kaisery.github.io/trpl-zh-cn/ch07-02-defining-modules-to-control-scope-and-privacy.html#%E6%A8%A1%E5%9D%97%E5%B0%8F%E6%8A%84)

这里我们提供一个简单的参考，用来解释模块、路径、`use` 关键词和 `pub` 关键词如何在编译器中工作，以及大部分开发者如何组织他们的代码。

- *从 crate 根节点开始*：当编译一个 crate，编译器首先在 crate 根文件（通常，对于一个库 crate 而言是 _src/lib.rs_，对于一个二进制 crate 而言是 _src/main.rs_）中寻找需要被编译的代码。
- *声明模块*：在 **crate 根文件**中，你可以声明一个新模块；
	- 比如，你用 `mod garden;` 声明了一个叫做 `garden` 的模块。编译器会在下列路径中寻找模块代码：
	    - 内联，在大括号中，当 `mod garden` 后方不是一个分号而是一个大括号
	    - 在文件 _src/garden.rs_
	    - 在文件 _src/garden/mod.rs_
- *声明子模块*：在**除了 crate 根节点以外**的其他文件中，你可以定义子模块。
	- 比如，你可能在 _src/garden.rs_ 中定义了 `mod vegetables;`。编译器会在**以父模块命名的目录**中寻找子模块代码：
	    - 内联，在大括号中，当 `mod vegetables` 后方不是一个分号而是一个大括号
	    - 在文件 _src/garden/vegetables.rs_
	    - 在文件 _src/garden/vegetables/mod.rs_
- *模块中的代码路径*：一旦一个模块是你 crate 的一部分，你可以在隐私规则允许的前提下，从同一个 crate 内的任意地方，**通过代码路径引用该模块的代码**。
	- 举例而言，一个 garden vegetables 模块下的 `Asparagus` 类型可以在 `crate::garden::vegetables::Asparagus` 被找到
- *私有 vs 公用*：
	- 一个模块里的代码默认对其父模块私有。
	- 为了使一个模块公用，应当在**声明时**使用 `pub mod` 替代 `mod`。
	- 为了使一个公用模块内部的成员公用，应当在**声明前**使用 `pub`。
- *use 关键字*：在一个作用域内，`use` 关键字创建了一个成员的快捷方式，用来==减少长路径的重复==。
	- 在任何可以引用 `crate::garden::vegetables::Asparagus` 的**作用域**，你可以通过 `use crate::garden::vegetables::Asparagus;` 创建一个快捷方式，
	- 然后你就可以在**作用域**中只写 `Asparagus` 来使用该类型

```
backyard
├── Cargo.lock
├── Cargo.toml
└── src
    ├── garden
    │   └── vegetables.rs
    ├── garden.rs
    └── main.rs
```

这个例子中的 crate 根文件是 _src/main.rs_：

```rust
use crate::garden::vegetables::Asparagus;

pub mod garden;

fn main() {
    let plant = Asparagus {};
    println!("I'm growing {:?}!", plant);
}
```

`pub mod garden;` 行告诉编译器应该包含在 _src/garden.rs_ 文件中发现的代码

```rust
pub mod vegetables;
```

意味着在 _src/garden/vegetables.rs_ 中的代码也应该被包括

```rust
#[derive(Debug)]
pub struct Asparagus {}
```

### [在模块中对相关代码进行分组](https://kaisery.github.io/trpl-zh-cn/ch07-02-defining-modules-to-control-scope-and-privacy.html#%E5%9C%A8%E6%A8%A1%E5%9D%97%E4%B8%AD%E5%AF%B9%E7%9B%B8%E5%85%B3%E4%BB%A3%E7%A0%81%E8%BF%9B%E8%A1%8C%E5%88%86%E7%BB%84)

_模块_ 让我们可以将一个 crate 中的代码进行分组，以提高可读性与重用性。

因为一个模块中的代码默认是私有的，所以还可以利用模块控制项的 _私有性_

- 私有项是不可为外部使用的内在详细实现。
- 我们也可以将模块和它其中的项标记为公开的，这样，外部代码就可以使用并依赖与它们。

> 在餐饮业，餐馆中会有一些地方被称之为 _前台_（_front of house_），还有另外一些地方被称之为 _后台_（_back of house_）。
> - 前台是招待顾客的地方，在这里，店主可以为顾客安排座位，服务员接受顾客下单和付款，调酒师会制作饮品。
> - 后台则是由厨师工作的厨房，洗碗工的工作地点，以及经理做行政工作的地方组成。

我们可以将函数放置到**嵌套的模块**中，来使我们的 crate 结构与实际的餐厅结构相同。

通过执行 `cargo new --lib restaurant`，来创建一个新的名为 `restaurant` 的库。在 _src/lib.rs_ 中，来定义一些模块和函数。

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn serve_order() {}

        fn take_payment() {}
    }
}
```

- 我们定义一个模块，是以 `mod` 关键字为起始，然后指定模块的名字（本例中叫做 `front_of_house`），并且用花括号包围模块的主体。
- 在模块内，我们还可以定义其他的模块，就像本例中的 `hosting` 和 `serving` 模块。
- 模块还可以保存一些定义的其他项，比如结构体、枚举、常量、特性、或者函数。

在前面我们提到了，`src/main.rs` 和 `src/lib.rs` 叫做 crate 根。之所以这样叫它们是因为这两个文件的内容都分别在 crate 模块结构的根组成了一个名为 `crate` 的模块，该结构被称为 _模块树_（_module tree_）

- 这个树展示了一些模块是如何被嵌入到另一个模块的
	- 例如，`hosting` 嵌套在 `front_of_house` 中
- 这个树还展示了一些模块是互为 _兄弟_（_siblings_）的，这意味着它们定义在同一模块中
	- `hosting` 和 `serving` 被一起定义在 `front_of_house` 中
- 如果一个模块 A 被包含在模块 B 中，我们将模块 A 称为模块 B 的 _子_（_child_），模块 B 则是模块 A 的 _父_（_parent_）
- 注意，整个模块树都植根于名为 `crate` 的隐式模块下。

```
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

## [引用模块项目的路径](https://kaisery.github.io/trpl-zh-cn/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#%E5%BC%95%E7%94%A8%E6%A8%A1%E5%9D%97%E9%A1%B9%E7%9B%AE%E7%9A%84%E8%B7%AF%E5%BE%84)

来看一下 Rust 如何在模块树中找到一个项的位置，我们使用路径的方式，就像在文件系统使用路径一样。为了调用一个函数，我们需要知道它的路径。

路径有两种形式：

- **绝对路径**（_absolute path_）是以 crate 根（root）开头的全路径；对于外部 crate 的代码，是以 crate 名开头的绝对路径，对于当前 crate 的代码，则以字面值 `crate` 开头。
- **相对路径**（_relative path_）从当前模块开始，以 `self`、`super` 或定义在当前模块中的标识符开头。

绝对路径和相对路径都后跟一个或多个由双冒号（`::`）分割的标识符。

```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // 绝对路径
    crate::front_of_house::hosting::add_to_waitlist();

    // 相对路径
    front_of_house::hosting::add_to_waitlist();
}
```

> 示例 7-3：使用绝对路径和相对路径来调用 `add_to_waitlist` 函数，文件名：*src/lib.rs*

选择使用相对路径还是绝对路径，要取决于你的项目，也取决于你是更倾向于将项的定义代码与使用该项的代码**分开来移动**，还是**一起移动**。

- 如果我们要将 `front_of_house` 模块和 `eat_at_restaurant` 函数一起移动到一个名为 `customer_experience` 的模块中，我们需要更新 `add_to_waitlist` 的绝对路径，但是相对路径还是可用的。
- 如果我们要将 `eat_at_restaurant` 函数单独移到一个名为 `dining` 的模块中，还是可以使用原本的绝对路径来调用 `add_to_waitlist`，但是相对路径必须要更新。
- **我们更倾向于使用绝对路径**，因为把“代码定义”和“项调用”各自独立地移动是更常见的。

### [使用 `pub` 关键字暴露路径](https://kaisery.github.io/trpl-zh-cn/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#%E4%BD%BF%E7%94%A8-pub-%E5%85%B3%E9%94%AE%E5%AD%97%E6%9A%B4%E9%9C%B2%E8%B7%AF%E5%BE%84)

试着编译一下示例 7-3，错误信息说 `hosting` 模块是私有的

```rust
error[E0603]: module `hosting` is private
  --> src/lib.rs:12:21
   |
12 |     front_of_house::hosting::add_to_waitlist();
   |                     ^^^^^^^ private module
   |
note: the module `hosting` is defined here
  --> src/lib.rs:2:5
   |
2  |     mod hosting {
   |     ^^^^^^^^^^^
```

**在 Rust 中，默认所有项（函数、方法、结构体、枚举、模块和常量）对父模块都是私有的**

- 父模块中的项不能使用子模块中的私有项，但是子模块中的项可以使用它们父模块中的项。这是因为子模块封装并隐藏了它们的实现详情

使用 `pub` 关键字来创建公共项，使子模块的内部部分暴露给上级模块

- **使模块公有并不使其内容也是公有的**。模块上的 `pub` 关键字只允许其父模块引用它，而不允许访问内部代码
- 需要更深入地选择将一个或多个项变为公有

```rust
mod front_of_house {
	// 公有模块
    pub mod hosting {
	    // 公有函数
        pub fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // 绝对路径
    crate::front_of_house::hosting::add_to_waitlist();

    // 相对路径
    front_of_house::hosting::add_to_waitlist();
}
```

> 示例 7-7: 为 `mod hosting` 和 `fn add_to_waitlist` 添加 `pub` 关键字使它们可以在 `eat_at_restaurant` 函数中被调用

### [二进制和库 crate 包的最佳实践](https://kaisery.github.io/trpl-zh-cn/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#%E4%BA%8C%E8%BF%9B%E5%88%B6%E5%92%8C%E5%BA%93-crate-%E5%8C%85%E7%9A%84%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5)

> 我们提到过包可以同时包含一个 _src/main.rs_ 二进制 crate 根和一个 _src/lib.rs_ 库 crate 根，并且这两个 crate 默认以包名来命名。

通常，这种包含二进制 crate 和库 crate 的模式的包，在二进制 crate 中只有足够的代码来启动一个可执行文件，可执行文件调用库 crate 的代码。又因为库 crate 可以共享，这使得其它项目从包提供的大部分功能中受益。

**模块树应该定义在 _src/lib.rs_ 中**。

- 这样通过以包名开头的路径，公有项就可以在二进制 crate 中使用。
- 二进制 crate 就完全变成了同其它外部 crate 一样的库 crate 的用户：它只能使用公有 API。
- 这有助于你设计一个好的 API；你不仅仅是作者，也是用户

### [`super` 开始的相对路径](https://kaisery.github.io/trpl-zh-cn/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#super-%E5%BC%80%E5%A7%8B%E7%9A%84%E7%9B%B8%E5%AF%B9%E8%B7%AF%E5%BE%84)

我们可以通过在路径的开头使用 `super` ，从父模块开始构建相对路径，而不是从当前模块或者 crate 根开始。

- 使用 `super` 允许我们引用父模块中的已知项，这使得重新组织模块树变得更容易 —— 当模块与父模块关联的很紧密（但某天父模块可能要移动到模块树的其它位置）

```rust
fn deliver_order() {}

mod back_of_house {
    fn fix_incorrect_order() {
        cook_order();
        super::deliver_order();
    }

    fn cook_order() {}
}
```

> 我们认为 `back_of_house` 模块和 `deliver_order` 函数之间可能具有某种关联关系，并且，如果我们要重新组织这个 crate 的模块树，需要一起移动它们。因此，我们使用 `super`，这样一来，如果这些代码被移动到了其他模块，我们只需要更新很少的代码。

### [创建公有的结构体和枚举](https://kaisery.github.io/trpl-zh-cn/ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#%E5%88%9B%E5%BB%BA%E5%85%AC%E6%9C%89%E7%9A%84%E7%BB%93%E6%9E%84%E4%BD%93%E5%92%8C%E6%9E%9A%E4%B8%BE)

我们还可以使用 `pub` 来设计公有的结构体和枚举

如果我们在一个结构体定义的前面使用了 `pub` ，

- 这个结构体会变成公有的，
- **但是这个结构体的字段仍然是私有的**。
- 我们可以根据情况决定每个字段是否公有。

```rust
mod back_of_house {
  pub struct Breakfast {
      pub toast: String,
      seasonal_fruit: String,
  }

  impl Breakfast {
	  // 公有的关联函数
      pub fn summer(toast: &str) -> Breakfast {
          Breakfast {
              toast: String::from(toast),
              seasonal_fruit: String::from("peaches"),
          }
      }
  }
}

pub fn eat_at_restaurant() {
  // 在夏天订购一个黑麦土司作为早餐
  let mut meal = back_of_house::Breakfast::summer("Rye");
  // 改变主意更换想要面包的类型
  meal.toast = String::from("Wheat");
  println!("I'd like {} toast please", meal.toast);

  // 不允许查看或修改早餐附带的季节水果
  // meal.seasonal_fruit = String::from("blueberries");
}
```

> 示例 7-9: 带有公有和私有字段的结构体，文件名：src/lib.rs

- 因为 `back_of_house::Breakfast` 具有私有字段，所以这个结构体需要提供一个公共的**关联函数**来构造 `Breakfast` 的实例 (这里我们命名为 `summer`)

---

如果我们将“枚举”设为公有，则它的**所有成员都将变为公有**
- 枚举成员默认就是公有的

```rust
mod back_of_house {
    pub enum Appetizer {
        Soup,
        Salad,
    }
}

pub fn eat_at_restaurant() {
    let order1 = back_of_house::Appetizer::Soup;
    let order2 = back_of_house::Appetizer::Salad;
}
```

## [使用 `use` 关键字将路径引入作用域](https://kaisery.github.io/trpl-zh-cn/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#%E4%BD%BF%E7%94%A8-use-%E5%85%B3%E9%94%AE%E5%AD%97%E5%B0%86%E8%B7%AF%E5%BE%84%E5%BC%95%E5%85%A5%E4%BD%9C%E7%94%A8%E5%9F%9F)

可以使用 `use` 关键字创建一个短路径，然后就可以在作用域中的任何地方使用这个更短的名字。
- 通过 `use` 引入作用域的路径也会检查私有性，同其它路径一样。

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```
> 示例 7-11

**`use` 只能创建 `use` 所在的特定作用域内的短路径**

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting;

mod customer {
    pub fn eat_at_restaurant() {
        hosting::add_to_waitlist();
    }
}
```
> 示例 7-12: 编译器错误显示短路径不再适用于 `customer` 模块中：

为了修复这个问题，两个办法
1. 将 `use` 移动到 `customer` 模块内
2. 在子模块 `customer` 内通过 `super::hosting` **引用父模块中的这个短路径**

```rust
// 将 `use` 移动到 `customer` 模块内，
mod customer {
  use crate::front_of_house::hosting;
  
  pub fn eat_at_restaurant() {
      hosting::add_to_waitlist();
  }
}

// 在子模块 `customer` 内通过 `super::hosting` 引用父模块中的这个短路径。
use crate::front_of_house::hosting;
mod customer {
  pub fn eat_at_restaurant2() {
      super::hosting::add_to_waitlist();
  }
}
```

### [创建惯用的 `use` 路径](https://kaisery.github.io/trpl-zh-cn/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#%E5%88%9B%E5%BB%BA%E6%83%AF%E7%94%A8%E7%9A%84-use-%E8%B7%AF%E5%BE%84)

在示例 7-11 中，你可能会比较疑惑，
- 为什么我们是指定 `use crate::front_of_house::hosting` ，然后在 `eat_at_restaurant` 中调用 `hosting::add_to_waitlist` ，
- 而不是通过指定一直到 `add_to_waitlist` 函数的 `use` 路径来得到相同的结果

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use crate::front_of_house::hosting::add_to_waitlist;

pub fn eat_at_restaurant() {
    add_to_waitlist();
}
```
> 示例 7-13：不清楚 `add_to_waitlist` 是在哪里被定义的。

要想使用 `use` 将函数的父模块引入作用域，我们必须在调用函数时指定父模块，
- 这样可以**清晰地表明函数不是在本地定义的**，
- 同时使完整路径的重复度最小化

使用 `use` 引入结构体、枚举和其他项时，**习惯是指定它们的完整路径**

```rust
use std::collections::HashMap;

fn main() {
    let mut map = HashMap::new();
    map.insert(1, 2);
}
```
> 示例 7-14: 将 `HashMap` 引入作用域的习惯用法

**如何将两个具有相同名称但不同父模块的 `Result` 类型引入作用域**
- Rust 不允许使用 `use` 语句将两个具有相同名称的项带入作用域
- 使用父模块可以区分这两个 `Result` 类型

```rust
use std::fmt;
use std::io;

fn function1() -> fmt::Result {
    // --snip--
}

fn function2() -> io::Result<()> {
    // --snip--
}
```

### [使用 `as` 关键字提供新的名称](https://kaisery.github.io/trpl-zh-cn/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#%E4%BD%BF%E7%94%A8-as-%E5%85%B3%E9%94%AE%E5%AD%97%E6%8F%90%E4%BE%9B%E6%96%B0%E7%9A%84%E5%90%8D%E7%A7%B0)

使用 `use` 将两个同名类型引入同一作用域的另一个办法：
- 在这个类型的路径后面，**使用 `as` 指定一个新的本地名称或者别名**

```rust
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {
    // --snip--
}

fn function2() -> IoResult<()> {
    // --snip--
}
```
> 示例 7-16: 使用 `as` 关键字重命名引入作用域的类型


