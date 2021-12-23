# watch 命令

当您希望在每次文件更改时呈现您的书时， `watch` 命令很有用。 每次更改文件时，您都可以重复发出 `mdbook build`。 但是使用 `mdbook watch` 将监视您的文件，并在您修改文件时自动触发构建； 这包括重新创建已删除的但在`SUMMARY.md`中但仍然提到的文件！

## 指定目录

`watch` 命令可以将目录作为参数用作书的根目录而不是当前工作目录。

```bash
mdbook watch path/to/book
```

## --open

当您使用 `--open` (`-o`) 选项时，mdbook 将在您的默认 Web 浏览器中打开渲染的书籍。

## --dest-dir

`--dest-dir` (`-d`) 选项允许您更改书籍的输出目录。 相对路径是相对于书的根目录解释的。 如果未指定，它将默认为 `book.toml` 中 `build.build-dir` 键的值，或为 `./book`。

## 指定排除模式

`watch` 命令不会自动为 `book` 根目录中的 `.gitignore` 文件中列出的文件触发构建。 `.gitignore` 文件可能包含 [gitignore
文档](https://git-scm.com/docs/gitignore) 文档中描述的文件模式。 这对于忽略某些编辑器创建的临时文件很有用。

***注意**: 仅使用书籍根目录中的 `.gitignore`文件。 全局的 `$HOME/.gitignore` 或 父目录中的`.gitignore` 文件则不会被使用。*
