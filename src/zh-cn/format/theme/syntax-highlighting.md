# 语法高亮

mdBook 使用具有自定义主题的 [Highlight.js] 来突出显示语法。

[Highlight.js]: https://highlightjs.org

自动语言检测已关闭，因此您可能希望像这样指定您使用的编程语言：

~~~markdown
```rust
fn main() {
    // Some code
}
```
~~~

## 支持的语言

默认情况下支持这些语言，但您可以通过提供自己的 `highlight.js` 文件来添加更多语言：

- apache
- armasm
- bash
- c
- coffeescript
- cpp
- csharp
- css
- d
- diff
- go
- handlebars
- haskell
- http
- ini
- java
- javascript
- json
- julia
- kotlin
- less
- lua
- makefile
- markdown
- nginx
- objectivec
- perl
- php
- plaintext
- properties
- python
- r
- ruby
- rust
- scala
- scss
- shell
- sql
- swift
- typescript
- x86asm
- xml
- yaml

## Custom theme

与主题的其余部分一样，用于语法突出显示的文件可以用您自己的文件覆盖。

- ***highlight.js*** 通常您不必覆盖此文件，除非您想使用更新的版本。
- ***highlight.css*** highlight.js 用于语法高亮显示的主题。

如果你想为 `highlight.js` 使用另一个主题，请从他们的网站下载它，或者自己制作，将其重命名为 `highlight.css` 并将其放在你的书的 `theme` 文件夹中。

现在将使用您的主题而不是默认主题。

## 隐藏代码行

mdBook 中有一项功能可以让您通过在代码行前加上 `#` 来隐藏代码行。

```bash
# fn main() {
    let x = 5;
    let y = 6;

    println!("{}", x + y);
# }
```

将渲染为

```rust
# fn main() {
    let x = 5;
    let y = 7;

    println!("{}", x + y);
# }
```

**目前，这只适用于使用 `rust` 注释的代码示例。 因为它会与某些编程语言的语义发生冲突。 将来，我们希望通过 `book.toml` 使其可配置，以便每个人都可以从中受益。**

## 改进默认主题

如果您认为默认主题对于特定语言来说看起来不太合适，或者可以改进，请随时[提交一个新issue](https://github.com/rust-lang/mdBook/issues)来解释您的想法，我会查看它。

您还可以创建pull-request来建议需要改进的地方。

总体而言，主题应该是轻松而清醒的，没有太多华丽的色彩。
