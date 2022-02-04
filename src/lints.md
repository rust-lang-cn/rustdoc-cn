# Lints

`rustdoc` 提供 lints 来帮助你编写以及测试文档。你可以像使用其他 lints 来使用它们：

```rust
#![allow(rustdoc::broken_intra_doc_links)] // allows the lint, no diagnostics will be reported
#![warn(rustdoc::broken_intra_doc_links)] // warn if there are broken intra-doc links
#![deny(rustdoc::broken_intra_doc_links)] // error if there are broken intra-doc links
```
注意，出了`missing_docs`，这些 lints 只有当运行`rustdoc`的时候才会生效，`rustc`不会。

这是`rustdoc` lints 的列表：

## broken_intra_doc_links

这个 lint 是**默认警告**。这个 lint 当 [intra-doc link] 解析错误时会提示。比如：

[intra-doc link]: linking-to-items-by-name.md

```rust
/// I want to link to [`Nonexistent`] but it doesn't exist!
pub fn foo() {}
```
你会得到警告：

```text
warning: unresolved link to `Nonexistent`
 --> test.rs:1:24
  |
1 | /// I want to link to [`Nonexistent`] but it doesn't exist!
  |                        ^^^^^^^^^^^^^ no item named `Nonexistent` in `test`
```
当存在歧义时也会得到警告，以及如何消除歧义的建议：

```rust
/// [`Foo`]
pub fn function() {}

pub enum Foo {}

pub fn Foo(){}
```

```text
warning: `Foo` is both an enum and a function
 --> test.rs:1:6
  |
1 | /// [`Foo`]
  |      ^^^^^ ambiguous link
  |
  = note: `#[warn(rustdoc::broken_intra_doc_links)]` on by default
help: to link to the enum, prefix with the item type
  |
1 | /// [`enum@Foo`]
  |      ^^^^^^^^^^
help: to link to the function, add parentheses
  |
1 | /// [`Foo()`]
  |      ^^^^^^^

```

## private_intra_doc_links

这个 lint 是**默认警告**. 这个 lint 会在 [intra-doc links] 从一个公共 item 连接一个私有 item 时提示。
比如：

```rust
#![warn(rustdoc::private_intra_doc_links)] // note: unnecessary - warns by default.

/// [private]
pub fn public() {}
fn private() {}
```
会给出这个文档中链接时损坏的：

```text
warning: public documentation for `public` links to private item `private`
 --> priv.rs:1:6
  |
1 | /// [private]
  |      ^^^^^^^ this item is private
  |
  = note: `#[warn(rustdoc::private_intra_doc_links)]` on by default
  = note: this link will resolve properly if you pass `--document-private-items`
```
注意到取决于你是否传递`--document-private-items`参数会有不同的行为！如果你有私有 item 的文档，尽管会有警告，仍然会生成这个链接：

```text
warning: public documentation for `public` links to private item `private`
 --> priv.rs:1:6
  |
1 | /// [private]
  |      ^^^^^^^ this item is private
  |
  = note: `#[warn(rustdoc::private_intra_doc_links)]` on by default
  = note: this link resolves only because you passed `--document-private-items`, but will break without
```

[intra-doc links]: linking-to-items-by-name.html

## missing_docs

这个 lint **默认允许**。缺少文档时提示。
比如：

```rust
#![warn(missing_docs)]

pub fn undocumented() {}
# fn main() {}
```
`undocumented` 函数会有下面的警告：

```text
warning: missing documentation for a function
  --> your-crate/lib.rs:3:1
   |
 3 | pub fn undocumented() {}
   | ^^^^^^^^^^^^^^^^^^^^^
```
注意不像其他 lint，这个 lint 也对 `rustc` 有效。

## missing_crate_level_docs

这个 lint 是**默认允许**。提示 crate 根没有文档。
比如：

```rust
#![warn(rustdoc::missing_crate_level_docs)]
```
会生成下面的警告：

```text
warning: no documentation found for this crate's top-level module
  |
  = help: The following guide may be of use:
          https://doc.rust-lang.org/nightly/rustdoc/how-to-write-documentation.html
```
当前默认是允许的，但是计划未来默认警告。这可以不使用`missing_docs`这种严重的警告，介绍给新用户如何给他们的 crate 写文档。

## missing_doc_code_examples

这个 lint **默认允许** 并且 **nightly-only**。当文档缺少代码示例时提示。
比如：

```rust
#![warn(rustdoc::missing_doc_code_examples)]

/// There is no code example!
pub fn no_code_example() {}
# fn main() {}
```

`no_code_example` 函数会有下面的警告：

```text
warning: Missing code example in this documentation
  --> your-crate/lib.rs:3:1
   |
LL | /// There is no code example!
   | ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```
为了修复这个 lint，你需要在文档块中加入代码示例：

```rust
/// There is no code example!
///
/// ```
/// println!("calling no_code_example...");
/// no_code_example();
/// println!("we called no_code_example!");
/// ```
pub fn no_code_example() {}
```

## private_doc_tests

这个 lint **默认允许**。 提示私有 item 有文档测试。
比如：

```rust
#![warn(rustdoc::private_doc_tests)]

mod foo {
    /// private doc test
    ///
    /// ```
    /// assert!(false);
    /// ```
    fn bar() {}
}
# fn main() {}
```

提示：

```text
warning: Documentation test in private item
  --> your-crate/lib.rs:4:1
   |
 4 | /     /// private doc test
 5 | |     ///
 6 | |     /// ```
 7 | |     /// assert!(false);
 8 | |     /// ```
   | |___________^
```

## invalid_codeblock_attributes

这个 lint **默认警告**。提示文档例子中的代码块属性有潜在的错误。 
比如：

```rust
#![warn(rustdoc::invalid_codeblock_attributes)]  // note: unnecessary - warns by default.

/// Example.
///
/// ```should-panic
/// assert_eq!(1, 2);
/// ```
pub fn foo() {}
```

提示：

```text
warning: unknown attribute `should-panic`. Did you mean `should_panic`?
 --> src/lib.rs:1:1
  |
1 | / /// Example.
2 | | ///
3 | | /// ```should-panic
4 | | /// assert_eq!(1, 2);
5 | | /// ```
  | |_______^
  |
  = note: `#[warn(rustdoc::invalid_codeblock_attributes)]` on by default
  = help: the code block will either not be tested if not marked as a rust one or won't fail if it doesn't panic when running
```

上面的例子中，正确的拼写是`should_panic`。可以提示一些常用的属性 typo 错误。

## invalid_html_tags

这个 lint **默认允许** 并且是 **nightly-only**。提示没有关闭的或者无效的 HTML tag。
比如：

```rust
#![warn(rustdoc::invalid_html_tags)]

/// <h1>
/// </script>
pub fn foo() {}
```

提示：

```text
warning: unopened HTML tag `script`
 --> foo.rs:1:1
  |
1 | / /// <h1>
2 | | /// </script>
  | |_____________^
  |
  note: the lint level is defined here
 --> foo.rs:1:9
  |
1 | #![warn(rustdoc::invalid_html_tags)]
  |         ^^^^^^^^^^^^^^^^^^^^^^^^^^

warning: unclosed HTML tag `h1`
 --> foo.rs:1:1
  |
1 | / /// <h1>
2 | | /// </script>
  | |_____________^

warning: 2 warnings emitted
```

## invalid_rust_codeblocks

这个 lint **默认警告**。提示文档中的 Rust 代码块无效（比如，空的，无法被解析为 Rust 代码）
比如：

```rust
/// Empty code blocks (with and without the `rust` marker):
///
/// ```rust
/// ```
///
/// Invalid syntax in code blocks:
///
/// ```rust
/// '<
/// ```
pub fn foo() {}
```

提示：

```text
warning: Rust code block is empty
 --> lint.rs:3:5
  |
3 |   /// ```rust
  |  _____^
4 | | /// ```
  | |_______^
  |
  = note: `#[warn(rustdoc::invalid_rust_codeblocks)]` on by default

warning: could not parse code block as Rust code
  --> lint.rs:8:5
   |
8  |   /// ```rust
   |  _____^
9  | | /// '<
10 | | /// ```
   | |_______^
   |
   = note: error from rustc: unterminated character literal
```

## bare_urls

这个 lint **默认警告**。提示 Url 不是一个链接。
比如：

```rust
#![warn(rustdoc::bare_urls)] // note: unnecessary - warns by default.

/// http://example.org
/// [http://example.net]
pub fn foo() {}
```

会出现：

```text
warning: this URL is not a hyperlink
 --> links.rs:1:5
  |
1 | /// http://example.org
  |     ^^^^^^^^^^^^^^^^^^ help: use an automatic link instead: `<http://example.org>`
  |
  = note: `#[warn(rustdoc::bare_urls)]` on by default

warning: this URL is not a hyperlink
 --> links.rs:3:6
  |
3 | /// [http://example.net]
  |      ^^^^^^^^^^^^^^^^^^ help: use an automatic link instead: `<http://example.net>`

warning: 2 warnings emitted
```
