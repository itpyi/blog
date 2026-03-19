---
title: "在 Chirpy 中为 Typst 增加代码高亮"
date: 2026-03-19 15:35:00 +0800
categories: [建站日志]
tags: [jekyll, chirpy, Typst, prism.js]
author: [copilot, itpyi]
description: 在不改动 Chirpy 主题源码的前提下，为 Typst 代码块补齐语法高亮，并保持与原生代码块一致的展示和交互。
---

Jekyll 默认高亮引擎 Rouge 目前不支持 Typst。为了让 Typst 代码块具备可读的语法高亮，同时保持 Chirpy 升级友好，采用了前端补充高亮的方案。

## 实现原理

核心逻辑放在 `_includes/metadata-hook.html`。这是 Chirpy 预留的扩展入口，可在不改主题源码的情况下向页面注入脚本与样式。

主要步骤如下：
1. 引入 Prism 核心与 Typst 语法定义（本地文件）。
2. 检测页面中的 `language-typst` 代码块，交给 Prism 做词法标注。
3. 将渲染结果包装成 Chirpy 的代码块结构（含标题、行号、复制按钮），复用现有交互与样式体系。
4. 用少量 CSS 覆盖颜色，分别适配浅色和深色模式。

## 关键文件

- `_includes/metadata-hook.html`：注入脚本、执行高亮、处理 DOM 结构与主题配色。
- `assets/js/lib/prism.min.js`：Prism 核心（本地化，避免外链不稳定）。
- `assets/js/lib/prism-typst.js`：Typst 语法定义。

## 线上稳定性修正

本地预览正常、线上偶发不生效时，问题通常不在构建，而在浏览器端执行时序：

1. Prism 或目标代码块在初次执行时尚未就绪。
2. 单次初始化策略错过时机后不会再补跑。

对应处理：

- 初始化改为“可重试”：在短时间窗口内重复检查依赖是否就绪。
- 补充多个触发点：`DOMContentLoaded`、`load`、`pageshow`。
- 对 `prism-typst.js` 做作用域隔离，避免全局变量冲突导致脚本中断。
- 移除 `prism.min.js` 的 `sourceMappingURL` 注释，消除无关的 Source Map 警告噪音。

## 使用方式

```typst
#set page(width: 10cm, height: auto)
#set text(font: "Linux Libertine", size: 11pt)

= Introduction
#lorem(20)

#let greet(name) = [Hello, #name!]
#greet("Typst")
```

以上写法无需额外 front matter 配置，发布后会自动进入 Typst 高亮流程。

## 维护说明

该方案不改动 Chirpy Gem 内部模板文件，后续升级主题时冲突面较小。常规维护建议如下：

- 优先使用本地静态资源，不依赖第三方 CDN 可用性。
- 主题升级后，优先检查 `metadata-hook` 注入点是否仍在。
- 若 Rouge 后续原生支持 Typst，可移除这套前端补丁并回归服务端高亮。
