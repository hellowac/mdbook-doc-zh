# MathJax 支持

mdBook 通过 [MathJax](https://www.mathjax.org/) 选择性地支持数学方程式。

要启用 MathJax，您需要在 `output.html` 部分下的 `book.toml` 中添加 `mathjax-support` 键。

```toml
[output.html]
mathjax-support = true
```

> **注意:** 尚不支持 MathJax 使用的常用分隔符。 您目前不能使用 `$$ ... $$` 作为分隔符，并且 `\[ ... \]` 分隔符需要额外的反斜杠才能工作。 希望这个限制很快就会解除。
>
> **注意:** 当您在 MathJax 块中使用双反斜杠（例如 `\begin{cases} \frac 1 2 \\ \frac 3 4 \end{cases}` 等命令中）时，您需要添加*两个额外的反斜杠*（例如，`\begin{cases} \frac 1 2 \\\\ \frac 3 4 \end{cases}`)。

## 内联方程式

内联方程由 `\\(` 和 `\\)` 分隔。 例如，要渲染以下内联方程 \\( \int x dx = \frac{x^2}{2} + C \\)，您可以编写以下内容：

```text
\\( \int x dx = \frac{x^2}{2} + C \\)
```

### 块方程式

块方程由 `\\[` 和 `\\]` 分隔。 渲染以下等式

\\[ \mu = \frac{1}{N} \sum_{i=0} x_i \\]

你会写：

```bash
\\[ \mu = \frac{1}{N} \sum_{i=0} x_i \\]
```
