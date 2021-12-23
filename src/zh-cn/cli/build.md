# build 命令

build 命令用于渲染您的书：

```bash
mdbook build
```

它将尝试解析您的`SUMMARY.md`文件以了解您的书的结构并获取相应的文件。
请注意，在 `SUMMARY.md` 中提到但不存在的文件将会自动创建。

为方便起见，渲染的输出将保持与源相同的目录结构。 大量的books在渲染时将保持结构化。

## 指定目录

`build` 命令可以将目录作为参数用作书的根目录而不是当前工作目录。

```bash
mdbook build path/to/book
```

## --open

当您使用 `--open` (`-o`) 标志时，mdbook 在构建后, 将在你默认的 Web 浏览器中打开渲染的books。

## --dest-dir

`--dest-dir` (`-d`) 选项允许您更改书籍的输出目录。 相对路径是相对于书的根目录解释的。 如果未指定，它将默认为 `book.toml` 中 `build.build-dir` 键的值，或为 `./book`。

-------------------

***注意:*** *build 命令将所有文件（不包括扩展名为 `.md` 的文件）从源目录复制到 `build` 目录中。*
