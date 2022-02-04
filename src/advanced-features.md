# 高级特性

本页列出的特性不属于其他的主要类别

## `#[cfg(doc)]`: 文档平台相关或者特性相关信息

对于条件编译，rustdoc 会与编译器相同的方式处理。只生成目标主机可用的文档，其他的被 “过滤掉”。当对于不同目标提供不同的东西并且你希望文档反映你所有的可用项目，这可能会有问题。

如果你希望确保 rustdoc 可以看到某个 item，而忽略目标的平台相关信息，你可以使用 `#[cfg(doc)]`。Rustdoc 会在构建文档时设置它，所以使用了这个的 item 会确保生成到文档中，比如 `#[cfg(any(windows, doc))]` 会在 Windows 上构建以及生成所有的文档。

请注意 `cfg` 不会传递到文档测试中。

比如：

```rust
/// Token struct that can only be used on Windows.
#[cfg(any(windows, doc))]
pub struct WindowsToken;
/// Token struct that can only be used on Unix.
#[cfg(any(unix, doc))]
pub struct UnixToken;
```

这里，各自的 tokens 只能在各自的平台上使用，但是会同时出现在文档中。

### 特定平台文档之间的交互

Rustdoc 没有什么如同你在每种平台运行一次的魔法方法来编译文档（这种魔法方式被称为 ['holy grail of rustdoc'][#1998]）。代替的是，会一次读到你**所有的**代码，与 Rust 编译器通过参数 `--cfg doc` 读到的一样。但是，Rustdoc 有一个技巧可以在接收到特定平台代码时处理它。

为你的 crate 生成文档，Rustdoc 只需要知道你的公共函数签名。尤其是，不需要知道你的函数实现，所以会忽略所有函数体类型错误和名称解析错误。注意，这不适用函数体之外的东西：因为 Rustdoc 会记录类型，需要知道类型是什么。比如，无论平台如何，这些代码可以工作：

```rust,ignore (platform-specific,rustdoc-specific-behavior)
pub fn f() {
    use std::os::windows::ffi::OsStrExt;
}
```

但是这些不行，因为函数签名中存在为止类型：

```rust,ignore (platform-specific,rustdoc-specific-behavior)
pub fn f() -> std::os::windows::ffi::EncodeWide<'static> {
    unimplemented!()
}
```

更多的代码示例，请参考 [the rustdoc test suite][realistic-async]。

[#1998]: https://github.com/rust-lang/rust/issues/1998
[realistic-async]: https://github.com/rust-lang/rust/blob/b146000e910ccd60bdcde89363cb6aa14ecc0d95/src/test/rustdoc-ui/error-in-impl-trait/realistic-async.rs

## 增加文档搜索的 item 别名

这个特性可以通过 `doc(alias)` 属性增加 `rustdoc` 搜索 item 的别名。比如：

```rust,no_run
#[doc(alias = "x")]
#[doc(alias = "big")]
pub struct BigX;
```

然后，当搜索时，你可以输入 "x" 或者 "big"，搜索结果优先显示 `BigX` 结构。

文档别名也有一些限制：你不能使用 `"` 或者空格。

你可以使用列表一次增加多个别名：

```rust,no_run
#[doc(alias("x", "big"))]
pub struct BigX;
```
