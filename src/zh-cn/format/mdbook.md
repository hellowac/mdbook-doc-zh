# mdBook 特定功能

## 隐藏代码行

mdBook 中有一个功能可以让你通过在代码行前面加上 `#` 来隐藏代码行，就像你在 [Rustdoc][rustdoc-hide] 中所做的那样。

[rustdoc-hide]: https://doc.rust-lang.org/stable/rustdoc/documentation-tests.html#hiding-portions-of-the-example

```bash
# fn main() {
    let x = 5;
    let y = 6;

    println!("{}", x + y);
# }
```

会渲染成

```rust
# fn main() {
    let x = 5;
    let y = 6;

    println!("{}", x + y);
# }
```

## 包含文件

使用以下语法，您可以将文件包含到您的书中：

```hbs
\{{#include file.rs}}
```

文件路径必须相对于当前源文件。

mdBook 会将包含的文件解释为 Markdown。
由于 include 命令通常用于插入代码片段和示例，因此您通常会用 ```` ``` ```` 包裹该命令以显示文件内容而不解释它们。

````hbs
```
\{{#include file.rs}}
```
````

## 包含文件的一部分

通常您只需要文件的特定部分，例如 相关行举例。 我们支持四种不同的部分模式包括：

```hbs
\{{#include file.rs:2}}
\{{#include file.rs::10}}
\{{#include file.rs:2:}}
\{{#include file.rs:2:10}}
```

第一个命令只包含文件 `file.rs` 的第二行。
第二个命令包括到第 10 行的所有行，即从 11 到文件末尾的行被省略。
第三个命令包括第 2 行的所有行，即省略第一行。
最后一个命令包括 `file.rs` 的摘录，由第 2 行到第 10 行组成。

为了避免在修改包含的文件时破坏您的书籍，您还可以使用锚点而不是行号来包含特定部分。
锚点是一对匹配的线。 锚点开始的行必须与正则表达式 `ANCHOR:\s*[\w_-]+` 匹配，类似地，结束行必须与正则表达式 `ANCHOR_END:\s*[\w_-]+` 匹配。
这允许您在任何类型的注释行中放置锚点。

考虑包含以下文件：

```rs
/* ANCHOR: all */

// ANCHOR: component
struct Paddle {
    hello: f32,
}
// ANCHOR_END: component

////////// ANCHOR: system
impl System for MySystem { ... }
////////// ANCHOR_END: system

/* ANCHOR_END: all */
```

然后在书中，你所要做的就是：

````hbs
Here is a component:
```rust,no_run,noplayground
\{{#include file.rs:component}}
```

Here is a system:
```rust,no_run,noplayground
\{{#include file.rs:system}}
```

This is the full file.
```rust,no_run,noplayground
\{{#include file.rs:all}}
```
````

包含在锚点内的锚点匹配模式的行将被忽略。

## 包含一个文件，但隐藏最初指定行之外的所有内容

`rustdoc_include` helper 用于包含来自外部 Rust 文件的代码，这些文件包含完整的示例，但最初仅以与 `include` 相同的方式显示用行号或锚点指定的特定行。

不在行号范围内或锚点之间的行仍将包含在内，但它们将以 `#` 开头。 这样，读者可以展开代码片段以查看完整示例，而 Rustdoc 将在您运行 `mdbook test` 时使用完整示例。

例如，考虑一个名为 `file.rs` 的文件，其中包含这个 Rust 程序：

```rust
fn main() {
    let x = add_one(2);
    assert_eq!(x, 3);
}

fn add_one(num: i32) -> i32 {
    num + 1
}
```

我们可以使用以下语法包含一个最初仅显示第 2 行的代码段：

````hbs
调用 `add_one` 函数, 我们传入一个 `i32` 并且将返回值赋值给 `x`:

```rust
\{{#rustdoc_include file.rs:2}}
```
````

这与我们手动插入代码并使用`#`隐藏除第 2 行之外的所有代码具有相同的效果：

````hbs
调用 `add_one` 函数, 我们传入一个 `i32` 并且将返回值赋值给 `x`:

```rust
# fn main() {
    let x = add_one(2);
#     assert_eq!(x, 3);
# }
#
# fn add_one(num: i32) -> i32 {
#     num + 1
# }
```
````

也就是说，它看起来像这样（单击“expand”图标以查看文件的其余部分）：

```rust
# fn main() {
    let x = add_one(2);
#     assert_eq!(x, 3);
# }
#
# fn add_one(num: i32) -> i32 {
#     num + 1
# }
```

## 插入可运行的 Rust 文件

使用以下语法，您可以将可运行的 Rust 文件插入到您的书中：

```hbs
\{{#playground file.rs}}
```

Rust 文件的路径必须**相对于当前源文件**。

当点击 play 时，代码片段将被发送到 [Rust Playground] 进行编译和运行。 结果被送回并直接显示在代码下方。

下面是渲染的代码片段的样子：

{{#playground example.rs}}

[Rust Playground]: https://play.rust-lang.org/

## 控制页面 \<title\>

A chapter can set a \<title\> that is different from its entry in the table of contents (sidebar) by including a `\{{#title ...}}` near the top of the page.

通过在页面顶部附近包含一个`\{{#title ...}}`，章节可以设置一个\<title\>，该\<title\> 与其在目录（侧边栏）中的条目不同。

```hbs
\{{#title My Title}}
```
