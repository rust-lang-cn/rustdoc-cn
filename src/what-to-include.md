# 什么包含（和排除）

说起来项目中的所有内容都需要有文档简单正确，但是我们如何才能做到，以及是否还有内容没有文档？

在顶层的 `src/lib.rs` 或者你的二进制项目 `main.rs` 文件中，包含下面的属性：

```rust
#![warn(missing_docs)]
```

现在运行 `cargo doc`，检查输出，这是一个例子：

```text
 Documenting docdemo v0.1.0 (/Users/username/docdemo)
warning: missing documentation for the crate
 --> src/main.rs:1:1
  |
1 | / #![warn(missing_docs)]
2 | |
3 | | fn main() {
4 | |     println!("Hello, world!");
5 | | }
  | |_^
  |
note: the lint level is defined here
 --> src/main.rs:1:9
  |
1 | #![warn(missing_docs)]
  |         ^^^^^^^^^^^^

warning: 1 warning emitted

    Finished dev [unoptimized + debuginfo] target(s) in 2.96s
```

作为一个库作者，加入 lint `#![deny(missing_docs)]` 是一个确保项目拥有良好文档的好方法，`#![warn(missing_docs)]` 是通向良好文档的好方法。除了文档，`#![deny(missing_doc_code_examples)]` 确保每个函数有一个使用示例。在我们上面的例子中，添加 crate 级别的 lint 来警告。

下面的章节有更多 lints 的细节 [Lints][rustdoc-lints].

## 例子

当然这很简单，但是文档的力量之一是展示的代码易于理解，而不是生产级别。文档通常会忽略错误处理，因为例子需要排除一些不必要的内容保持简单。

`Async` 是一个好例子。为了执行 `async` 例子，一个 executor 是需要的，例子中通常会省略它，让用户自己将 `async` 代码放到自己的运行时。

最好不要在例子中使用 `unwrap()`，并且如果错误处理使得例子难以理解应该被隐藏起来。

``````text
/// Example
/// ```rust
/// let fourtytwo = "42".parse::<u32>()?;
/// println!("{} + 10 = {}", fourtytwo, fourtytwo+10);
/// ```
``````

当 rustdoc wrap 这些到 main 函数中，会编译错误因为 `ParseIntError` trait 没有实现。为了同时帮助读者和测试，这个例子还需要增加些额外代码：

``````text
/// Example
/// ```rust
/// # main() -> Result<(), std::num::ParseIntError> {
/// let fortytwo = "42".parse::<u32>()?;
/// println!("{} + 10 = {}", fortytwo, fortytwo+10);
/// #     Ok(())
/// # }
/// ```
``````

这两个例子在文档页面是相同的，但是对你 crate 的使用者有一些额外的信息。更多的文档测试内容在接下来的 [文档测试] 章节中。

## 什么被排除

默认情况下，你的公共接口可能会默认包含在 rustdoc 输出中。`#[doc(hidden)]` 属性可以隐藏实现细节来鼓励本 crate 的惯用法。

比如，一个内部的 `macro!` 使得 crate 更容易实现如果对用户暴露会使得用户困惑。一个内部的`Error`类型可能存在，并且 `impl` 细节应该被隐藏，如同 [API Guidelines] 中描述的一样。

## 自定义输出

传递一个自定义 css 文件给 `rustdoc` 定义文档的款式是可行的。

```bash
rustdoc --extend-css custom.css src/lib.rs
```

一个良好的例子就是使用这个特性来创建本书的暗黑主题。记住，暗黑主题也包含在点击画笔的输出中。使用可选参数可以很容易使用自定义主题 `.css` 文件：

```bash
rustdoc --theme awesome.css src/lib.rs
```

这是一个新主题 [Ayu] 的例子。

[Ayu]: https://github.com/rust-lang/rust/blob/master/src/librustdoc/html/static/themes/ayu.css
[API Guidelines]: https://rust-lang.github.io/api-guidelines/documentation.html#rustdoc-does-not-show-unhelpful-implementation-details-c-hidden
[Documentation tests]: documentation-tests.md
[on this blog]: https://blog.guillaume-gomez.fr/articles/2016-09-16+Generating+doc+with+rustdoc+and+a+custom+theme
[rustdoc-lints]: lints.md
