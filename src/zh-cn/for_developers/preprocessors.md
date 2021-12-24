# 预处理器

**预处理器**只是一小段代码，它在书籍加载后和渲染之前立即运行，允许您更新和修改书籍。 可能的用例是：

- 创建一个自定义 helpers 例如 `\{{#include /path/to/file.md}}`
- 更新链接，以便HTML渲染器渲染时将 `[部分章节1](some_chapter1.md)` 自动更改为 `[部分章节2](some_chapter2.html)`
- 替换 latex 风格的表达式 (`$$ \frac{1}{3} $$`) 为 等价的 mathjax 表达式

## 连接到 MDBook

MDBook 使用一种相当简单的机制来发现第三方插件。
一个新表被添加到 `book.toml`（例如 `foo` 预处理器的 `preprocessor.foo`），
然后 `mdbook` 将尝试调用 `mdbook-foo` 程序作为构建过程的一部分。

预处理器可以硬编码来指定`preprocessor.foo.renderer` 键, 指定他应该运行那些后端。 例如，将 [MathJax](../format/mathjax.md) 用于非 HTML 渲染器是没有意义的。

```toml
[book]
title = "My Book"
authors = ["Michael-F-Bryan"]

[preprocessor.foo]
# The command can also be specified manually
command = "python3 /path/to/foo.py"
# Only run the `foo` preprocessor for the HTML and EPUB renderer
renderer = ["html", "epub"]
```

一旦定义了预处理器并开始构建过程，mdBook 将执行 `preprocessor.foo.command` 键中定义的命令两次。

它第一次运行为预处理器确定它是否支持给定的渲染器。

mdBook 向进程传递两个参数：第一个参数是字符串`supports`，第二个参数是渲染器名称。

如果预处理器支持给定的渲染器，则它应该以状态代码 `0` 退出，如果不支持，则返回非零退出代码。

如果预处理器支持渲染器，则 mdbook 将第二次运行它，且将 JSON 数据传递到 stdin。 JSON 包含一个 `[context, book]` 数组，其中 `context` 是序列化的对象 `PreprocessorContext`，而 `book` 是包含书籍内容的 [`Book`] 对象。

预处理器应该将 [`Book`] 对象的 JSON 格式返回到标准输出，并带有它希望执行的任何修改。

最简单的入门方法是创建您自己的 `Preprocessor` trait 实现（例如在 `lib.rs` 中），
然后创建一个 shell 二进制文件，将输入转换为正确的 `Preprocessor` 方法。
为方便起见，`examples/` 目录中有一个示例 `no-op` 预处理器，它可以很容易地适用于其他预处理器。

<details>
<summary>示例预处理器 no-op </summary>

```rust
// nop-preprocessors.rs

use crate::nop_lib::Nop;
use clap::{App, Arg, ArgMatches, SubCommand};
use mdbook::book::Book;
use mdbook::errors::Error;
use mdbook::preprocess::{CmdPreprocessor, Preprocessor, PreprocessorContext};
use semver::{Version, VersionReq};
use std::io;
use std::process;

pub fn make_app() -> App<'static, 'static> {
    App::new("nop-preprocessor")
        .about("A mdbook preprocessor which does precisely nothing")
        .subcommand(
            SubCommand::with_name("supports")
                .arg(Arg::with_name("renderer").required(true))
                .about("Check whether a renderer is supported by this preprocessor"),
        )
}

fn main() {
    let matches = make_app().get_matches();

    // Users will want to construct their own preprocessor here
    let preprocessor = Nop::new();

    if let Some(sub_args) = matches.subcommand_matches("supports") {
        handle_supports(&preprocessor, sub_args);
    } else if let Err(e) = handle_preprocessing(&preprocessor) {
        eprintln!("{}", e);
        process::exit(1);
    }
}

fn handle_preprocessing(pre: &dyn Preprocessor) -> Result<(), Error> {
    let (ctx, book) = CmdPreprocessor::parse_input(io::stdin())?;

    let book_version = Version::parse(&ctx.mdbook_version)?;
    let version_req = VersionReq::parse(mdbook::MDBOOK_VERSION)?;

    if !version_req.matches(&book_version) {
        eprintln!(
            "Warning: The {} plugin was built against version {} of mdbook, \
             but we're being called from version {}",
            pre.name(),
            mdbook::MDBOOK_VERSION,
            ctx.mdbook_version
        );
    }

    let processed_book = pre.run(&ctx, book)?;
    serde_json::to_writer(io::stdout(), &processed_book)?;

    Ok(())
}

fn handle_supports(pre: &dyn Preprocessor, sub_args: &ArgMatches) -> ! {
    let renderer = sub_args.value_of("renderer").expect("Required argument");
    let supported = pre.supports_renderer(renderer);

    // Signal whether the renderer is supported by exiting with 1 or 0.
    if supported {
        process::exit(0);
    } else {
        process::exit(1);
    }
}

/// The actual implementation of the `Nop` preprocessor. This would usually go
/// in your main `lib.rs` file.
mod nop_lib {
    use super::*;

    /// A no-op preprocessor.
    pub struct Nop;

    impl Nop {
        pub fn new() -> Nop {
            Nop
        }
    }

    impl Preprocessor for Nop {
        fn name(&self) -> &str {
            "nop-preprocessor"
        }

        fn run(&self, ctx: &PreprocessorContext, book: Book) -> Result<Book, Error> {
            // In testing we want to tell the preprocessor to blow up by setting a
            // particular config value
            if let Some(nop_cfg) = ctx.config.get_preprocessor(self.name()) {
                if nop_cfg.contains_key("blow-up") {
                    anyhow::bail!("Boom!!1!");
                }
            }

            // we *are* a no-op preprocessor after all
            Ok(book)
        }

        fn supports_renderer(&self, renderer: &str) -> bool {
            renderer != "not-supported"
        }
    }
}
```

</details>

## 实现预处理器的提示

通过将 `mdbook` 作为库引入，预处理器可以访问现有的基础架构来处理书籍。

例如，自定义预处理器可以使用 [`CmdPreprocessor::parse_input()`] 函数反序列化写入`stdin`的 JSON。 然后可以通过 [`Book::for_each_mut()`] 就地修改 `Book` 的每一章，然后使用 `serde_json` crate 写入 `stdout` 。

章节可以直接访问（通过递归迭代章节）或通过 [`Book::for_each_mut()`] 便捷方法访问。

该 `chapter.content` 只是一个字符串，恰好是 Markdown。 虽然完全可以使用正则表达式或进行手动查找和替换，但您可能希望将输入处理为对计算机更友好的内容。 [`pulldown-cmark`][pc] crate 实现了一个生产质量的基于事件的 Markdown 解析器，使用 [`pulldown-cmark-to-cmark`][pctc] 允许您将事件转换回 Markdown 文本。

以下代码块显示了如何从 Markdown 中删除所有强调，而不会意外破坏文档。

```rust
fn remove_emphasis(
    num_removed_items: &mut usize,
    chapter: &mut Chapter,
) -> Result<String> {
    let mut buf = String::with_capacity(chapter.content.len());

    let events = Parser::new(&chapter.content).filter(|e| {
        let should_keep = match *e {
            Event::Start(Tag::Emphasis)
            | Event::Start(Tag::Strong)
            | Event::End(Tag::Emphasis)
            | Event::End(Tag::Strong) => false,
            _ => true,
        };
        if !should_keep {
            *num_removed_items += 1;
        }
        should_keep
    });

    cmark(events, &mut buf, None).map(|_| buf).map_err(|err| {
        Error::from(format!("Markdown serialization failed: {}", err))
    })
}
```

对于其他所有内容，请查看[完整示例][example]。

## 用不同的语言实现预处理器

mdBook 利用 `stdin` 和 `stdout` 与预处理器通信的事实使得用 Rust 以外的语言实现它们变得容易。

下面的代码展示了如何在Python中实现一个简单的预处理器，将修改第一章的内容。

下面的示例遵循上面显示的配置，其中 `preprocessor.foo.command` 实际上指向一个 Python 脚本。

```python
import json
import sys


if __name__ == '__main__':
    if len(sys.argv) > 1: # 我们检查我们是否收到任何参数
        if sys.argv[1] == "supports": 
            # 那么我们最好返回 0 的退出状态代码，因为另一个参数将只是渲染器的名称
            sys.exit(0)

    # 从标准输入加载上下文和书籍表示
    context, book = json.load(sys.stdin)
    # 现在，我们可以修改第一章的内容
    book['sections'][0]['Chapter']['content'] = '# Hello'
    # 我们完成了本书的修改，我们可以将它打印到标准输出，
    print(json.dumps(book))
```

[preprocessor-docs]: https://docs.rs/mdbook/latest/mdbook/preprocess/trait.Preprocessor.html
[pc]: https://crates.io/crates/pulldown-cmark
[pctc]: https://crates.io/crates/pulldown-cmark-to-cmark
[example]: https://github.com/rust-lang/mdBook/blob/master/examples/nop-preprocessor.rs
[an example no-op preprocessor]: https://github.com/rust-lang/mdBook/blob/master/examples/nop-preprocessor.rs
[`CmdPreprocessor::parse_input()`]: https://docs.rs/mdbook/latest/mdbook/preprocess/trait.Preprocessor.html#method.parse_input
[`Book::for_each_mut()`]: https://docs.rs/mdbook/latest/mdbook/book/struct.Book.html#method.for_each_mut
[`PreprocessorContext`]: https://docs.rs/mdbook/latest/mdbook/preprocess/struct.PreprocessorContext.html
[`Book`]: https://docs.rs/mdbook/latest/mdbook/book/struct.Book.html
