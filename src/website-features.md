# Website 特性

这些特性是关于使用`rustdoc`生成的网站。

## 自定义搜索引擎

如果你经常参考在线 Rust 文档，你会喜欢使用一个自定义搜索引擎。这能让你使用导航地址栏直接搜索 `rustdoc` 网站。大多数浏览器支持这种特性，让你来定义一个包含 `%S`（你搜索的内容）的 URL 模版，比如对于标准库你可以使用这个模板：

```text
https://doc.rust-lang.org/stable/std/?search=%s
```

注意这会列出你搜索到的所有结果。如果你想要直接打开搜索到的第一个结果可以使用下面的模板：

```text
https://doc.rust-lang.org/stable/std/?search=%s&go_to_first=true
```

这个 URL 在末尾加入了`go_to_first=true`搜索参数，会自动跳转到第一个搜索结果。
