# 什么是 rustdoc?

标准 Rust 版本包含了名为 `rustdoc` 的工具。它的作用是为 Rust 项目生成文档，Rustdoc 接受一个 crate 根目录或者一个 markdown 文件作为参数，生成 HTML，CSS 和 Javascript 文件。

## 基本使用

让我们试用一下，使用 Cargo 创建一个新项目

```bash
$ cargo new docs --lib
$ cd docs
```

在 `src/lib.rs` 中，Cargo 生成了一些事例代码，删掉使用下面的代码代替

```rust
/// foo is a function
fn foo() {}
```

然后我们运行 `rustdoc` 。我们可以在 crate 根目录执行下面的命令

```bash
$ rustdoc src/lib.rs
```
这会创建一个新目录`doc`，其中是一个网站结构。在我们的例子中，主页面是 `doc/lib/index.html`。如果你使用浏览器打开，可以在 tab 栏看到 "Crate lib"，页面没有内容。

## 配置 rustdoc

现在有两个问题：第一，为什么我们的包名字是 "lib"？第二，为什么没有任何内容？

第一个问题的原因是因为`rustdoc`试图更好用，像`rustc`就假定我们 crate 的名字就是 crate 根目录文件的名字。为了修复这个问题，可以通过命令行传递参数：

```bash
$ rustdoc src/lib.rs --crate-name docs
```

现在， `doc/docs/index.html` 文件生成，页面名称为 "Crate docs"。

对于第二个问题，因为我们的函数`foo`不是公共的；`rustdoc`默认只会为公共函数生成文档，如果我们将代码修改为

```rust
/// foo is a function
pub fn foo() {}
```

然后重新运行 `rustdoc`:

```bash
$ rustdoc src/lib.rs --crate-name docs
```

现在我们生成了文档，打开`doc/docs/index.html`，显示了`foo`函数的连接页面，文件位置是`doc/docs/fn.foo.html`。在函数的页面上，你可以看到我们写的注释"foo is a function"。

## 通过 Cargo 使用 rustdoc

Cargo 整合了`rustdoc`，使得生成文档更容易。代替`rustdoc`命令，我们可以这样做：

```bash
$ cargo doc
```

实际上，会这样调用`rustdoc`:

```bash
$ rustdoc --crate-name docs src/lib.rs -o <path>/docs/target/doc -L
dependency=<path>/docs/target/debug/deps
```

你可以使用`cargo doc --verbose`看到这个过程。

它会自动生成正确的 crate 名称`--crate-name`，同时指向`src/lib.rs`。但是其他的参数表示什么呢？

 - `-o` 控制文档的输出。不使用顶层的`doc` 目录，Cargo 会把生成的文档生成在  `target`。这是 Cargo 项目的惯用生成文件的位置。
 - `-L` 帮助 rustdoc 找到代码的依赖，如果我们的项目有依赖，也会生成依赖的文档。

## Outer 和 inner 文档

`///` 符号用来对下面一个 item 生成文档，所以称为 outer 文档。还有符号`//!`，用来生成 item 内部的文档，也叫做 inner 文档，通常用来对整个 crate 生成文档，因为是 crate 的根，没有 item 在前面。所以为了生成整个 crate 的文档，你需要这样用 `//!`：

``` rust
//! This is my first rust crate
```

当这样用的时候，会生成 item 内部的文档，也就是 crate 自己。

为了获取更多`//!`的信息，请看 [the Book]( https://doc.rust-lang.org/book/ch14-02-publishing-to-crates-io.html#commenting-contained-items)

## 利用单独的 Markdown 文件

`rustdoc` 可以为单独的 markdown 文件生成 HTML。让我们尝试一下，创建`README.md`，并输入如下内容

````text
# Docs

This is a project to test out `rustdoc`.

[Here is a link!](https://www.rust-lang.org)

## Example

```rust
fn foo() -> i32 {
    1 + 1
}
```
````

然后运行 `rustdoc` 命令：

```bash
$ rustdoc README.md
```

你能发现生成的`docs/doc/README.html`文件。

不过现在 Cargo 不能这样操作 markdown 文件。

## 总结

本节涵盖了`rustdoc`的基本使用方法。本书的剩余部分会展示`rustdoc`所有的可选功能，以及如何使用它们。
