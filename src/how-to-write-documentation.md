# 如何写文档

好的文档并不自然。存在一些矛盾的目标使得写好文档很困难。即要求对领域很专业又要写出对新手很友好的文档。
文档因此经常隐去一些细节，或者留给读者一些未解答的问题。

Rust 文档有一些原则指导任何人来写文档，从而每个人都有机会来使用代码。

本章不仅覆盖如何编写文档，还介绍了如何写出**好**文档。尽可能清晰完整非常重要。根据经验：
你编写的文档越多你的 crate 越好。如果 item 是公共的，就应该有文档。

## 开始

编写 crate 文档首先应该从首页开始。例如 [`hashbrown`] crate 级别的文档总结了这个 crate 的角色是什么，
说明了使用的详细技术，以及为什么你需要这个 crate。

在介绍了 crate 之后，首页给出使用 crate 的代码示例很重要。在代码例子中展示库如何使用，不要使用裁剪过的代码，
使得用户可以直接复制粘贴就能运行。

[`futures`] 使用内联注释逐行解释使用 [`Future`] 的复杂性，因为用户接触
rust 的 [`Future`] 的第一个例子可能就是这个。

[`backtrace`] 文档描述了整个使用过程，说明了`Cargo.toml`文件应该如何修改，传递命令行参数
给编译器，并展示了一个使用 backtrace 的例子。

最后，首页会成为如何使用 crate 的综合参考，就像 [`regex`]。
在这个首页，所有的依赖被列出，边缘情况被列出，实际例子被列出。然后首页继续展示如何使用正则表达式，然后还列出了
crate 的特性。

不要担心你的新 crate 与已经开发一段时间的 crate 比较。要是文档逐步完善，请逐步开始添加介绍，示例和特性。罗马不是
一天建成的！

`lib.rs` 的第一行开始会是首页，它们与 rustdoc 其他部分不同，要以`//!`开始表明这是模块级别或者 crate 级别的文档。
这是一个简单的例子：

```rust,no_run
//! Fast and easy queue abstraction.
//!
//! Provides an abstraction over a queue.  When the abstraction is used
//! there are these advantages:
//! - Fast
//! - [`Easy`]
//!
//! [`Easy`]: http://thatwaseasy.example.com

/// This module makes it easy.
pub mod easy {

    /// Use the abstraction function to do this specific thing.
    pub fn abstraction() {}

}
```

理想情况下，文档第一行是没有技术细节的句子，但是很好的描述了在 Rust 生态中的位置。
阅读这行后，用户应该知道 crate 是否满足他们的需要。

## 文档组成

无论是 modules, structs, funtions, macros： 代码的公共 API 都应该有文档。很少有人嫌弃文档太多！

每个 item 的文档应该都以下面的结构构成：

```text
[short sentence explaining what it is]

[more detailed explanation]

[at least one code example that users can copy/paste to try it]

[even more advanced explanations if necessary]
```

编写文档时，基本结构应该很容易遵循；你可能认为代码示例微不足道，但是它真的很重要，
能帮助用户理解 item 是什么，如何使用，以及存在的目的是什么。

让我们看一个来自 [standard library] 的例子，
[`std::env::args()`][env::args] 函数：

``````markdown
Returns the arguments which this program was started with (normally passed
via the command line).

The first element is traditionally the path of the executable, but it can be
set to arbitrary text, and may not even exist. This means this property should
not be relied upon for security purposes.

On Unix systems shell usually expands unquoted arguments with glob patterns
(such as `*` and `?`). On Windows this is not done, and such arguments are
passed as-is.

# Panics

The returned iterator will panic during iteration if any argument to the
process is not valid unicode. If this is not desired,
use the [`args_os`] function instead.

# Examples

```
use std::env;

// Prints each argument on a separate line
for argument in env::args() {
    println!("{}", argument);
}
```

[`args_os`]: ./fn.args_os.html
``````

在第一个空行之间的所有内容都会被用于搜索和模块的简介。比如，上面的`std::enve::args()`函数就会在展示在 [`std::env`] 模块文档中。
将摘要保持在一行是良好习惯：简介是好文档的目标。

因为类型系统很好定义了函数的参数和返回值类型，所以将其显式写入文档没有好处，
尤其是`rustdoc`会自动在函数签名中添加指向类型的超链接。

在上面的例子中，`Panics`小节解释了代码何时可能会意外退出，
可以帮助读者规避 panic。如果你知道代码的边缘情况，尽可能增加 panic 小节。

如同你所看到的，它遵循了给出的结构建议：简短描述函数的作用，然后提供了更多信息以及最后提供了代码示例。

## Markdown

`rustdoc`使用 [CommonMark Markdown specification]。你可能会对它们的网站感兴趣：:

 - [CommonMark quick reference]
 - [current spec]

补充了标准 CommonMark 语法， `rustdoc` 支持几种扩展：

### Strikethrough（删除线）

文本可以通过两个波浪线来渲染删除线：

```text
An example of ~~strikethrough text~~.
```

这个例子会渲染成：

> An example of ~~strikethrough text~~.

这使用 [GitHub Strikethrough extension][strikethrough].

### Footnotes（角标）

角标会生成一个小号数字链接，点击数字链接会跳转到这个 item 的位置。角标标签类似与链接语法。例子如下：

```text
This is an example of a footnote[^note].

[^note]: This text is the contents of the footnote, which will be rendered
    towards the bottom.
```

这个例子会渲染成：

> This is an example of a footnote[^note].
>
> [^note]: This text is the contents of the footnote, which will be rendered
>     towards the bottom.

角标数字会根据角标位置自动生成。

### Tables（表格）

表格可以可以通过竖线和短横线来表示表格的行和列。被转换为符合 HTML 形状的表格。比如：

```text
| Header1 | Header2 |
|---------|---------|
| abc     | def     |
```

这个例子会被渲染成类似这样：

> | Header1 | Header2 |
> |---------|---------|
> | abc     | def     |

阅读 [GitHub Tables extension][tables] 的说明获取更多细节。

### Task lists

任务列表可以用于检查需要完成的条目。
比如：

```md
- [x] Complete task
- [ ] Incomplete task
```

会被渲染成：

> - [x] Complete task
> - [ ] Incomplete task

阅读 [task list extension] 获得更多细节。

### Smart punctuation（标点符号）

一些 ASCII 符号可以自动转换为更好看的 Unicode 符号：

| ASCII sequence | Unicode |
|----------------|---------|
| `--`           | –       |
| `---`          | —       |
| `...`          | …       |
| `"`            | “ or ”, depending on context |
| `'`            | ‘ or ’, depending on context |

所以，不需要手工输入这些 Unicode 符号！

[`backtrace`]: https://docs.rs/backtrace/0.3.50/backtrace/
[commonmark markdown specification]: https://commonmark.org/
[commonmark quick reference]: https://commonmark.org/help/
[env::args]: https://doc.rust-lang.org/stable/std/env/fn.args.html
[`Future`]: https://doc.rust-lang.org/std/future/trait.Future.html
[`futures`]: https://docs.rs/futures/0.3.5/futures/
[`hashbrown`]: https://docs.rs/hashbrown/0.8.2/hashbrown/
[`regex`]: https://docs.rs/regex/1.3.9/regex/
[standard library]: https://doc.rust-lang.org/stable/std/index.html
[current spec]: https://spec.commonmark.org/current/
[`std::env`]: https://doc.rust-lang.org/stable/std/env/index.html#functions
[strikethrough]: https://github.github.com/gfm/#strikethrough-extension-
[tables]: https://github.github.com/gfm/#tables-extension-
[task list extension]: https://github.github.com/gfm/#task-list-items-extension-
