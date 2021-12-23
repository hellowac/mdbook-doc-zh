# test 命令

写书时，有时需要自动化一些测试。 例如，[The Rust Programming Book](https://doc.rust-lang.org/stable/book/) 使用了许多可能会过时的代码示例。 因此，能够自动测试这些代码示例对他们来说非常重要。

mdBook 支持一个`test`命令，该命令将运行书中所有可用的测试。 目前，仅支持 rustdoc 测试，但将来可能会扩展。

## 在一个代码块上禁用测试

rustdoc 不测试包含 `ignore` 属性的代码块：

```rust,ignore
fn main() {}
```

指定非 Rust 语言的代码块 rustdoc 也不测试：

```markdown
**Foo**: _bar_
```

rustdoc 会测试没有指定语言的代码块：

```
This is going to cause an error!
```

## 指定目录

`test` 命令可以将目录作为参数用作书的根目录而不是当前工作目录。

```bash
mdbook test path/to/book
```

## --library-path

`--library-path` (`-L`) 选项允许您将目录添加到 `rustdoc` 在构建和测试示例时使用的库搜索路径。 可以使用多个选项 (`-L foo -L bar`) 或逗号分隔列表 (`-L foo,bar`) 指定多个目录。 该路径应指向包含项目构建输出的 Cargo [构建缓存] deps 目录。 例如，如果您的 Rust 项目的 book 位于名为 `my-book` 的目录中，则以下命令将在运行`test`时包含 crate 的依赖项：

[构建缓存]: https://doc.rust-lang.org/cargo/guide/build-cache.html "build cache"

```shell
mdbook test my-book -L target/debug/deps/
```

有关更多信息，请参阅 `rustdoc` 命令行[文档]。

[文档]: https://doc.rust-lang.org/rustdoc/command-line-arguments.html#-l--library-path-where-to-look-for-dependencies "rustdoc 命令行文档"

## --dest-dir

`--dest-dir` (`-d`) 选项允许您更改书籍的输出目录。 相对路径是相对于书的根目录解释的。 如果未指定，它将默认为 `book.toml` 中 `build.build-dir` 键的值，或为 `./book`。
