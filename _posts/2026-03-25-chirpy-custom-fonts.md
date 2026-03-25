---
title: "在 Chirpy 中自定义正文字体"
date: 2026-03-25 17:50:00 +0800
categories: [建站日志]
tags: [jekyll, chirpy, 字体, css, google-fonts]
author: [copilot, itpyi]
description: 在不改动 Chirpy 主题源码的前提下，通过覆盖机制实现字体定制，并保持主题升级时的配置稳定性。
---

Chirpy 主题默认使用 Source Sans Pro 作为正文字体。如果需要替换为其他字体（例如衬线字体或特定的中文字体组合），直接修改 Gem 内部文件会在主题升级时丢失。为保持配置稳定性，采用项目根目录覆盖的方式实现。

## 实现原理

Chirpy 是基于 Ruby Gem 分发的主题，其源文件位于：

```
/opt/homebrew/lib/ruby/gems/3.3.0/gems/jekyll-theme-chirpy-7.5.0/
```

Jekyll 在构建时的加载顺序为：先加载 Gem 主题文件，再加载项目根目录中的同名文件，后者会覆盖前者。利用这一机制，可在不修改主题源码的前提下定制字体。

字体定制涉及两个环节：
1. 从 CDN 加载字体资源（如 Google Fonts）。
2. 在 CSS 中应用到目标元素（如 `body` 标签）。

对应的覆盖点分别是 `_data/origin/cors.yml` 和 `assets/css/jekyll-theme-chirpy.scss`。

## 关键文件

### `_data/origin/cors.yml`

此文件控制主题在 HTML `<head>` 中加载的 webfonts URL。在项目根目录创建同名文件，内容如下：

```yaml
# 指定 Google Fonts API URL
webfonts: https://fonts.googleapis.com/css2?family=[字体名称]:wght@400;700&family=[中文字体名称]&display=swap

# 预连接优化
resources:
  - https://fonts.googleapis.com
  - https://fonts.gstatic.com
```

将 `[字体名称]` 和 `[中文字体名称]` 替换为实际字体名（URL 编码格式，空格用 `+` 替代）。`wght@400;700` 指定加载常规和粗体字重。

### `assets/css/jekyll-theme-chirpy.scss`

主题的 CSS 入口文件。在项目根目录创建同名文件，添加字体覆盖规则：

```scss
---
---

/* prettier-ignore */
@use 'main
{%- if jekyll.environment == 'production' -%}
  .bundle
{%- endif -%}
';

/* 覆盖正文字体 */
body {
  font-family: '[字体名称]', '[中文字体名称]', '[系统中文字体]', serif !important;
}
```

关键点：
- 保留原有的 `@use 'main'` 语句，确保主题样式正常加载。
- 在其后追加 `body` 规则，使用 `!important` 覆盖主题默认的 `font-family`（主题变量使用了 `!default`，无法直接覆盖）。
- 字体回退链：优先使用自定义字体，中文字符回退到中文字体，最后回退到系统衬线字体。

## 维护说明

该方案不改动 Gem 内部文件，主题升级时冲突面较小。常规维护建议如下：

- 配置文件位于项目根目录，不受 `bundle update` 影响。
- 主题升级后，优先检查 `_data/origin/cors.yml` 和 `assets/css/jekyll-theme-chirpy.scss` 的结构是否变化。
- 若主题后续在 `_config.yml` 中增加原生字体配置选项，可考虑迁移到官方方案。
- 重新构建站点（`bundle exec jekyll build`）即可生效，无需额外配置。
- 使用浏览器开发者工具检查元素的 Computed 样式，确认字体加载正确。

若未来主题原生支持更灵活的字体配置，可移除这套覆盖文件并迁移到官方实现。
