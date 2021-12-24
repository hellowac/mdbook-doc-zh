# index.hbs

`index.hbs` 是用于渲染book的[handlebars]模板。 Markdown 文件被处理为 html，然后注入到该模板中。

[handlebars]: http://handlebarsjs.com/

如果您想更改书籍的布局或样式，您可能需要稍微修改此模板。 本节是您需要了解的内容。

## Data

许多数据通过"context"(上下文)暴露给[handlebars]模板。 在[handlebars]模板中，您可以使用这些数据。

```handlebars
{{name_of_property}}
```

以下是公开的属性列表：

- ***language*** 书的语言采用 `en` 形式，如 `book.toml` 中指定（如果未指定，默认为 `en`）。 例如在 <code class="language-html">\<html lang="{{ language }}"></code> 中使用。
- ***title*** 用于当前页面的标题。 这与 `{{chapter_title }} - {{ book_title }}` 相同，除非没有设置 `book_title`，在这种情况下它只是默认为 `chapter_title`。
- ***book_title*** 书名，在`book.toml`中指定
- ***chapter_title*** 当前章节的标题，如`SUMMARY.md`中所列

- ***path*** 从源目录到原始 Markdown 文件的相对路径
- ***content*** 这是渲染过的Markdown。
- ***path_to_root*** 这是一个只包含 `../` 的路径，它指向根目录。 由于保留了原始目录结构，因此使用此 `path_to_root` 预先添加相关链接很有用。

- ***chapters*** 是一个形式为字典的数组

  ```json
  {"section": "1.2.1", "name": "name of this chapter", "path": "dir/markdown.md"}
  ```

  包含本书的所有章节。 例如，它用于构建目录（侧边栏）。

## Handlebars Helpers

除了您可以访问的属性外，还有一些[handlebars] helpers可供您使用。

### 1. toc

toc helper 是这样使用的

```handlebars
{{#toc}}{{/toc}}
```

并输出看起来像这样的东西，这取决于你的书的结构

```html
<ul class="chapter">
    <li><a href="link/to/file.html">Some chapter</a></li>
    <li>
        <ul class="section">
            <li><a href="link/to/other_file.html">Some other Chapter</a></li>
        </ul>
    </li>
</ul>
```

如果你想用另一种结构制作目录，你可以访问包含所有数据的章节属性。 目前唯一的限制是您必须使用 JavaScript 而不是使用[handlebars] helpers来完成。

```html
<script>
var chapters = {{chapters}};
// Processing here
</script>
```

### 2. previous / next

`previous` 和 `next` 的helpers公开了指向前一章和下一章的`link`(链接)和`name`(名称)属性。

它们是这样使用的

```handlebars
{{#previous}}
    <a href="{{link}}" class="nav-chapters previous">
        <i class="fa fa-angle-left"></i>
    </a>
{{/previous}}
```

仅在上一章/下一章存在时才会渲染 inner html。 当然，可以根据自己的喜好更改 inner html。

------

*如果您想公开其他属性或helpers，请[创建一个新issue](https://github.com/rust-lang/mdBook/issues)*
