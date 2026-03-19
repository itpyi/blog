---
title: "Chirpy 主题添加 Typst 代码高亮支持"
date: 2026-03-19 15:35:00 +0800
categories: [建站日志]
tags: [jekyll, chirpy, Typst, prism.js]
author: [copilot, itpyi]
description: 采用 Prism.js 前端注入方案，在不修改 Chirpy 主题源码的前提下，为 Typst 代码块添加语法高亮及原生风格的 UI 支持。
---

Jekyll 默认的语法高亮引擎 Rouge 尚未内置 Typst 支持。为在博客中正确展示 Typst 代码，本文采用前端注入 Prism.js 的方案。该方案在保持 Chirpy 主题源码清洁（Upstream Compatible）的同时，实现了与原生代码块一致的 UI 体验（行号、复制按钮、深色模式适配）。

## 实现原理

方案核心逻辑位于 `_includes/metadata-hook.html`。该文件由 Chirpy 主题预留，用于注入自定义 `<head>` 内容。

主要步骤如下：
1.  **引入 Prism.js**：加载 Prism 核心库（手动模式）及 Typst 语言定义文件。
2.  **DOM 重构**：脚本自动检索 `language-typst` 代码块，将其重构为 Chirpy 标准的 `table.rouge-table` 结构。这使得 Prism 渲染的代码块能直接复用主题现有的 CSS 样式和 JavaScript 交互（如剪贴板复制）。
3.  **样式适配**：通过 CSS 覆盖 Prism 默认样式，确保在明亮和深色模式下颜色表现正常。

## 关键文件

### 语言定义文件
由于 Prism 官方 CDN 的 Typst 包路径不稳定，需将语言定义文件下载至本地仓库：
- 路径：`assets/js/lib/prism-typst.js`

### 注入脚本 (`_includes/metadata-hook.html`)

将以下代码写入 `_includes/metadata-hook.html`。该脚本负责加载资源并执行 DOM 转换。

```html
<!-- Custom: Add Prism.js for Typst syntax highlighting -->
{% if page.layout == 'post' %}
  <!-- Load full Prism.js with manual mode -->
  <script src="https://cdn.jsdelivr.net/npm/prismjs@1.29.0/prism.min.js" data-manual></script>
  
  <!-- Load local Typst language definition -->
  <script src="{{ '/assets/js/lib/prism-typst.js' | relative_url }}"></script>
  
  <script>
    (function() {
      function highlightTypst() {
        // 查找所有 Typst 代码块
        document.querySelectorAll('pre > code.language-typst').forEach(function(element) {
          var pre = element.parentElement;
          // 避免重复处理
          if (!pre || pre.closest('.language-typst.highlighter-rouge')) return;

          var rawText = element.textContent;
          // 调用 Prism 进行高亮
          var highlighted = (window.Prism && Prism.languages.typst) 
            ? Prism.highlight(rawText, Prism.languages.typst, 'typst')
            : rawText;

          // 生成行号
          var lines = rawText.replace(/\n$/, '').split('\n');
          var lineNumbers = '';
          for (var i = 1; i <= lines.length; i++) lineNumbers += i + '\n';

          // 构建符合 Chirpy 规范的 DOM 结构
          var wrapper = document.createElement('div');
          wrapper.className = 'language-typst highlighter-rouge';
          wrapper.innerHTML =
            '<div class="code-header">' +
              '<span data-label-text="Typst"><i class="fas fa-code fa-fw small"></i></span>' +
              '<button aria-label="copy" data-title-succeed="已复制！"><i class="far fa-clipboard"></i></button>' +
            '</div>' +
            '<div class="highlight"><code><table class="rouge-table"><tbody><tr>' +
              '<td class="rouge-gutter gl"><pre class="lineno">' + lineNumbers + '</pre></td>' +
              '<td class="rouge-code"><pre class="prism-typst-code language-typst prism-highlighted">' + highlighted + '</pre></td>' +
            '</tr></tbody></table></code></div>';

          // 替换原始节点
          pre.replaceWith(wrapper);
        });
      }
      
      if (document.readyState === 'loading') {
        document.addEventListener('DOMContentLoaded', highlightTypst);
      } else {
        highlightTypst();
      }
    })();
  </script>

  <!-- 样式适配（代码略，见源码） -->
  <style>
    /* ... CSS rules for light/dark mode ... */
  </style>
{% endif %}
```

## 效果演示

配置完成后，使用标准 Markdown 语法即可触发高亮：

````markdown
```typst
#set page(width: 10cm, height: auto)
= Hello Typst
#let func(x) = x * 2
```
````

渲染效果如下：

```typst
#set page(width: 10cm, height: auto)
#set text(font: "Linux Libertine", size: 11pt)

= Introduction
#lorem(20)

#let greet(name) = [Hello, #name!]
#greet("Typst")
```

对比原生 Python 高亮（用于检查 UI 一致性）：

```python
def fibonacci(n):
    if n <= 1:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)
```

## 维护说明

此方案不修改 `_layouts` 或 `assets` 中的主题原生文件，因此升级 Chirpy 主题时通常不会发生冲突。若后续 Rouge 官方添加了 Typst 支持，只需删除 `_includes/metadata-hook.html` 中的相关代码即可回退至服务端渲染。
