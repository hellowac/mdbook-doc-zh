# 主题

默认渲染器使用一个[handlebars]模板来渲染你的markdown文件，并带有一个包含在 mdBook 二进制文件中的默认主题。

[handlebars]: http://handlebarsjs.com/

主题是完全可定制的，您可以通过在项目根目录的 `src` 文件夹旁边添加一个`theme`目录，有选择地替换主题中的每个文件。 使用要覆盖的文件的名称创建一个新文件，现在将使用该文件而不是默认文件。

以下是您可以覆盖的文件：

- **_index.hbs_** [handlebars] 模板.
- **_head.hbs_** 附加到 HTML `<head>` 部分。
- **_header.hbs_** 内容附加在每个书页的顶部。
- **_css/_** 包含用于设计样式的 CSS 文件。
  - **_css/chrome.css_** 用于 UI 元素。
  - **_css/general.css_** 是基本样式。
  - **_css/print.css_** 是打印机输出的样式。
  - **_css/variables.css_** 包含在其他 CSS 文件中使用的变量。
- **_book.js_** 主要用于添加客户端功能，例如隐藏/取消隐藏侧边栏，更改主题，...
- **_highlight.js_** 是用于突出显示代码片段的 JavaScript，您不需要修改它。
- **_highlight.css_** 是用于代码突出显示的主题。
- **_favicon.svg_** 和 **_favicon.png_** 使用的图标。 SVG 版本由[较新的浏览器][newer browsers]使用。

通常，当您想要调整主题时，您不需要覆盖所有文件。
如果您只需要更改样式表，则覆盖所有其他文件没有意义。
因为自定义文件优先于内置文件，所以它们不会随着新的修复程序/功能而更新。

**注意:** 覆盖文件时，可能会破坏某些功能。 因此，我建议使用默认主题中的文件作为模板，并且只添加/修改您需要的内容。 您可以使用 `mdbook init --theme` 自动将默认主题复制到源目录中，然后删除不想覆盖的文件。

如果完全替换所有内置主题，请务必在配置中设置 [`output.html.preferred-dark-theme`]，默认为内置`navy`主题。

[`output.html.preferred-dark-theme`]: ../configuration/renderers.md#html-renderer-options
[newer browsers]: https://caniuse.com/#feat=link-icon-svg
