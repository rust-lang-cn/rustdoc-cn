# `#[doc]` 属性

`#[doc]` 属性可以让你控制 `rustdoc` 工作的各个方面。

`#[doc]` 最基本的作用就是处理文档内容。就是说，`///` 就是 `#[doc]` 的语法糖。下面的两行注释是一样的：

```rust,no_run
/// This is a doc comment.
#[doc = " This is a doc comment."]
# fn f() {}
```

（请注意属性版本的开始的空格。）

在大多数情况下，`///` 比 `#[doc]` 更容易使用。一种后面更容易使用的场景是给宏生成文档；`collapse-docs` 会组合多个 `#[doc]`属性为一条文档注释，比如：

```rust,no_run
#[doc = "This is"]
#[doc = " a "]
#[doc = "doc comment"]
# fn f() {}
```

这样可能感觉更灵活。注意这跟下面的写法是一样的：

```rust,no_run
#[doc = "This is\n a \ndoc comment"]
# fn f() {}
```

给出的文档会渲染成 markdown，会删除换行符。

另一个有用的场景是引入外部文件：

```rust,no_run,ignore
#[doc = include_str!("../README.md")]
# fn f() {}
```

`doc` 属性有更多的选项！不会包含在输出中，但是可以控制输出的表示。我们将它们分为两大类：在 crate 层面使用的，和在 item 层面使用的。

## crate 层面

这些选项控制文档在 crate 层面如何表示。

### `html_favicon_url`

这个 `doc` 属性让你控制你的文档图标。

```rust,no_run
#![doc(html_favicon_url = "https://example.com/favicon.ico")]
```

这会在你的文档中加入 `<link rel="shortcut icon" href="{}">`，属性的值会填入 `{}`。

如果你不使用这个属性，就没有图标。

### `html_logo_url`

这个 `doc` 属性可以让你控制左上角的 logo。

```rust,no_run
#![doc(html_logo_url = "https://example.com/logo.jpg")]
```

这会在你的文档中加入 `<a href='index.html'><img src='{}' alt='logo' width='100'></a>`，属性的值会填入 `{}`。

如果你不使用这个属性，就没有 logo。

### `html_playground_url`

这个 `doc` 属性让你控制文档示例中的 "run" 按钮的请求到哪里。

```rust,no_run
#![doc(html_playground_url = "https://playground.example.com/")]
```

现在，当你按下 "run"，会向对应网站发出请求。

如果你没有使用这个属性，没有运行按钮。

### `issue_tracker_base_url`

这个 `doc` 属性在标准库中使用最多；当一个特性未稳定时，需要提供 issue number 来追踪这个特性。`rustdoc` 使用这个 number，加入到给定的基本 URL 来链接到追踪的网址。

```rust,no_run
#![doc(issue_tracker_base_url = "https://github.com/rust-lang/rust/issues/")]
```

### `html_root_url`

`#[doc(html_root_url = "…")]` 属性的值表明了生成外部 crate 的 URL。当 rustdoc 需要生成一个外部 crate item 的链接时，首先检查本地外部 crate 的文档，如果存在直接链接指向。如果失败，就会使用 `--extern-html-root-url` 命令行参数的值，如果没有这个参数，才会使用 `html_root_url` ，如果还是无效，外部 item 不会链接。

```rust,no_run
#![doc(html_root_url = "https://docs.rs/serde/1.0")]
```

### `html_no_source`

默认情况下，`rustdoc` 会包含你的源码链接到文档中。
但是如果你这样写：

```rust,no_run
#![doc(html_no_source)]
```

就不会。

### `test(no_crate_inject)`

默认情况下，`rustdoc` 会自动加一行 `extern crate my_crate;` 到每个文档测试中。
但是如果你这样写了：

```rust,no_run
#![doc(test(no_crate_inject))]
```

就不会。

### `test(attr(...))`

这个 `doc` 属性允许你对你所有的文档测试加上某个属性。比如，如果你想要你的文档测试存在警告时失败，可以这样写：

```rust,no_run
#![doc(test(attr(deny(warnings))))]
```

## item 层面

这些 `#[doc]` 属性单独给 item 使用，控制 item 文档表示。

### `inline` and `no_inline`

<span id="docno_inlinedocinline"></span>

这两个属性可以用于 `use` 声明。比如，考虑如下 Rust 代码：

```rust,no_run
pub use bar::Bar;

/// bar docs
pub mod bar {
    /// the docs for Bar
    pub struct Bar;
}
# fn main() {}
```

文档会生成 "Re-exports" 小节，表示 `pub use bar::Bar;` 其中 `Bar` 会链接到自己的页面。

如果我们将代码改为：

```rust,no_run
#[doc(inline)]
pub use bar::Bar;
# pub mod bar { pub struct Bar; }
# fn main() {}
```

`Bar` 就会出现在 `Structs` 小节，就像 `Bar` 就定义在顶层一样，而不是 `pub use` 的。

然后我们修改原始的例子，使 `bar` 私有：

```rust,no_run
pub use bar::Bar;

/// bar docs
mod bar {
    /// the docs for Bar
    pub struct Bar;
}
# fn main() {}
```

这里，因为 `bar` 不是公共的，`Bar` 没有自己的页面，所有没有链接可以指向。`rustdoc` 将会内联定义，所以会得到与 `#[doc(inline)]` 一样的结果：`Bar` 就会出现在 `Structs` 小节，就像 `Bar` 就定义在顶层一样。如果我们加上 `no_inline` 属性：

```rust,no_run
#[doc(no_inline)]
pub use bar::Bar;

/// bar docs
mod bar {
    /// the docs for Bar
    pub struct Bar;
}
# fn main() {}
```

现在我们有了 `Re-exports`，并且 `Bar` 没有链接到任何页面。

一个特殊情况：在 Rust 2018 以及更高版本，如果你 `pub use` 你的依赖，`rustdoc` 不会作为 modules 内联除非你加上 `#[doc(inline)]`。

### `hidden`

<span id="dochidden"></span>

任何标注了 `#[doc(hidden)]` 的 item 不会出现在文档中，除非 `strip-hidden` pass 被删除。

### `alias`

这个属性给搜索索引增加了别名。

让我们举个例子：

```rust,no_run
#[doc(alias = "TheAlias")]
pub struct SomeType;
```

现在，如果你输入 "TheAlias" 搜索，也会显示 `SomeType`。当然如果你输入 `SomeType` 也会显示 `SomeType`！

#### FFI 例子

文档属性在写 c 库的 bingding 时尤其有用。比如，我们有一个下面这样的 C 函数：

```c
int lib_name_do_something(Obj *obj);
```

它输入一个指向 `Obj` 类型的指针返回一个整数。在 Rust 中，可能会这样写：

```ignore (using non-existing ffi types)
pub struct Obj {
    inner: *mut ffi::Obj,
}

impl Obj {
    pub fn do_something(&mut self) -> i32 {
        unsafe { ffi::lib_name_do_something(self.inner) }
    }
}
```

函数已经被转换为一个方法便于使用。但是如果你想要寻找 Rust 相当的 `lib_name_do_something`，你没有办法做到。

为了避免这个限制，我们只需要在 `do_something` 方法加上 `#[doc(alias = "lib_name_do_something")]`，然后就可以了！

用户可以直接搜索 `lib_name_do_something` 然后找到`Obj::do_something`。
