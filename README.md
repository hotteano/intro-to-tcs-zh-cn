# 《理论计算机科学导论》中文翻译

本仓库是Boaz Barak所著《Introduction to Theoretical Computer Science》的中文翻译。本书原文见[boazbk/tcs](https://github.com/boazbk/tcs)。

若读者对翻译文本有疑问/意见等，欢迎提交issues/pull requests。关于文章内容本身的疑问，请移步原文仓库。

## How To Build

本项目使用 MDBook 构建。你需要先安装 Rust 环境（推荐使用 `rustup`），并确保你可以使用 `cargo`。然后运行：

```bash
cargo install mdbook mdbook-admonish mdbook-mathpunc mdbook-mermaid mdbook-numeq mdbook-numthm mdbook-toc mdbook-footnote mdbook-katex
mdbook serve # will launch a website
mdbook build # will generate html files
```

参加本书的翻译工作的译者有（按拼音序）：
