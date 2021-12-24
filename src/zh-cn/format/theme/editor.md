# 编辑器

除了提供可运行的代码环境外，`mdBook` 还允许它们可编辑。 为了启用可编辑的代码块，需要在 `book.toml` 中添加以下内容：

```toml
[output.html.playground]
editable = true
```

要使特定块可用于编辑，需要向其添加属性`editable`：

<pre><code class="language-markdown">```rust,editable
fn main() {
    let number = 5;
    print!("{}", number);
}
```</code></pre>

以上将导致这个可编辑和运行的代码块：

```rust,editable
fn main() {
    let number = 5;
    print!("{}", number);
}
```

请注意代码块中的 `撤销更改` 按钮。

## 自定义编辑器

默认情况下，编辑器是 [Ace](https://ace.c9.io/) 编辑器，但如果需要，可以通过提供不同的文件夹来覆盖该功能：

```toml
[output.html.playground]
editable = true
editor = "/path/to/editor"
```

请注意，为了使编辑器更改正常运行，需要覆盖 `theme` 文件夹内的 `book.js`，因为它与默认的 Ace 编辑器有一些耦合。
