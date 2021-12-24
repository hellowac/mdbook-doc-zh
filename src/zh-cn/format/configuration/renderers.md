# 渲染配置

## HTML 渲染选项

HTML 渲染器也有几个选项。 渲染器的所有选项都需要在 TOML 表 `[output.html]` 下指定。

以下配置选项可用：

[prefers-color-scheme]: https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-color-scheme "color scheme"

- **theme:** mdBook 带有一个默认主题和它所需的所有资源文件。 但是如果设置了这个选项，mdBook 将有选择性地用在指定文件夹中找到主题文件并覆盖默认主题文件。
- **default-theme:** 在“更改主题”下拉菜单中默认选择的主题配色方案。 默认为`light`。
- **preferred-dark-theme:** 默认的深色主题。 如果浏览器通过['prefers-color-scheme'][prefers-color-scheme]CSS 媒体查询请求网站的深色版本，则将使用此主题。 默认为`navy`。
- **curly-quotes:** 将直引号转换为卷引号，但出现在代码块和代码跨度中的引号除外。 默认为`false`。
- **mathjax-support:** 添加对 [MathJax](../mathjax.md) 的支持。 默认为`false`。
- **copy-fonts:** 将 fonts.css 和相应的字体文件复制到输出目录并在默认主题中使用它们。 默认为`true`。
- **google-analytics:** 此字段已被弃用，并将在未来版本中删除。 使用`theme/head.hbs` 文件添加适当的谷歌分析代码。
- **additional-css:** 如果您需要在不覆盖整个样式的情况下稍微更改书籍的外观，您可以指定一组样式表，这些样式表将在默认样式表之后加载，您可以在其中更改样式。
- **additional-js:** 如果您需要在不删除当前行为的情况下向书中添加一些行为，您可以指定一组 JavaScript 文件，这些文件将与默认文件一起加载。
- **print:** 配置打印选项的子表。 默认情况下，mdBook 添加了对将书籍打印为单页的支持。 这可以使用书右上角的打印图标访问。
- **no-section-label:** mdBook 默认在目录列中添加节标签。 例如，“1.”、“2.1”。 将此选项设置为 `true` 以禁用这些标签。 默认为`false`。
- **fold:** 用于配置侧边栏部分折叠行为的子表。
- **playground:** 用于配置各种运行环境设置的子表。
- **search:** 用于配置浏览器内搜索功能的子表。 mdBook 必须在启用`search`(搜索)功能的情况下编译（默认情况下启用）。
- **git-repository-url:**  该书的 git 仓库的 url。 如果提供了图标链接，将在书的菜单栏中输出。
- **git-repository-icon:** 用于 git 仓库链接的 FontAwesome 图标类。 默认为`fa-github`。
- **edit-url-template:** 编辑 url 模板，当提供时显示“建议编辑”按钮，用于直接跳转到编辑当前查看的页面。 例如 GitHub 项目将此设置为 `https://github.com/<owner>/<repo>/edit/master/{path}` 或对于 Bitbucket 项目将其设置为 `https://bitbucket.org/<owner>/ <repo>/src/master/{path}?mode=edit` 其中 {path} 将替换为仓库中文件的完整路径。
- **redirect:** 用于生成重定向到已迁移页面的子表。 该表包含键值对，其中键是需要创建重定向的文件位置，以相对于构建目录的绝对路径表示，（例如 `/appendices/bibliography.html`）。 该值可以是浏览器应导航到的任何有效 URI（例如 `https://rust-lang.org/`、`/overview.html` 或 `../bibliography.html`）。
- **input-404:** 用于丢失文件的 Markdown 文件的名称。 相应的输出文件将相同，扩展名替换为 `html`。 默认为 `404.md`。
- **site-url:** 将托管图书的网址。 这是确保 404 文件中的导航链接和 脚本/css 导入正常工作所必需的，即使在访问子目录中的 url 时也是如此。 默认为 `/`。 The url where the book will be hosted. This is required to ensure
- **cname:** 将托管您的图书的 DNS 子域或顶级域。 根据 GitHub Pages 的要求，此字符串将写入站点根目录中名为 CNAME 的文件（请参阅[管理 GitHub Pages 站点的自定义域][custom domain]）。

[custom domain]: https://docs.github.com/en/github/working-with-github-pages/managing-a-custom-domain-for-your-github-pages-site "adf"

`[output.html.print]` 表的可用配置选项：

- **enable:** 启用打印支持。 当为`false`时，将不会呈现所有打印支持。 默认为`true`。

`[output.html.fold]` 表的可用配置选项：

- **enable:** 启用部分折叠。 关闭时，所有折叠都打开。 默认为`false`。
- **level:** 越高折叠区域越开放。 当 level 为 0 时，所有折叠都关闭。 默认为`0`。

`[output.html.playground]` 表的可用配置选项：

- **editable:** 允许编辑源代码。 默认为 `false`.
- **copyable:** 在代码片段上显示复制按钮。 默认为 `true`.
- **copy-js:** 将编辑器的 JavaScript 文件复制到输出目录。默认为 `true`.
- **line-numbers** 在可编辑的代码部分显示行号。 `editable` 和 `copy-js` 都必须设置为 `true`. 默认为 `false`.

[Ace]: https://ace.c9.io/

`[output.html.search]` 表的可用配置选项：

- **enable:** 启用搜索功能。 默认为 `true`.
- **limit-results:** 搜索结果的最大数量。 默认为 `30`.
- **teaser-word-count:** 用于搜索结果预览的字数。
  默认为 `30`.
- **use-boolean-and:** 定义多个搜索词之间的逻辑链接。 如果为 true，则所有搜索词都必须出现在每个结果中。 默认为 `false`.
- **boost-title:** 如果搜索词出现在标题中，则搜索结果分数的提升因子。 默认为 `2`.
- **boost-hierarchy:** 如果搜索词出现在层次结构中，则搜索结果分数的提升因子。 层次结构包含父文档的所有标题和所有父标题。 默认为 `1`.
- **boost-paragraph:** 如果搜索词出现在文本中，则搜索结果分数的提升因子。 默认为 `1`.
- **expand:** 如果搜索应该匹配更长的结果，则为真，例如 搜索 `micro` 应该匹配 `microwave`。默认为 `true`.
- **heading-split-level:** 搜索结果将链接到包含结果的文档部分。 文档按此级别或更低级别的标题分成多个部分。 默认为 `3`. (`### This is a level 3 heading`)
- **copy-js:** 将搜索实现的 JavaScript 文件复制到输出目录。 默认为 `true`.

这显示了 **book.toml** 中所有可用的 HTML 输出选项：

```toml
[book]
title = "Example book"
authors = ["John Doe", "Jane Doe"]
description = "The example book covers examples."

[output.html]
theme = "my-theme"
default-theme = "light"
preferred-dark-theme = "navy"
curly-quotes = true
mathjax-support = false
copy-fonts = true
additional-css = ["custom.css", "custom2.css"]
additional-js = ["custom.js"]
no-section-label = false
git-repository-url = "https://github.com/rust-lang/mdBook"
git-repository-icon = "fa-github"
edit-url-template = "https://github.com/rust-lang/mdBook/edit/master/guide/{path}"
site-url = "/example-book/"
cname = "myproject.rs"
input-404 = "not-found.md"

[output.html.print]
enable = true

[output.html.fold]
enable = false
level = 0

[output.html.playground]
editable = false
copy-js = true
line-numbers = false

[output.html.search]
enable = true
limit-results = 30
teaser-word-count = 30
use-boolean-and = true
boost-title = 2
boost-hierarchy = 1
boost-paragraph = 1
expand = true
heading-split-level = 3
copy-js = true

[output.html.redirect]
"/appendices/bibliography.html" = "https://rustc-dev-guide.rust-lang.org/appendix/bibliography.html"
"/other-installation-methods.html" = "../infra/other-installation-methods.html"
```

## Markdown 渲染

Markdown渲染器 将运行预处理器，然后持续输出Markdown解析结果 。 这对于调试预处理器非常有用，尤其是与 `mdbook test` 结合使用以查看 `mdbook` 传递给 `rustdoc` 的 Markdown文本。

Markdown渲染器 包含在 `mdbook` 中，但默认情况下禁用。 通过向 `book.toml` 添加一个空表来启用它，如下所示：

```toml
[output.markdown]
```

Markdown渲染器 目前没有配置选项； 仅是启用还是禁用。

请参阅 [预处理](preprocessors.md) 了解如何指定哪些预处理器应在 Markdown 渲染器之前运行。

## 自定义渲染

可以通过将 [output.foo] 表添加到 `book.toml` 来启用自定义渲染器。
与[预处理](preprocessors.md)类似，这将指示 `mdbook` 将书的表示传递给 `mdbook-foo` 以进行渲染。 有关更多详细信息，请参阅[替换后端][alternative backends]一章。

自定义渲染器可以访问其表中的所有字段（即`[output.foo]` 下的任何内容）。 mdBook 检查两个常见字段：

- **command:** 自定义渲染器执行的命令。 默认为带有 `mdbook-` 前缀的渲染器名称（例如 `mdbook-foo`）。
- **optional:** 如果为`true`，则如果未安装该命令将被忽略，否则 mdBook 将失败并显示错误。 默认为`false`。

[alternative backends]: ../../for_developers/backends.md
