# init 命令

每本新书都有一些相同的最小模板。 为此，mdBook 包含了一个 `init` 命令。

`init` 命令的用法如下：

```bash
mdbook init
```

第一次使用 `init` 命令时，会为你设置几个文件：

```bash
book-test/
├── book
└── src
    ├── chapter_1.md
    └── SUMMARY.md
```

- `src` 目录是你用 Markdown 写书的地方。 它包含所有源文件、配置文件等。
- `book` 目录是你的书被渲染的地方。 所有输出都已准备好上传到服务器以供观众查看。
- `SUMMARY.md` 是本书的骨架，将在[另一章](../format/summary.md)中进行更详细的讨论。

**提示**：从 SUMMARY.md 生成章节

当一个`SUMMARY.md`文件已经存在时，`init`命令会首先解析它并根据`SUMMARY.md`中使用的路径生成不存在的文件。 这允许您思考和创建您的书的整个结构，然后让 mdBook 为您生成它。

## 指定目录

`init` 命令可以将目录作为参数用作书的根目录而不是当前工作目录。

```bash
mdbook init path/to/book
```

## --theme

当您使用 `--theme` 参数时，默认主题将被复制到源目录中名为 `theme` 的目录中，以便您可以对其进行修改。

主题被选择性覆盖，这意味着如果您不想覆盖特定文件，只需将其删除即可使用默认文件。

## --title

指定书名。 如果未提供，交互式提示将要求提供标题。

```bash
mdbook init --title="my amazing book"
```

## --ignore

创建一个 `.gitignore` 文件，配置为忽略[构建][building]书籍时创建的 `book` 目录。 如果未提供，交互式提示将询问是否应创建。

[building]: build.md
