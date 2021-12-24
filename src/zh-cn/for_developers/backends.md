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

When it loads a book into memory, `mdbook` will inspect your `book.toml` file to
try and figure out which backends to use by looking for all `output.*` tables.
If none are provided it'll fall back to using the default HTML renderer.

Notably, this means if you want to add your own custom backend you'll also need
to make sure to add the HTML backend, even if its table just stays empty.

Now you just need to build your book like normal, and everything should *Just
Work*.

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

The reason we didn't need to specify the full name/path of our `wordcount`
backend is because `mdbook` will try to *infer* the program's name via
convention. The executable for the `foo` backend is typically called
`mdbook-foo`, with an associated `[output.foo]` entry in the `book.toml`. To
explicitly tell `mdbook` what command to invoke (it may require command-line
arguments or be an interpreted script), you can use the `command` field.

```diff
  [book]
  title = "mdBook Documentation"
  description = "Create book from markdown files. Like Gitbook but implemented in Rust"
  authors = ["Mathieu David", "Michael-F-Bryan"]

  [output.html]

  [output.wordcount]
+ command = "python /path/to/wordcount.py"
```

## Configuration

Now imagine you don't want to count the number of words on a particular chapter
(it might be generated text/code, etc). The canonical way to do this is via the
usual `book.toml` configuration file by adding items to your `[output.foo]`
table.

The `Config` can be treated roughly as a nested hashmap which lets you call
methods like `get()` to access the config's contents, with a
`get_deserialized()` convenience method for retrieving a value and automatically
deserializing to some arbitrary type `T`.

To implement this, we'll create our own serializable `WordcountConfig` struct
which will encapsulate all configuration for this backend.

First add `serde` and `serde_derive` to your `Cargo.toml`,

```
cargo add serde serde_derive
```

And then you can create the config struct,

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

Now we just need to deserialize the `WordcountConfig` from our `RenderContext`
and then add a check to make sure we skip ignored chapters.

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

## Output and Signalling Failure

While it's nice to print word counts to the terminal when a book is built, it
might also be a good idea to output them to a file somewhere. `mdbook` tells a
backend where it should place any generated output via the `destination` field
in [`RenderContext`].

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

> **Note:** There is no guarantee that the destination directory exists or is
> empty (`mdbook` may leave the previous contents to let backends do caching),
> so it's always a good idea to create it with `fs::create_dir_all()`.
>
> If the destination directory already exists, don't assume it will be empty.
> To allow backends to cache the results from previous runs, `mdbook` may leave
> old content in the directory.

There's always the possibility that an error will occur while processing a book
(just look at all the `unwrap()`'s we've written already), so `mdbook` will
interpret a non-zero exit code as a rendering failure.

For example, if we wanted to make sure all chapters have an *even* number of
words, erroring out if an odd number is encountered, then you may do something
like this:

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

Now, if we reinstall the backend and build a book,

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

As you've probably already noticed, output from the plugin's subprocess is
immediately passed through to the user. It is encouraged for plugins to follow
the "rule of silence" and only generate output when necessary (e.g. an error in
generation or a warning).

All environment variables are passed through to the backend, allowing you to use
the usual `RUST_LOG` to control logging verbosity.

## Handling missing backends

If you enable a backend that isn't installed, the default behavior is to throw an error:

```text
The command `mdbook-wordcount` wasn't found, is the "wordcount" backend installed?
If you want to ignore this error when the "wordcount" backend is not installed,
set `optional = true` in the `[output.wordcount]` section of the book.toml configuration file.
```

This behavior can be changed by marking the backend as optional.

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

This demotes the error to a warning, and it will instead look like this:

```text
The command was not found, but was marked as optional.
    Command: wordcount
```

## Wrapping Up

Although contrived, hopefully this example was enough to show how you'd create
an alternative backend for `mdbook`. If you feel it's missing something, don't
hesitate to create an issue in the [issue tracker] so we can improve the user
guide.

The existing backends mentioned towards the start of this chapter should serve
as a good example of how it's done in real life, so feel free to skim through
the source code or ask questions.

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
