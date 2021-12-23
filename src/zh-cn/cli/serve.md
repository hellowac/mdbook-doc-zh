# serve 命令

默认情况下， `serve` 命令用于通过 HTTP 在 `localhost:3000` 提供服务来预览一本书：

```bash
mdbook serve
```

`serve` 命令监视书的 `src` 目录的变化，为每次变化重建书和刷新客户端； 这包括重新创建已删除但仍然在SUMMARY.md中提到的文件！ `websocket` 连接用于触发客户端刷新。

***注意:*** *`serve` 命令用于测试一本书的 HTML 输出，并不打算成为网站的完整 HTTP 服务器。*

## 指定目录

`serve` 命令可以将目录作为参数用作书的根目录而不是当前工作目录。

```bash
mdbook serve path/to/book
```

### Server 可选项

`serve` 主机名默认为 `localhost`，端口默认为 `3000`。 可以在命令行上指定任一选项：

```bash
mdbook serve path/to/book -p 8000 -n 127.0.0.1 
```

#### --open

当您使用 `--open` (`-o`) 标志时，mdbook 将在启动服务器后在您的默认 Web 浏览器中打开该书。

#### --dest-dir

`--dest-dir` (`-d`) 选项允许您更改书籍的输出目录。 相对路径是相对于书的根目录解释的。 如果未指定，它将默认为 `book.toml` 中 `build.build-dir` 键的值，或为 `./book`。

#### 指定排除模式

`watch` 命令不会自动为 `book` 根目录中的 `.gitignore` 文件中列出的文件触发构建。 `.gitignore` 文件可能包含 [gitignore
文档](https://git-scm.com/docs/gitignore) 文档中描述的文件模式。 这对于忽略某些编辑器创建的临时文件很有用。

***注意**: 仅使用书籍根目录中的 `.gitignore`文件。 全局的 `$HOME/.gitignore` 或 父目录中的`.gitignore` 文件则不会被使用。*
