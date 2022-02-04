# Passes

Rustdoc 有一个概念叫做 "passes"。在 `rustdoc` 命令最终生成文档之前存在一些转换。

自定义 passes 被**废弃**了，已经可用的 passes 不会稳定可能随时会在发布中更改

过去最常用的自定义 passes 是 `strip-private` pass。你现在可以更容易的做到这个，通过传递参数 [`--document-private-items`](./unstable-features.md#--document-private-items).
