# SUMMARY.md

mdBook 使用摘要文件来了解要包含哪些章节、它们应该以什么顺序出现、它们的层次结构是什么以及源文件在哪里。 没有这个文件，就没有书。

这个markdown文件必须命名为`SUMMARY.md`。 它的格式非常严格，必须遵循下面概述的结构，以便于解析。 下面未指定的任何元素，无论是格式还是文本，最多可能会被忽略，或者在尝试构建书籍时可能会导致错误。

## 结构

1. ***标题*** - 虽然是可选的，但通常以标题开头，通常是<code
   class="language-markdown"># Summary</code>。 然而，这被解析器忽略，并且可以省略。

   ```markdown
   # Summary
   ```

2. ***前缀章节*** - 在主要编号章节之前，可以添加不编号的前缀章节。 这对于前言、介绍等很有用。但是，有一些限制。 前缀章节不能嵌套； 它们都应该在根级别。 一旦添加了编号的章节，就无法添加前缀章节。

   ```markdown
   [A Prefix Chapter](relative/path/to/markdown.md)

   - [First Chapter](relative/path/to/markdown2.md)
   ```

3. ***部分标题*** - 标题可用作以下编号章节的标题。 这可用于在逻辑上分隔本书的不同部分。 标题呈现为不可点击的文本。 标题是可选的，编号的章节可以根据需要分成任意多个部分。

   ```markdown
   # My Part Title

   - [First Chapter](relative/path/to/markdown.md)
   ```

4. ***编号章节*** - 编号的章节概述了本书的主要内容，并且可以嵌套，从而形成一个很好的层次结构（章节、子章节等）。

   ```markdown
   # Title of Part

   - [First Chapter](relative/path/to/markdown.md)
   - [Second Chapter](relative/path/to/markdown2.md)
      - [Sub Chapter](relative/path/to/markdown3.md)

   # Title of Another Part

   - [Another Chapter](relative/path/to/markdown4.md)
   ```

   编号的章节可以用`-` 或`*` 表示（不要混合分隔符）。

5. ***后缀章节*** - 与前缀章节一样，后缀章节是无编号的，但它们位于已编号的章节之后。

   ```markdown
   - [Last Chapter](relative/path/to/markdown.md)

   [Title of Suffix Chapter](relative/path/to/markdown2.md)
   ```

6. ***草稿章节*** - 草稿章节是没有文件和内容的章节。 一章草稿的目的是表明未来仍有待编写的章节。 或者当您仍在对书的结构进行布局以避免创建文件时，您仍在大量更改书的结构。 草稿章节将在 HTML 呈现器中呈现为目录中的禁用链接，您可以在左侧目录中的下一章节中看到。 草稿章节像普通章节一样编写，但不写入文件路径。

   ```markdown
   - [Draft Chapter]()
   ```

7. ***分割线*** - 可以在任何其他元素之前、之间和之后添加分隔符。 它们会在构建的目录中生成 HTML 渲染行。 分隔符是仅包含破折号和至少三个破折号的行： : `---`.

   ```markdown
   # My Part Title
   
   [A Prefix Chapter](relative/path/to/markdown.md)

   ---

   - [First Chapter](relative/path/to/markdown2.md)
   ```
  
### 样例

下面是本指南的`SUMMARY.md`的markdown文件源，结果目录呈现在左侧。

```markdown
{{#include ../../SUMMARY.md}}
```
