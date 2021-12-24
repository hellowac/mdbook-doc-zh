# 常规配置

您可以在 ***book.toml*** 文件中为您的书配置参数。

下面是一个 ***book.toml*** 文件的示例：

```toml
[book]
title = "Example book"
author = "John Doe"
description = "The example book covers examples."

[rust]
edition = "2018"

[build]
build-dir = "my-example-book"
create-missing = false

[preprocessor.index]

[preprocessor.links]

[output.html]
additional-css = ["custom.css"]

[output.html.search]
limit-results = 15
```

## 支持的配置选项

需要注意的是，配置中指定的**任何**相对路径将始终相对于配置文件所在的书的根目录。

### General metadata

这是关于您的书的常规信息。

- **title:** 书名
- **authors:** 本书的作者
- **description:** 这本书的描述，作为元信息添加到每个页面的html`<head>`中
- **src:** 默认情况下，源目录位于根文件夹下名`src`的目录中。 但这可以使用配置文件中的 `src` 键进行配置。
- **language:** 本书的主要语言，例如用作语言属性`<html lang="en">`。

**book.toml**&nbsp;

```toml
[book]
title = "Example book"
authors = ["John Doe", "Jane Doe"]
description = "The example book covers examples."
src = "my-src"  # the source files will be found in `root/my-src` instead of `root/src`
language = "en"
```

### Rust 选项

Rust 语言的选项，与运行测试和运行环境集成相关。

- **edition**: 默认情况下用于代码片段的 Rust 版本。 默认值为“2015”。 可以使用`edition2015`、`edition2018`或`edition2021`注释来控制单个代码块，例如：

~~~text
```rust,edition2015
// This only works in 2015.
let try = true;
```
~~~

### build 选项

这控制着书的构建过程。

- **build-dir:** 放置渲染书籍的目录。默认情况下，这是书籍根目录中的`book/`。
- **create-missing:** 默认情况下，在构建本书时将创建在`SUMMARY.md`中指定的任何缺失文件（即`create-missing = true`）。 如果值是`false`，那么如果任何文件不存在，构建过程将改为退出并显示错误。
- **use-default-preprocessors:** 通过将此选项设置为`false`来禁用（`links` 和`index`）默认的预处理器。

  如果您通过其配置表声明了相同的和/或其他预处理器，它们将替换默认的预处理运行。

  - 为清楚起见，在没有预处理器配置的情况下，将运行默认的 `links` 和 `index`。
  - 设置 `use-default-preprocessors = false` 将禁止这些默认预处理器运行。
  - 例如，添加 `[preprocessor.links]` 将确保无论 `use-default-preprocessors` 如何， `links` 都将运行。
