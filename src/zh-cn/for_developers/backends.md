# 替换后端

“后端”只是一个 `mdbook` 在渲染过程中调用的程序。
该程序通过`stdin`传递书籍和配置信息的 JSON 表示。
一旦后端收到此信息，它就可以自由地做任何想做的事。

GitHub 上已经有几个替代后端，可以用作在实践中如何完成的大概示例。

- [mdbook-linkcheck] - 一个用于验证这本书不包含任何损坏的链接的简单程序
- [mdbook-epub] - 一个 EPUB 渲染器
- [mdbook-test] - 一个通过 [rust-skeptic] 运行本书内容的程序，以验证所有内容是否正确编译和运行（类似于 `rustdoc --test`）
- [mdbook-man] - 从书中生成手册页

此页面将引导您以简单的字数统计程序的形式创建您自己的替代后端。 虽然它将用 Rust 编写，但没有理由不能使用 Python 或 Ruby 之类的其他编程语言来完成。

## 起步

首先，您需要创建一个新的二进制程序并添加 `mdbook` 作为依赖项。

```shell
>$ cargo new --bin mdbook-wordcount
>$ cd mdbook-wordcount
>$ cargo add mdbook
```

当我们的 `mdbook-wordcount` 插件被调用时，`mdbook` 将通过我们插件的 `stdin` 向它发送 [`RenderContext`] 的 JSON 版本。 为方便起见，有一个 [`RenderContext::from_json()`] 构造函数，它将加载一个 `RenderContext`。

这是我们的后端加载本书所需的所有样板。

```rust
// src/main.rs
extern crate mdbook;

use std::io;
use mdbook::renderer::RenderContext;

fn main() {
    let mut stdin = io::stdin();
    let ctx = RenderContext::from_json(&mut stdin).unwrap();
}
```

> **注意:** `RenderContext` 包含一个`version`字段。 这让后端可以确定它们是否与它正在调用的 `mdbook` 版本兼容。 该`version`直接来自 mdbook 的 `Cargo.toml` 中的相应字段。

建议后端使用 `semver` crate 检查此字段，并在可能存在兼容性问题时发出警告。

## 核查 Book

现在我们的后端有这本书的副本，让我们数一数每章有多少字！

因为 `RenderContext` 包含一个[`Book`] 字段（`book`），并且`Book` 具有[`Book::iter()`] 方法来迭代`Book` 中的所有项目，这一步 结果证明和第一个一样容易。

```rust

fn main() {
    let mut stdin = io::stdin();
    let ctx = RenderContext::from_json(&mut stdin).unwrap();

    for item in ctx.book.iter() {
        if let BookItem::Chapter(ref ch) = *item {
            let num_words = count_words(ch);
            println!("{}: {}", ch.name, num_words);
        }
    }
}

fn count_words(ch: &Chapter) -> usize {
    ch.content.split_whitespace().count()
}
```

## 启用后端

现在我们已经有了基础运行代码，我们想要实际使用它。 首先，安装程序。

```shell
cargo install --path .
```

然后`cd`到你想要计算单词数的特定书籍目录. 并更新它的`book.toml`文件。

```diff
  [book]
  title = "mdBook Documentation"
  description = "Create book from markdown files. Like Gitbook but implemented in Rust"
  authors = ["Mathieu David", "Michael-F-Bryan"]

+ [output.html]

+ [output.wordcount]
```

当它将一本书加载到内存中时，`mdbook` 将检查您的 `book.toml` 文件，以通过查找所有 `output.*` 表来尝试找出要使用的后端。 如果没有提供，它将回退到使用默认的 HTML 渲染器。

值得注意的是，这意味着如果您想添加自己的自定义后端，您还需要确保添加 HTML 后端，即使它的表只是为空。

现在你只需要像往常一样构建你的书，一切都应该*正常工作*。

```shell
$ mdbook build
...
2018-01-16 07:31:15 [INFO] (mdbook::renderer): Invoking the "mdbook-wordcount" renderer
mdBook: 126
Command Line Tool: 224
init: 283
build: 145
watch: 146
serve: 292
test: 139
Format: 30
SUMMARY.md: 259
Configuration: 784
Theme: 304
index.hbs: 447
Syntax highlighting: 314
MathJax Support: 153
Rust code specific features: 148
For Developers: 788
Alternative Backends: 710
Contributors: 85
```

我们不需要指定 `wordcount` 后端的全名/路径的原因是因为 `mdbook` 将尝试通过约定推断程序的名称。
`foo` 后端的可执行文件通常称为 `mdbook-foo`，在 `book.toml` 中有一个关联的 `[output.foo]` 条目。
要明确告诉 `mdbook` 调用什么命令（它可能需要命令行参数或解释脚本），您可以使用`command`字段。

```diff
  [book]
  title = "mdBook Documentation"
  description = "Create book from markdown files. Like Gitbook but implemented in Rust"
  authors = ["Mathieu David", "Michael-F-Bryan"]

  [output.html]

  [output.wordcount]
+ command = "python /path/to/wordcount.py"
```

## 配置

现在假设您不想计算特定章节的字数（它可能是生成的文本/代码等）。 执行此操作的规范方法是通过配置文件`book.toml` ， 将项目添加到配置文件的 [output.foo] 表中。

`Config` 可以粗略地视为一个嵌套的 **hashmap**，它允许您调用 `get()` 之类的方法来访问配置的内容，使用 `get_deserialized()` 便捷方法检索值并自动反序列化为某个任意类型 `T`。

为了实现这一点，我们将创建我们自己的可序列化 `WordcountConfig` 结构，它将封装此后端的所有配置。

首先将 `serde` 和 `serde_derive` 添加到 `Cargo.toml`，

```bash
cargo add serde serde_derive
```

然后你可以创建配置结构，

```rust
extern crate serde;
#[macro_use]
extern crate serde_derive;

...

#[derive(Debug, Default, Serialize, Deserialize)]
#[serde(default, rename_all = "kebab-case")]
pub struct WordcountConfig {
  pub ignores: Vec<String>,
}
```

现在我们只需要从我们的 `RenderContext` 反序列化 `WordcountConfig`，然后添加一个检查以确保跳过被我们忽略的章节。

```diff
  fn main() {
      let mut stdin = io::stdin();
      let ctx = RenderContext::from_json(&mut stdin).unwrap();
+     let cfg: WordcountConfig = ctx.config
+         .get_deserialized("output.wordcount")
+         .unwrap_or_default();

      for item in ctx.book.iter() {
          if let BookItem::Chapter(ref ch) = *item {
+             if cfg.ignores.contains(&ch.name) {
+                 continue;
+             }
+
              let num_words = count_words(ch);
              println!("{}: {}", ch.name, num_words);
          }
      }
  }
```

## 输出和信令故障

虽然在构建一本书时将字数打印到终端是很好的，但将它们输出到某个文件中也可能是一个好主意。 `mdbook` 告诉后端它应该通过 `RenderContext` 中的`destination`字段将任何生成的输出放置在哪里。

```diff
+ use std::fs::{self, File};
+ use std::io::{self, Write};
- use std::io;
  use mdbook::renderer::RenderContext;
  use mdbook::book::{BookItem, Chapter};

  fn main() {
    ...

+     let _ = fs::create_dir_all(&ctx.destination);
+     let mut f = File::create(ctx.destination.join("wordcounts.txt")).unwrap();
+
      for item in ctx.book.iter() {
          if let BookItem::Chapter(ref ch) = *item {
              ...

              let num_words = count_words(ch);
              println!("{}: {}", ch.name, num_words);
+             writeln!(f, "{}: {}", ch.name, num_words).unwrap();
          }
      }
  }
```

> **注意:** 无法保证目标目录存在或为空（`mdbook` 可能会保留之前的内容让后端进行缓存），因此使用 `fs::create_dir_all()` 创建它总是一个好主意。
>
> 如果目标目录已经存在，不要假设它是空的。 为了允许后端缓存先前运行的结果，`mdbook` 可能会在目录中保留旧内容。

处理书籍时总是有可能发生错误（只需查看我们已经编写的所有 `unwrap()` ），因此 `mdbook` 会将非零退出代码解释为渲染失败。

例如，如果我们想确保所有章节都有偶数个单词，如果遇到奇数就出错，那么你可以这样做：

```diff
+ use std::process;
  ...

  fn main() {
      ...

      for item in ctx.book.iter() {
          if let BookItem::Chapter(ref ch) = *item {
              ...

              let num_words = count_words(ch);
              println!("{}: {}", ch.name, num_words);
              writeln!(f, "{}: {}", ch.name, num_words).unwrap();

+             if cfg.deny_odds && num_words % 2 == 1 {
+               eprintln!("{} has an odd number of words!", ch.name);
+               process::exit(1);
              }
          }
      }
  }

  #[derive(Debug, Default, Serialize, Deserialize)]
  #[serde(default, rename_all = "kebab-case")]
  pub struct WordcountConfig {
      pub ignores: Vec<String>,
+     pub deny_odds: bool,
  }
```

现在，如果我们重新安装后端并构建一本书，

```shell
$ cargo install --path . --force
$ mdbook build /path/to/book
...
2018-01-16 21:21:39 [INFO] (mdbook::renderer): Invoking the "wordcount" renderer
mdBook: 126
Command Line Tool: 224
init: 283
init has an odd number of words!
2018-01-16 21:21:39 [ERROR] (mdbook::renderer): Renderer exited with non-zero return code.
2018-01-16 21:21:39 [ERROR] (mdbook::utils): Error: Rendering failed
2018-01-16 21:21:39 [ERROR] (mdbook::utils):    Caused By: The "mdbook-wordcount" renderer failed
```

您可能已经注意到，插件子进程的输出会立即传递给用户。 鼓励插件遵循“沉默规则”，仅在必要时生成输出（例如生成错误或警告）。

所有环境变量都传递到后端，允许您使用通常的`RUST_LOG`来控制日志记录的详细程度。

## 处理缺失的后端

如果启用未安装的后端，默认行为是抛出错误：

```text
The command `mdbook-wordcount` wasn't found, is the "wordcount" backend installed?
If you want to ignore this error when the "wordcount" backend is not installed,
set `optional = true` in the `[output.wordcount]` section of the book.toml configuration file.
```

可以通过将后端标记为可选来更改此行为。

```diff
  [book]
  title = "mdBook Documentation"
  description = "Create book from markdown files. Like Gitbook but implemented in Rust"
  authors = ["Mathieu David", "Michael-F-Bryan"]

  [output.html]

  [output.wordcount]
  command = "python /path/to/wordcount.py"
+ optional = true
```

这会将错误降级为警告，而是如下所示：

```text
The command was not found, but was marked as optional.
    Command: wordcount
```

## Wrapping Up

虽然人为设计，但希望这个例子足以展示您如何为 `mdbook` 创建替换后端。 如果您觉得缺少某些内容，请不要犹豫在问题跟踪器中[创建问题][issue tracker]，以便我们改进用户指南。

本章开头提到的现有后端应该作为它在现实生活中如何完成的一个很好的例子，所以请随意浏览源代码或提出问题。

[mdbook-linkcheck]: https://github.com/Michael-F-Bryan/mdbook-linkcheck
[mdbook-epub]: https://github.com/Michael-F-Bryan/mdbook-epub
[mdbook-test]: https://github.com/Michael-F-Bryan/mdbook-test
[mdbook-man]: https://github.com/vv9k/mdbook-man
[rust-skeptic]: https://github.com/budziq/rust-skeptic
[`RenderContext`]: https://docs.rs/mdbook/*/mdbook/renderer/struct.RenderContext.html
[`RenderContext::from_json()`]: https://docs.rs/mdbook/*/mdbook/renderer/struct.RenderContext.html#method.from_json
[`semver`]: https://crates.io/crates/semver
[`Book`]: https://docs.rs/mdbook/*/mdbook/book/struct.Book.html
[`Book::iter()`]: https://docs.rs/mdbook/*/mdbook/book/struct.Book.html#method.iter
[`Config`]: https://docs.rs/mdbook/*/mdbook/config/struct.Config.html
[issue tracker]: https://github.com/rust-lang/mdBook/issues
