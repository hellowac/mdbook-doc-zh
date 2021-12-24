# 开发者

虽然 `mdbook` 主要用作命令行工具，但您也可以直接导入底层库并使用它来管理书籍。
它还具有相当灵活的插件机制，如果您需要对本书进行一些分析或以不同的格式呈现它，则允许您创建自己的自定义工具和使用者（通常称为**后端**）。

*开发者* 章节在这里向您展示 `mdbook` 的更高级用法。

开发人员可以通过两种主要方式与本书的构建过程挂钩，

- [预处理器](preprocessors.md)
- [替换后端](backends.md)

## 构建过程

渲染书籍项目的过程需要经过几个步骤。

1. 加载 book
    - 解析 `book.toml`, 如果不存在，则回退到默认的 `Config`
    - 加载 book 章节到内存
    - 发现应该使用哪些预处理器/后端
2. 运行预处理器
3. 依次调用各个后端

## 使用 `mdbook` 作为库

`mdbook` 二进制文件只是 `mdbook crate` 的包装器，将其功能作为命令行程序公开。 因此，很容易创建自己的程序，在内部使用 `mdbook` ，添加自己的功能（例如自定义预处理器）或调整构建过程。

了解如何使用 `mdbook crate` 的最简单方法是查看 [API 文档][API Docs]。 顶级文档解释了如何使用 [`MDBook`] 类型加载和构建一本书，而 [config] 模块对配置系统给出了很好的解释。

[`MDBook`]: https://docs.rs/mdbook/*/mdbook/book/struct.MDBook.html
[API Docs]: https://docs.rs/mdbook/*/mdbook/
[config]: https://docs.rs/mdbook/*/mdbook/config/index.html
