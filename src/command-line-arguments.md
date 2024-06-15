# Command-line arguments

这里是可以传递给 `rustdoc` 的参数列表：

## `-h`/`--help`: help

这样使用：

```bash
$ rustdoc -h
$ rustdoc --help
```

这会展示 `rustdoc` 内置的帮助，包含了大量可用的命令行 flags。

有些 flags 是未稳定的；这个页面只会只包好稳定的参数，`--help` 会包含所有的。

## `-V`/`--version`: version information

使用该flag的方式如下：

```bash
$ rustdoc -V
$ rustdoc --version
```

这将显示 `rustdoc` 的版本，如下所示：

```text
rustdoc 1.17.0 (56124baa9 2017-04-24)
```

## `-v`/`--verbose`: more verbose output

使用该flag的方式如下：

```bash
$ rustdoc -v src/lib.rs
$ rustdoc --verbose src/lib.rs
```

这将启用 "冗长模式"，即会将更多信息写入标准输出。
写入的内容取决于你传入的其它标志。
例如，使用 `--version`：

```text
$ rustdoc --verbose --version
rustdoc 1.17.0 (56124baa9 2017-04-24)
binary: rustdoc
commit-hash: hash
commit-date: date
host: host-triple
release: 1.17.0
LLVM version: 3.9
```

## `-o`/`--out-dir`: output directory path

使用该flag的方式如下：

```bash
$ rustdoc src/lib.rs -o target/doc
$ rustdoc src/lib.rs --out-dir target/doc
```

默认情况下，`rustdoc` 的输出会出现在当前工作目录下名为 `doc` 的目录中。
使用此标记后，它将把所有输出到你指定的目录。

## `--crate-name`: controlling the name of the crate

使用该flag的方式如下：

```bash
$ rustdoc src/lib.rs --crate-name mycrate
```

默认情况下，"rustdoc"会假定你的 crate 名称与".rs "文件相同。
您可以使用 `--crate-name` 来覆盖这一假设。

## `--document-private-items`: Show items that are not public

使用该flag的方式如下：

```bash
$ rustdoc src/lib.rs --document-private-items
```

默认情况下，`rustdoc` 只记录可公开访问的项目。

```rust
pub fn public() {} // this item is public and will be documented
mod private { // this item is private and will not be documented
    pub fn unreachable() {} // this item is public, but unreachable, so it will not be documented
}
```

`--document-private-items` documents all items, even if they're not public.

## `-L`/`--library-path`: where to look for dependencies

使用该flag的方式如下：

```bash
$ rustdoc src/lib.rs -L target/debug/deps
$ rustdoc src/lib.rs --library-path target/debug/deps
```

如果你的 crate 有依赖库，`rustdoc` 需要知道在哪里可以找到它们。
传递 `--library-path` 会给 `rustdoc` 提供一个查找这些依赖项的列表。

这个标志可以接受任意数量的目录作为参数，并在搜索时使用所有的目录。

## `--cfg`: passing configuration flags

使用该flag的方式如下：

```bash
$ rustdoc src/lib.rs --cfg feature="foo"
```

该flag接受与 `rustc --cfg` 相同的值，并用它来配置编译。
上面的例子使用了 `feature`，但任何 `cfg` 值都可以接受。

## `--extern`: specify a dependency's location

使用该flag的方式如下：

```bash
$ rustdoc src/lib.rs --extern lazy-static=/path/to/lazy-static
```

与"--library-path "类似，"--extern "是关于指定的位置。
library-path "提供了搜索目录，而"--extern则可以让你精
确地指定依赖项的位置。

## `-C`/`--codegen`: pass codegen options to rustc

使用该flag的方式如下：

```bash
$ rustdoc src/lib.rs -C target_feature=+avx
$ rustdoc src/lib.rs --codegen target_feature=+avx

$ rustdoc --test src/lib.rs -C target_feature=+avx
$ rustdoc --test src/lib.rs --codegen target_feature=+avx

$ rustdoc --test README.md -C target_feature=+avx
$ rustdoc --test README.md --codegen target_feature=+avx
```

当 rustdoc 生成文档、查找文档测试或执行文档测试时，它需要编译一些 rust 代码，至少是部分编译。
测试时，需要编译一些 rust 代码，至少是部分编译。这个flag允许你告诉 rustdoc
在运行这些编译时向 rustc 提供一些额外的 codegen 选项。
大多数情况下这些选项不会影响正常的文档运行，但如果某些内容依赖于目标特性
或者文档测试需要使用一些额外的选项，这个flag允许你影响。

该标志的参数与 rustc 的 `-C` 标志相同。运行 `rustc -C help` 以
获得完整的列表。

## `--test`: run code examples as tests

使用该flag的方式如下：

```bash
$ rustdoc src/lib.rs --test
```

该flag将把你的代码示例作为测试运行。更多信息，请参阅(document-tests.md)。

另请参阅 `--test-args`。

## `--test-args`: pass options to test runner

使用该flag的方式如下：

```bash
$ rustdoc src/lib.rs --test --test-args ignored
```

运行文档测试时，该flag将向测试运行器传递选项。
更多信息，请参阅(document-tests.md)。

另请参阅 `--test`。

## `--target`: generate documentation for the specified target triple

使用该flag的方式如下：

```bash
$ rustdoc src/lib.rs --target x86_64-pc-windows-gnu
```

与 `rustc` 的 `--target` 标志类似，它会为与主机三元组不同的目标三元组生成文档。

交叉编译代码的所有常见注意事项均适用。

## `--default-theme`: set the default theme

Using this flag looks like this:

```bash
$ rustdoc src/lib.rs --default-theme=ayu
```

Sets the default theme (for users whose browser has not remembered a
previous theme selection from the on-page theme picker).

The supplied value should be the lowercase version of the theme name.
The set of available themes can be seen in the theme picker in the
generated output.

Note that the set of available themes - and their appearance - is not
necessarily stable from one rustdoc version to the next. If the
requested theme does not exist, the builtin default (currently
`light`) is used instead.

## `--markdown-css`: include more CSS files when rendering markdown

Using this flag looks like this:

```bash
$ rustdoc README.md --markdown-css foo.css
```

When rendering Markdown files, this will create a `<link>` element in the
`<head>` section of the generated HTML. For example, with the invocation above,

```html
<link rel="stylesheet" type="text/css" href="foo.css" />
```

will be added.

When rendering Rust files, this flag is ignored.

## `--html-in-header`: include more HTML in <head>

Using this flag looks like this:

```bash
$ rustdoc src/lib.rs --html-in-header header.html
$ rustdoc README.md --html-in-header header.html
```

This flag takes a list of files, and inserts them into the `<head>` section of
the rendered documentation.

## `--html-before-content`: include more HTML before the content

Using this flag looks like this:

```bash
$ rustdoc src/lib.rs --html-before-content extra.html
$ rustdoc README.md --html-before-content extra.html
```

This flag takes a list of files, and inserts them inside the `<body>` tag but
before the other content `rustdoc` would normally produce in the rendered
documentation.

## `--html-after-content`: include more HTML after the content

Using this flag looks like this:

```bash
$ rustdoc src/lib.rs --html-after-content extra.html
$ rustdoc README.md --html-after-content extra.html
```

This flag takes a list of files, and inserts them before the `</body>` tag but
after the other content `rustdoc` would normally produce in the rendered
documentation.

## `--markdown-playground-url`: control the location of the playground

Using this flag looks like this:

```bash
$ rustdoc README.md --markdown-playground-url https://play.rust-lang.org/
```

When rendering a Markdown file, this flag gives the base URL of the Rust
Playground, to use for generating `Run` buttons.

## `--markdown-no-toc`: don't generate a table of contents

Using this flag looks like this:

```bash
$ rustdoc README.md --markdown-no-toc
```

When generating documentation from a Markdown file, by default, `rustdoc` will
generate a table of contents. This flag suppresses that, and no TOC will be
generated.

## `-e`/`--extend-css`: extend rustdoc's CSS

Using this flag looks like this:

```bash
$ rustdoc src/lib.rs -e extra.css
$ rustdoc src/lib.rs --extend-css extra.css
```

With this flag, the contents of the files you pass are included at the bottom
of Rustdoc's `theme.css` file.

While this flag is stable, the contents of `theme.css` are not, so be careful!
Updates may break your theme extensions.

## `--sysroot`: override the system root

Using this flag looks like this:

```bash
$ rustdoc src/lib.rs --sysroot /path/to/sysroot
```

Similar to `rustc --sysroot`, this lets you change the sysroot `rustdoc` uses
when compiling your code.

### `--edition`: control the edition of docs and doctests

Using this flag looks like this:

```bash
$ rustdoc src/lib.rs --edition 2018
$ rustdoc --test src/lib.rs --edition 2018
```

This flag allows `rustdoc` to treat your rust code as the given edition. It will compile doctests with
the given edition as well. As with `rustc`, the default edition that `rustdoc` will use is `2015`
(the first edition).

## `--theme`: add a theme to the documentation output

Using this flag looks like this:

```bash
$ rustdoc src/lib.rs --theme /path/to/your/custom-theme.css
```

`rustdoc`'s default output includes two themes: `light` (the default) and
`dark`. This flag allows you to add custom themes to the output. Giving a CSS
file to this flag adds it to your documentation as an additional theme choice.
The theme's name is determined by its filename; a theme file named
`custom-theme.css` will add a theme named `custom-theme` to the documentation.

## `--check-theme`: verify custom themes against the default theme

Using this flag looks like this:

```bash
$ rustdoc --check-theme /path/to/your/custom-theme.css
```

While `rustdoc`'s HTML output is more-or-less consistent between versions, there
is no guarantee that a theme file will have the same effect. The `--theme` flag
will still allow you to add the theme to your documentation, but to ensure that
your theme works as expected, you can use this flag to verify that it implements
the same CSS rules as the official `light` theme.

`--check-theme` is a separate mode in `rustdoc`. When `rustdoc` sees the
`--check-theme` flag, it discards all other flags and only performs the CSS rule
comparison operation.

### `--crate-version`: control the crate version

Using this flag looks like this:

```bash
$ rustdoc src/lib.rs --crate-version 1.3.37
```

When `rustdoc` receives this flag, it will print an extra "Version (version)" into the sidebar of
the crate root's docs. You can use this flag to differentiate between different versions of your
library's documentation.

## `@path`: load command-line flags from a path

If you specify `@path` on the command-line, then it will open `path` and read
command line options from it. These options are one per line; a blank line indicates
an empty option. The file can use Unix or Windows style line endings, and must be
encoded as UTF-8.

## `--passes`: add more rustdoc passes

This flag is **deprecated**.
For more details on passes, see [the chapter on them](passes.md).

## `--no-defaults`: don't run default passes

This flag is **deprecated**.
For more details on passes, see [the chapter on them](passes.md).

## `-r`/`--input-format`: input format

This flag is **deprecated** and **has no effect**.

Rustdoc only supports Rust source code and Markdown input formats. If the
file ends in `.md` or `.markdown`, `rustdoc` treats it as a Markdown file.
Otherwise, it assumes that the input file is Rust.

## `--nocapture`

When this flag is used with `--test`, the output (stdout and stderr) of your tests won't be
captured by rustdoc. Instead, the output will be directed to your terminal,
as if you had run the test executable manually. This is especially useful
for debugging your tests!
