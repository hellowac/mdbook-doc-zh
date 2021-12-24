# 配置预处理器

默认情况下，以下预处理器可用并包含在内：

- `links`: 处理器助手将展开一章中的 `{{ #playground }}`、`{{ #include }}` 和 `{{ #rustdoc_include }}` 并包含文件里面的内容。
- `index`: 将所有名为 `README.md` 的章节文件转换为 `index.md`。 也就是说，所有的 `README.md` 都会被渲染到渲染的书中的一个索引文件 `index.html` 中。

**book.toml** &nbsp;

```toml
[build]
build-dir = "build"
create-missing = false

[preprocessor.links]

[preprocessor.index]
```

## 自定义预处理器配置

像渲染器一样，预处理器需要有自己的表（例如`[preprocessor.mathjax]`）。 在该部分中，您可以通过向表中添加键值对来将额外的配置传递给预处理器。

例如：

```toml
[preprocessor.links]
# set the renderers this preprocessor will run for
renderers = ["html"]
some_extra_feature = true
```

### 将预处理器依赖项锁定到渲染器

您可以通过将两者绑定在一起来明确指定应该为渲染器运行预处理器。

```toml
[preprocessor.mathjax]
renderers = ["html"]  # mathjax only makes sense with the HTML renderer
```

### 提供您自己的命令

默认情况下，当您将 `[preprocessor.foo]` 表添加到 `book.toml` 文件时，`mdbook` 将尝试调用 `mdbook-foo` 可执行文件。
如果要使用不同的程序名称或传递命令行参数，可以通过添加`command`字段来覆盖此行为。

```toml
[preprocessor.random]
command = "python random.py"
```

### 需要一定的排序

预处理器的运行顺序可以通过 `before` 和 `after` 字段来控制。

例如，假设你希望你的 `linenos` 预处理器 处理可能已经被 `{{#include}}` 包含过的行； 然后你希望它在内置的 `links` 预处理器之后运行，你可以要求使用 `before` 或 `after` 字段：

```toml
[preprocessor.linenos]
after = [ "links" ]
```

或

```toml
[preprocessor.links]
before = [ "linenos" ]
```

也可以在同一个配置文件中指定以上两个，虽然是多余的。

通过 `before` 和 `after` 指定的具有相同优先级的预处理器按名称排序。 任何无限循环都将被检测到并产生错误。
