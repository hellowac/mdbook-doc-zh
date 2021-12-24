# 环境变量

通过设置相应的环境变量，可以从命令行覆盖所有配置值。 由于许多操作系统将环境变量限制为字母数字字符或 `_`，因此配置键的格式需要与正常的 `foo.bar.baz` 格式略有不同。

以 `MDBOOK_` 开头的变量用于配置。 密钥是通过删除 `MDBOOK_` 前缀并将结果字符串转换为 `kebab-case` 来创建的。 双下划线 (`__`) 分隔为嵌套键(`.`)，而单个下划线 (`_`) 则替换为破折号 (`-`)。

例子:

- `MDBOOK_foo` -> `foo`
- `MDBOOK_FOO` -> `foo`
- `MDBOOK_FOO__BAR` -> `foo.bar`
- `MDBOOK_FOO_BAR` -> `foo-bar`
- `MDBOOK_FOO_bar__baz` -> `foo-bar.baz`

因此，通过设置 `MDBOOK_BOOK__TITLE` 环境变量，您可以覆盖书名而无需修改您的 `book.toml`。

> **备注:** 为了方便设置更复杂的配置项，环境变量的值首先被解析为 JSON，如果解析失败则回退到字符串。
>
> 这意味着，如果您愿意，您可以在构建书籍时覆盖所有书籍元数据，例如
>
> ```shell
> $ export MDBOOK_BOOK="{'title': 'My Awesome Book', authors: ['Michael-F-Bryan']}"
> $ mdbook build
> ```

后一种情况在从脚本或 CI 调用 `mdbook` 的情况下可能很有用，适用情况为，有时无法在构建之前更新 `book.toml`。
