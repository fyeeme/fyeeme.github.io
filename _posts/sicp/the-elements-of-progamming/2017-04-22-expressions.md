---
layout:     post
title:      "我为什么要看 《计算机程序的构造和解释》？"
subtitle:   "这本书从理论上讲解了计算机程序的创建, 执行和研究。也是我一直所好奇的"
date:       2017-01-03 12:00:00
author:     "Allen Lau"
header-img: "img/post-bg-01.jpg"
keywords: "sicp Lisp 计算机程序的构造和解释"
tags: scip
categories: sicp
---


* 表达式， 组合
* 变量和环境

### 1.表达式和组合
<p>表达式是讲数字通过运算符的操作后进行合并产生的结果，例如：</p>

```[shell]
(+ 12 28)
40
```

```[shell]
(- 1000 334)
666
```

```[shel]
(* 5 99)
495
```

```[shell]
(/ 10 5)
2
```

<p>像这样的表达式，使用括号来包裹起来，表示一个过程应用，叫做组合。最左边的元素叫做操作符，其他的元素叫做操作数，组合的值是将操作数经过操作符应用产生的值。</p>

<p>像这样的将操作符放在表达式的最前面，称之为前缀符号，初次看起来有些困惑，因为它和一边的数学表达式 有些不一样，前缀符号有一些优点，其中的一个是它能够适用任意个参数的表达式，比如下面: </p>

```[shell]
(+  10 5 25 18 22 29)
61
```

```[shell]
(*  25 4 100)
61
```

<p>像这样的表达式，不会有任何歧义，因为操作符在左边，表达式被括号先点，第二个优点是，前缀操作符允许组合表达式的嵌套，也就是说，组合内的元素本身也是可以使表达式</p>

(+ (* 3 5) (- 10 6))
19

### 2.命名和环境

<p>在Schame的方言Lisp中，我们可以通过关键字 'define'来命名变量,例如：</p>

```[shell]
(define size 2)
```

<p>一旦我们将2分配给 size,我们就可以这样引用 2:</p>


```[shell]
size
2

(* size 5)
10
```
