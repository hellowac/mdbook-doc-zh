# 命令行工具

mdBook 既可以用作命令行工具，也可以用作 [Rust crate](https://crates.io/crates/mdbook)。 让我们首先关注命令行工具功能。

## 二进制文件安装

我们尽最大努力为主要平台提供了预编译的二进制文件。 访问[发布页面](https://github.com/rust-lang/mdBook/releases)以下载适合您平台的版本。

## 源码安装

mdBook 也可以通过在本地机器上编译源代码来安装。

### 先决条件

mdBook 是用 **[Rust](https://www.rust-lang.org/)** 编写的，因此需要用 **Cargo** 编译。 如果您还没有安装 Rust，请立即[安装](https://www.rust-lang.org/tools/install)。

### 安装 Crates.io 版本

如果您已经安装了 Rust 和 Cargo，则安装 mdBook 相对容易。 你只需要在你的终端中输入这段代码：

```bash
cargo install mdbook
```

这将从 [Crates.io](https://crates.io/) 获取最新版本的源代码并编译它。 您必须将 Cargo 的 `bin` 目录添加到您的 `PATH` 环境变量中。

在终端中运行 `mdbook help` 以验证它是否有效。 恭喜，您已经安装了 mdBook！

### 安装 Git 版本

**[git 版本](https://github.com/rust-lang/mdBook)**包含所有最新的错误修复和功能，如果您不能等到下一个版本，它们将在 **Crates.io** 的下一个版本中发布。 您可以自己构建 git 版本。 打开您的终端并导航到您选择的目录。 我们需要克隆 git 存储库，然后使用 Cargo 构建它。

```bash
git clone --depth=1 https://github.com/rust-lang/mdBook.git
cd mdBook
cargo build --release
```

可执行文件 `mdbook` 将在 `./target/release` 文件夹中，这应该添加到`PATH` 环境变量中。
