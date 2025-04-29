---
title: Typst 中的上下文相关变量
categories: [技术笔记]
tags: [Typst]
description: 借助 typst 中的上下文相关变量，使用户可以便捷地修改包中函数的行为。
---

考虑如下问题：我们希望写一个 Typst 包，包里包含函数 `A`、`B`、`C`，其作用是给它作用的文字前加上一个数字 `x`（并各自做一些其他事情，例如改变颜色等）。我们可以在包里给出预设的数字 `x`，但我们希望给用户选择数字的自由。我们希望用户可以通过一条简单的命令 `set-pre(y)` 将数字设成 `y`。

## 实际应用

上述特性显然有广泛的应用场景。例如在一个定理包中，我们希望使用者自行决定定理编号依赖于几级标题，例如

- 依赖 0 级标题：定理 1，定理 2……
- 依赖 1 级标题：定理 1.1，定理 1.2，定理 2.1……
- 依赖 2 级标题：定理 1.1.1，定理 1.1.2，定理 1.2.1，定理 2.1.1……

我们希望用户能通过一个简短的命令来决定定理编号依赖的标题级别，而不需要重新写一遍各个定理环境的定义。详见[github.com/itpyi/typst-dingli](https://github.com/itpyi/typst-dingli)。本文将详述实现上述功能的基本原理，相信读者读完本文将能够轻松理解上述包的实现，并理解 Typst 中上下文相关变量的使用方法。

## 失败方案

下述方案均不可行。

### 全局变量

结论：完全不可行。

`pkg.typ` 中：

```
#let x = 1

#let A(body) = {[#x #body]}
#let B(body) = {[#x #body]}
#let C(body) = {[#x #body]}
```

`main.typ` 中

```
#import "pkg.typ": *

#let x = 2

#A[#lorem(20)] //输出结果为 1 ...
```

这相当于在 A 后面重新定义了 `x`，无效。

或者 `main.typ` 中

```
#import "pkg.typ": *

#if true{
	x = 2
}

#A[#lorem(20)] //输出结果仍为 1 ...
```
此时 `x` 值的更改只在 `#if true` 后的花括号中成立，无效。

或者 `pkg.typ` 中：
```
#let x = 1

#let A(body) = {[#x #body]}

#let set-per(y) = {
	x = y //error：variables from outside the function are read-only and cannot be modified
}
```

且在 `main.typ` 中：

```
#import "pkg.typ": *

#set-per(2)

#A[#lorem(20)]
```

报错：函数中无法修改全局变量。

### `with` 特性

结论：可行，但需要重新定义函数。

`pkg.typ` 中：
```
#let x = 1

#let A(pre: x, body) = {[#pre #body]}
#let B(pre: x, body) = {[#pre #body]}
#let C(pre: x, body) = {[#pre #body]}
```
`main.typ` 中
```
#import "pkg.typ": *

#let A = A.with(pre: 2)

#A[#lorem(20)] //输出结果为 2 ...

#B[#lorem(20)] //输出结果为 1 ...

#C[#lorem(20)] //输出结果为 1 ...
```
我们发现，我们需要逐个地重新定义函数，才能使得改动对所有函数都生效。

如果试图在 `main.typ` 中
```
#import "pkg.typ": *

#let func-list = (A, B, C)
#for func in func-list{
	func = func.with(pre: 2)
}

#A[#lorem(20)] //输出结果为 1 ...

#B[#lorem(20)] //输出结果为 1 ...

#C[#lorem(20)] //输出结果为 1 ...
```
则无效，因为函数的重定义必须通过 `#let` 语句实现，但 for 循环中的 `#let` 只作用于循环的花括号内。

## 解决方案

使用 `state` 和 `context` 特性。见[官方文档](https://typst.app/docs/reference/introspection/state/)。

`pkg.typ` 中
```
#let x-value = state("x", 1)

#let A(body) = {
	let pre = context x-value.get()
	[#pre #body]
}

#let B(body) = {
	let pre = context x-value.get()
	[#pre #body]
}

#let C(body) = {
	let pre = context x-value.get()
	[#pre #body]
}

#let set-pre(y) = {
	x-value.update(y)
}
```
`main.typ`  中
```
#import "pkg.typ": *

#set-pre(2)

#A[#lorem(20)] //输出结果为 2 ...

#B[#lorem(20)] //输出结果为 1 ...

#C[#lorem(20)] //输出结果为 1 ...
```

## 更复杂的情形

由 `state` 定义的、`self.update()` 更新的、`self.get()` 获得的，是一个上下文相关的对象，Typst 规定，这样的对象必须在 `context` 环境中操作。考虑下诸例。
- 一个上下文依赖的对象不等于一个整数：

  ```
  #let s = state("x", 1)

  #let x = context s.get()
  #(x == 1) //false
  ```
-  `x` 是什么取决于如何声明它，所以它仍然是一个上下文依赖的对象而非整数：

  ```
  #let x = context s.get()
  #context{x == 1} //false
  ```
- `context` 作用于后面的整个表达式：

  ```
  #(context s.get() == 1) //true
  ```
- 另一种写法：

  ```
  #context{
    let x = s.get()
    x == 1
  } //true
  ```
- 可以定义更复杂的上下文依赖对象，但仍然不能将它等同于其取值：
  
  ```
  #let b = context{s.get() == 1}
  #b //true
  #(b == true) //false
  ```
- 定义变量时，编译器直接执行此命令，因此所有上下文相关的操作都必须在 `context` 环境中实现：
  
  ```
  #let b = {s.get() == 1} //error: can only be used when context is known
  #context b 
  ```
- 定义函数时，只在调用函数时执行代码，因此定义时不必写出 `context`，只要在 `context` 中调用此函数即可（虽然这种操作看起来有些危险）：
  
  ```
  #let b() = {s.get() == 1}
  #context b() //true
  #context(b()==true) //true
  ```
- 如果在定义函数时即声明 `context`，行为将与定义变量类似（注意第三行与上例的区别）：
  
  ```
  #let b() = context{s.get() == 1}
  #b() //true
  #(b() == true) //false
  ```

因此，如果函数 `A, B, C` 并不是简单地输出 `x`，而是要先计算 `x` 得到一个结果再输出，那么这个计算必须被放在 `context` 环境中。例如，我们希望函数 `A` 将 `x + 1` 放在文本前，则在 `pkg.typ` 中

```
#let x-value = state("y", 1)

// 不能这样写
#let A(body) = {
  let pre = context x-value.get()
  [#(pre + 1) #body] //error: cannot add content and integer
}

// 也不能这样写
#let A(body) = {
  let pre = context x-value.get()
  [#context{pre + 1} #body] //error: cannot add content and integer
}

// 可以这样写（方法一）
#let A(body) = {
  [#context{x-value.get()+1} #body]
}

// 也可以这样写（方法二）
#let pre() = {
  x-value.get() + 1
}

#let A(body) = {
  [#context pre() #body]
}

// 也可以这样写（方法三）
#let pre() = {
  let x = x-value.get()
  return x + 1
}

#let A(body) = {
  [#context pre() #body]
}

// 也可以这样写（方法四）
#let pre() = context{
  let x = x-value.get()
  return x + 1
}

#let A(body) = {
  [#pre() #body]
}
```

但是，如果在 `A` 中仍需对 `pre()` 的返回值进行其他操作，仅当 `x + 1 > 2` 时加数字前缀，则只能用方法二、三，不能用方法四，因为后者的返回值不是一个数。例如

```
// 可以这样写
#let pre() = {
  let x = x-value.get()
  return x + 1
}

#let A(body) = {
  context{
    let p = pre()
    if pre() > 2{[#p ]}
  }
  [#body]
}

// 不能这样写
#let pre() = context{
  let x = x-value.get()
  return x + 1
}

#let A(body) = {
  let p = pre()
  if p > 2{[#p ]} //error: cannot compare content and integer
  [#body]
}

// 也不能这样写
#let A(body) = {
  if context x-value.get() + 1 > 2{[#context{x-value.get()+1} ]} //error: expected boolean, found content
  [#body]
}
```

