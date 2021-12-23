# 指南

**mdBook** 是一个命令行工具和 Rust crate，用于使用 Markdown 创建书籍。
输出类似于 Gitbook 等工具，非常适合创建产品或 API 文档、教程、课程材料或任何需要干净、易于导航和可自定义的演示文稿的内容。
mdBook 是用 [Rust](https://www.rust-lang.org) 编写的；
它的性能和简单性使其非常适合用作通过自动化直接发布到托管网站（例如 [GitHub Pages](https://pages.github.com)）的工具。
事实上，本指南既是 mdBook 文档，也是 mdBook 生成内容的一个很好的例子。

mdBook 包括对 Markdown 预处理和用于生成 HTML 以外格式的替代渲染器的内置支持。
这些工具还支持其他功能，例如验证。
[搜索](https://crates.io/search?q=mdbook&sort=relevance) Rust 的 [crates.io](https://crates.io) 是发现更多扩展的好方法。

## API 文档

除了以上特性，mdBook 还有一个 Rust [API](https://docs.rs/mdbook/*/mdbook/)。 这允许您编写自己的预处理器或渲染器，以及将 mdBook 功能合并到其他应用程序中。 本指南的[面向开发人员](for_developers/index.html)部分包含更多信息和一些示例。

## 贡献

mdBook 是免费和开源的。 您可以在 [GitHub](https://github.com/rust-lang/mdBook) 上找到源代码，问题和功能请求可以发布在 [GitHub 问题跟踪器](https://github.com/rust-lang/mdBook/issues)上。 mdBook 依靠社区来修复错误和添加功能：如果您想做出贡献，请阅读 [CONTRIBUTING](<https://github.com/rust-lang/mdBook/blob/master/CONTRIBUTING.md>) 指南并考虑打开[拉取请求](https://github.com/rust-lang/mdBook/pulls)。

## 许可

mdBook 源代码和文档在 [Mozilla Public License v2.0](https://www.mozilla.org/MPL/2.0/) 下发布。
