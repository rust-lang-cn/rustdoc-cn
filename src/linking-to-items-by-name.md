# 通过名称链接 item

Rustdoc 能够使用 item 的路径直接内部链接。这称为 'intra-doc link'。

比如，下面的代码可以链接 `Bar` 的页面：

```rust
/// This struct is not [Bar]
pub struct Foo1;

/// This struct is also not [bar](Bar)
pub struct Foo2;

/// This struct is also not [bar][b]
///
/// [b]: Bar
pub struct Foo3;

/// This struct is also not [`Bar`]
pub struct Foo4;

/// This struct *is* [`Bar`]!
pub struct Bar;
```

不像常规的 markdown，`[bar][Bar]` 语法也被支持，不需要`[Bar]: ...` 链接。

反引号会被删除，所以 ``[`Option`]`` 可以正确地链接到`Option`。

## 有效链接

你可以链接作用域的任何东西，使用路径，包括 `Self`, `self`, `super`和 `crate`。Associated items (functions, types, and constants) 也是支持的，但是 [不能是空的 trait 实现][#79682]。Rustdoc 还支持 [the standard library documentation](../std/index.html#primitives) 列出的原始类型链接。

[#79682]: https://github.com/rust-lang/rust/pull/79682

你还可以链接范型参数，比如 `Vec<T>`。链接会如同你写了 ``[`Vec<T>`](Vec)``. 但是，Fully-qualified syntax（比如，`<Vec as IntoIterator>::into_iter()`） 还 [没有被支持][fqs-issue]。

[fqs-issue]: https://github.com/rust-lang/rust/issues/74563

```rust,edition2018
use std::sync::mpsc::Receiver;

/// This is a version of [`Receiver<T>`] with support for [`std::future`].
///
/// You can obtain a [`std::future::Future`] by calling [`Self::recv()`].
pub struct AsyncReceiver<T> {
    sender: Receiver<T>
}

impl<T> AsyncReceiver<T> {
    pub async fn recv() -> T {
        unimplemented!()
    }
}
```

Rustdoc 允许使用 URL fragment specifiers，就如同普通的链接：

```rust
/// This is a special implementation of [positional parameters].
///
/// [positional parameters]: std::fmt#formatting-parameters
struct MySpecialFormatter;
```

## Namespaces and Disambiguators（消歧义符）

Rust 中的路径有三种命名空间：type, value 和 macro。Item name 在命名空间内必须唯一，但是可以与其他命名空间的 item 同名。在歧义的情况下，rustdoc 会警告歧义，并给出消除歧义的建议。

```rust
/// See also: [`Foo`](struct@Foo)
struct Bar;

/// This is different from [`Foo`](fn@Foo)
struct Foo {}

fn Foo() {}
```

这些前缀展示在文档时会被删除，所以 `[struct@Foo]` 会被渲染成 `Foo`。

你也可以在函数名后加上 `()` 和在宏名后面加上 `!` 消除歧义：

```rust
/// This is different from [`foo!`]
fn foo() {}

/// This is different from [`foo()`]
macro_rules! foo {
  () => {}
}
```

## 警告，re-exports, and scoping

即使 item 被 re-exported，链接也在 item 定义的模块内解析。如果来自另一个 crate 的链接解析失败，不会给出警告。

```rust,edition2018
mod inner {
    /// Link to [f()]
    pub struct S;
    pub fn f() {}
}
pub use inner::S; // the link to `f` will still resolve correctly
```

当一个 item 被 re-export，rustdoc 允许加入额外的文档。额外的文档在 re-export 的作用域中解析，允许你链接 re-export 的 item，并且如果解析失败会给出警告。

```rust
/// See also [foo()]
pub use std::process::Command;

pub fn foo() {}
```

这对于过程宏非常有用，过程宏必须定义在自己的 crate 中。

注意：因为 `macro_ruls!` 宏的作用域是整个 Rust，`macro_rules!` 的 intra-doc 链接在 [relative to the crate root][#72243] 解析，而不是定义的模块内。

如果链接看起来不像是 intra-doc 链接，它们会被忽略并且不会生成警告，即使链接解析失败。比如，任何包含了 `/` 或者 `[]` 字符的链接都被忽略。

[#72243]: https://github.com/rust-lang/rust/issues/72243
