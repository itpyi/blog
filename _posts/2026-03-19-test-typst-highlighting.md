---
title: "Testing Typst Syntax Highlighting"
date: 2026-03-19 15:35:00 +0800
categories: [测试]
tags: [typst, 测试]
description: 测试 Typst 代码块的语法高亮功能
---

## Typst 语法高亮测试

这篇文章用来测试博客对 Typst 代码的语法高亮支持。

### 简单的 Typst 代码示例

```typst
#set page(width: 10cm, height: auto)
#set text(font: "Linux Libertine", size: 11pt)

= Introduction
#lorem(20)

== Hello World
Here is some *bold* and _italic_ text.

We can also write math: $ sum_(i=1)^n i = (n(n+1))/2 $
```

### 函数定义示例

```typst
#let greet(name) = {
  [Hello, #name!]
}

#greet("World")
#greet("Typst")
```

### 数学公式示例

```typst
$ "area" = pi r^2 $

$ sum_(k=1)^n k = (n(n+1))/2 $

$ mat(
  1, 2, ..., 10;
  2, 2, ..., 10;
  dots.v, dots.v, dots.down, dots.v;
  10, 10, ..., 10;
) $
```

### 表格示例

```typst
#table(
  columns: (auto, auto, auto),
  [*Name*], [*Age*], [*City*],
  [Alice], [25], [New York],
  [Bob], [30], [London],
  [Charlie], [35], [Tokyo],
)
```

### Python 代码（验证 Rouge 仍然工作）

下面是一段 Python 代码，用于验证 Rouge 语法高亮仍然正常工作：

```python
def fibonacci(n):
    """Calculate Fibonacci numbers"""
    if n <= 1:
        return n
    return fibonacci(n-1) + fibonacci(n-2)

# Test the function
for i in range(10):
    print(f"F({i}) = {fibonacci(i)}")
```

## 总结

如果你能看到 Typst 代码块和 Python 代码块都有漂亮的语法高亮，那么测试成功！
