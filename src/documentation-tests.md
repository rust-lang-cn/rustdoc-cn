# 文档测试

`rustdoc` 支持将你文档中的代码示例作为测试执行。这确保了文档中的代码示例保持更新。

基本写法是这样的：

```rust,no_run
/// # Examples
///
/// ```
/// let x = 5;
/// ```
# fn f() {}
```

三个反引号中的代码块。如果有一个文件名叫做`foo.rs`，运行`rustdoc --test foo.rs`会提取这个例子，然后作为测试执行。

需要注意的是，如果代码块没有设置语言，rustdoc 默认是 Rust 代码，所以下面的：

````markdown
```rust
let x = 5;
```
````

跟这个是相等的

````markdown
```
let x = 5;
```
````

还有一些微妙之处！请阅读获得更多详情。

## 文档测试通过或者失败

就像常规的单元测试，常规的文档测试也需要编译和运行“通过”。所以如果需要计算给出一个结果，可以使用`assert!`系列宏来检查：

```rust
let foo = "foo";
assert_eq!(foo, "foo");
```

这样，如果计算返回的结果不符合预期，代码就会 panic，文档测试会失败。

## 预处理例子

在上面的例子中，你注意到奇怪的事情：没有`main`函数！如果强制你为每个例子写`main`，增加了难度。
所以`rustdoc`在运行例子前会帮你处理好这个。这里是`rustdoc`预处理的完整算法：

1. 一些常用的 `allow` 属性会被插入，包括 `unused_variables`, `unused_assignments`, `unused_mut`, `unused_attributes`, 和 `dead_code`。小型示例经常出发这些警告。
2. 如果存在 `#![doc(test(attr(...)))]` 的属性会被加入。
3. `#![foo]` 属性会被作为 crate 属性保留。
4. 如果例子不包含 `extern crate`, 并且 `#![doc(test(no_crate_inject))]` 没有被指定，`extern crate <mycrate>;` 被插入（注意 `#[macro_use]` 要手动写一般）。
5. 最后，如果例子不包含 `fn main`，剩下的代码会被 main 函数 wrap：`fn main() { your_code }`。

对于第 4 条的详细解释，请阅读下面的“宏的文档”

## 隐藏例子的一部分

有时，你需要一些初始代码，或者一些会分散文档注意力的代码，但是它们对测试工作是必要的。考虑下面的示例代码：

```rust,no_run
/// ```
/// /// Some documentation.
/// # fn foo() {} // this function will be hidden
/// println!("Hello, World!");
/// ```
# fn f() {}
```

会渲染为：

```rust
/// Some documentation.
# fn foo() {}
println!("Hello, World!");
```

没错，这是对的，你可以加一些以 `# ` 开头的行，在输出中它们会隐藏，但是编译代码会用到。你可以利用这一点。在这个例子中，文档注释需要使用函数，但是我只想给你看文档注释，我需要加入函数的定义。同时，需要满足编译器编译，而隐藏这部分代码使得示例更清晰。你可以使用这个技术写出详细的示例代码并且保留你的可测试性文档。

比如，想象我们想要给这些代码写文档：

```rust
let x = 5;
let y = 6;
println!("{}", x + y);
```

文档注释最终可能是这样的：

> First, we set `x` to five:
>
> ```rust
> let x = 5;
> # let y = 6;
> # println!("{}", x + y);
> ```
>
> Next, we set `y` to six:
>
> ```rust
> # let x = 5;
> let y = 6;
> # println!("{}", x + y);
> ```
>
> Finally, we print the sum of `x` and `y`:
>
> ```rust
> # let x = 5;
> # let y = 6;
> println!("{}", x + y);
> ```

为了保持每个代码块可测试，我们要在每个代码块都有完整代码，但是我们不想文档读者每次都看到全部行代码：

````markdown
First, we set `x` to five:

```
let x = 5;
# let y = 6;
# println!("{}", x + y);
```

Next, we set `y` to six:

```
# let x = 5;
let y = 6;
# println!("{}", x + y);
```

Finally, we print the sum of `x` and `y`:

```
# let x = 5;
# let y = 6;
println!("{}", x + y);
```
````

通过复制例子的所有代码，例子可以通过编译，同时使用 `# ` 在文档中有些部分被隐藏。

`#` 的隐藏可以使用两个 `##` 来消除。 如果我们有一行注释，以 `#` 开头，那么这样写：

```rust
let s = "foo
## bar # baz";
```

我们可以转义第一个 `#` 来注释它：

```text
/// let s = "foo
/// ## bar # baz";
```

## 在文档测试中使用 `?`

当写例子时，很少会包含完整的错误处理，因为错误处理会增加很多样板代码。取而代之，你可能更希望这样：

```rust,no_run
/// ```
/// use std::io;
/// let mut input = String::new();
/// io::stdin().read_line(&mut input)?;
/// ```
# fn f() {}
```

问题是 `?` 返回 `Result<T, E>`，测试函数不能返回任何东西，所以会有类型错误。

你可以通过自己增加返回 `Result<T, E>` 的 `main` 函数来规避这个限制，因为 `Result<T, E>` 实现了 `Termination` trait：

```rust,no_run
/// A doc test using ?
///
/// ```
/// use std::io;
///
/// fn main() -> io::Result<()> {
///     let mut input = String::new();
///     io::stdin().read_line(&mut input)?;
///     Ok(())
/// }
/// ```
# fn f() {}
```

与下节的 `# `一起，你可以得到读者舒服，编译通过的完整解决方案：

```rust,no_run
/// ```
/// use std::io;
/// # fn main() -> io::Result<()> {
/// let mut input = String::new();
/// io::stdin().read_line(&mut input)?;
/// # Ok(())
/// # }
/// ```
# fn f() {}
```

从 1.34.0 版本开始，也可以省略 `fn main()`，但是你必须消除错误类型的歧义：

```rust,no_run
/// ```
/// use std::io;
/// let mut input = String::new();
/// io::stdin().read_line(&mut input)?;
/// # Ok::<(), io::Error>(())
/// ```
# fn f() {}
```

这是 `?` 操作符隐式转换带来的不便，因为类型不唯一所以类型推断会出错。你必须写 `(())`，`rustdoc` 才能理解你想要一个隐式返回值 `Result` 的函数。

## 在文档测试中显示警告

你可以通过运行 `rustdoc --test --test-args=--show-output` 在文档测试中显示警告（或者，如果你使用 cargo，`cargo test --doc -- --show-output` 也可以）。默认会隐藏 `unused` 警告，因为很多例子使用私有函数；你可以通过在例子顶部加入`#![warn(unused)]`来对没有使用的变量或者死代码进行警告。你还可以在 crate 根使用 [`#![doc(test(attr(warn(unused))))]`][test-attr] 开启全局警告。

[test-attr]: ./the-doc-attribute.md#testattr

## 宏的文档

这里是一个宏的文档的例子：

```rust
/// Panic with a given message unless an expression evaluates to true.
///
/// # Examples
///
/// ```
/// # #[macro_use] extern crate foo;
/// # fn main() {
/// panic_unless!(1 + 1 == 2, “Math is broken.”);
/// # }
/// ```
///
/// ```should_panic
/// # #[macro_use] extern crate foo;
/// # fn main() {
/// panic_unless!(true == false, “I’m broken.”);
/// # }
/// ```
#[macro_export]
macro_rules! panic_unless {
    ($condition:expr, $($rest:expr),+) => ({ if ! $condition { panic!($($rest),+); } });
}
# fn main() {}
```

你注意到三件事：我们需要自己增加 `extern crate` 一行，从而我们可以加上 `#[macro_use]` 属性。第二，我们需要自己增加 `main()`，理由同上。最后 `#` 的使用使得一些内容不会出现在输出中。

## 属性

代码块可以通过属性标注帮助 `rustdoc` 在测试例子代码时处理正确。

`ignore` 属性告诉 Rust 忽略你的代码。 这个属性很通用，并且考虑标注 `文本`或者使用`#`隐藏不想展示的部分。

```rust
/// ```ignore
/// fn foo() {
/// ```
# fn foo() {}
```

`should_panic` 告诉 `rustdoc` 代码应该编译通过但是运行时会 panic。如果代码没有 panic，测试失败。

```rust
/// ```should_panic
/// assert!(false);
/// ```
# fn foo() {}
```

`no_run` 属性会编译你的代码但是不运行它。这对于类似 "如果获取网页" 的例子很重要，你要确保它能编译但是不想在运行测试，因为测试环境可能没有网络。这个属性也可以被用来要求代码段有未定义行为。

```rust
/// ```no_run
/// loop {
///     println!("Hello, world");
/// }
/// ```
# fn foo() {}
```

`compile_fail` 告诉 `rustdoc` 应该编译失败。如果便已通过，测试失败。
但是需要注意现在版本 Rust 编译失败可能在将来 Rust 版本编译成功。

```rust
/// ```compile_fail
/// let x = 5;
/// x += 2; // shouldn't compile!
/// ```
# fn foo() {}
```

`edition2015`, `edition2018` 和 `edition2021` 告诉 `rustdoc` 代码应该使用哪个 edition 版本的 Rust 来编译。

```rust
/// Only runs on the 2018 edition.
///
/// ```edition2018
/// let result: Result<i32, ParseIntError> = try {
///     "1".parse::<i32>()?
///         + "2".parse::<i32>()?
///         + "3".parse::<i32>()?
/// };
/// ```
# fn foo() {}
```

## 语法参考

代码块的 *exact* 语法，包括边缘情况，可以在 CommonMark 说明的 [Fenced Code Blocks] 一节找到。

Rustdoc 也接受 *indented* 代码块作为 fenced 代码块的替代：不使用三个反引号，而是每行以四个或者以上空格开始。

```markdown
    let foo = "foo";
    assert_eq!(foo, "foo");
```

这也在 CommonMark 说明中的 [Indented Code Blocks](https://spec.commonmark.org/0.29/#indented-code-blocks) 小节。

但是通常更常用的是 fenced 代码块。不仅是 fenced 代码块更符合 Rust 惯用法，而且 indented 代码块无法使用诸如 `ignore` 或者 `should_panic` 这些属性。

### 收集文档测试时包含 item

Rustdoc 文档测试可以做到一些单元测试无法做到的事，可以扩展你的文档测试但是不出现在文档中。意味着，rustdoc 允许你有不出现在
文档中，但是会进行文档测试的 item，所以你可以利用文档测试这个功能使测试不出现在文档中，或者任意找一个私有 item 包含它。

当编译 crate 用来测试文档时（使用`--test`)，`rustdoc`会设置 `#[cfg(doctest)]`。注意这只会在公共 item 上设置；如果你需要测试私有 item，你需要写单元测试。

在这个例子中，我们增加不会编译的文档测试，确保我们的结构体只会获取有效数据：

```rust
/// We have a struct here. Remember it doesn't accept negative numbers!
pub struct MyStruct(pub usize);

/// ```compile_fail
/// let x = my_crate::MyStruct(-5);
/// ```
#[cfg(doctest)]
pub struct MyStructOnlyTakesUsize;
```

注意结构 `MyStructOnlyTakesUsize` 不是你的 crate 公共 API。`#[cfg(doctest)]` 的使用使得这个结构体只会在 `rustdoc` 收集文档测试时存在。这意味着当传递 `--test` 给 rustdoc 时才存在，在公共文档中是隐藏的。

另一个可能用到 `#[cfg(doctest)]` 的是你的 README 文件中的文档测试。
比如你可以在 `lib.rs` 写下面的代码来运行 README 中的文档测试：

```rust,no_run
#[doc = include_str!("../README.md")]
#[cfg(doctest)]
pub struct ReadmeDoctests;
```

这会在你的隐藏结构体 `ReadmeDoctests` 中包含你的 README 作为文档，也会作为文档测试来执行。
