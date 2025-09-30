---
title: NeoVim 和 LaTeX 配置
category: [技术笔记]
tag: [NeoVim, LaTeX]
---

我此前一直使用 VSCode 编辑 LaTeX。VSCode 有一点让我不满：它的联想和自动补全太不可控了，并不能精确地让我用**最少的按键操作实现最常用的输入**。我需要更可订制的编辑器。先尝试了 Vim，可是 VimTeX 的反向同步在 Vim 上似乎有些问题，最终我发现 NeoVim 是一个很好的选项。

## 基础配置
### 安装和配置 neovim

下载 NeoVim:
```shell
brew install neovim
```

在 `~/.config/nvim` 中写入[配置文件](https://github.com/Florestan-Eusebius/nvim)。

### 安装 TeX 发行版

下载 [MacTeX](https://mirrors.tuna.tsinghua.edu.cn/CTAN/systems/mac/mactex/)。

### 安装和配置 VimTeX

见配置文件。

### 编译 LaTeX 到 PDF

VimTeX 默认使用 latexmk 编译 TeX 文稿。

可通过如下命令安装：
```bash
sudo tlmgr install latexmk
```

Latexmk 可以由 `.latexmkrc` 文件设置其行为。例如如果要用 XeLaTeX 编译，则可以在 `.latexmkrc` 中写入
```make
$pdf_mode = 5;
$dvi_mode = $postscript_mode = 0;
```
其中 `$pdf_mode = 5` 表示通过 XeLaTeX 来生成 PDF 文件，参见[文档](https://mirror-hk.koddos.net/CTAN/support/latexmk/latexmk.pdf)。更多 `.latexmkrc` 文件样例可参考[样例库](https://ctan.org/tex-archive/support/latexmk/example_rcfiles)。

## 效率提升
### 使用 LuaSnip 实现片段补全

在 LaTeX 中敲公式时，往往会大量反复使用一些基础命令，例如 `\frac{}{}`、`\mathrm{}`、`\left(\right)` 等等。如果能用更简单的键盘操作实现这些命令，并将光标放置在合适的位置——例如花括号 `{}` 中，并通过 <kbd>Tab</kbd> 跳转出花括号或在不同的花括号间跳转——输入速度将得到极大提高。代码片段（snippet）补全功能正适合这个任务。

我采用 [LuaSnip](https://github.com/L3MON4D3/LuaSnip) 插件来自定义代码片段，主要参考 [Elijan Mastnak 的教程](https://ejmastnak.com/tutorials/vim-latex/luasnip/)。这教程写得十分清楚详尽，这里无需重复其内容。

### 中英文切换

[Rime](https://rime.im) 输入法支持对选定的应用开启 `vim_mode`。开启后，从输入模式按 <kbd>Esc</kbd> 键退出到常规模式时，输入法会自动切换到英文。例如，对于 Mac 系统上的鼠须管输入法，在 squirrel.custom.yaml 中插入如下代码，即可在 iTerm2 开启 vim 模式。
```yaml
app_options:
  com.googlecode.iterm2:
  ascii_mode: true
  vim_mode: true
```

更多讨论参考[此帖](https://github.com/lei4519/blog/issues/85)。