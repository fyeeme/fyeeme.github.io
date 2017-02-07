---
layout:     post
title:      "php命名空间解析规则-Importing, Aliases, and Name Resolution"
subtitle:   "上一节我们讨论php命名空间的用法，这一届我们将介绍如何使用'use'关键字，以及php如何解析命名空间"
date:       2017-01-03 12:00:00
author:     "Allen Lau"
header-img: "img/post-bg-01.jpg"
keywords: "php namespace"
tags: namespace
categories: php
---

<p>首先我们定义两个相同的php代码块，唯一的区别便是他们的命名空间不同</p>

lib1.php

```php?start_inline=1
<?php
// application library 1
namespace App\Lib1;

const MYCONST = 'App\Lib1\MYCONST';

function MyFunction() {
	return __FUNCTION__;
}

class MyClass {
	static function WhoAmI() {
		return __METHOD__;
	}
}
```

lib2.php
```php?start_inline=1
<?php
// application library 2
namespace App\Lib2;

const MYCONST = 'App\Lib2\MYCONST';

function MyFunction() {
	return __FUNCTION__;
}

class MyClass {
	static function WhoAmI() {
		return __METHOD__;
	}
}
```
<p>我们需要先了解几个php的概念</p>

<ul>
<li>完全限定名称Fully qualified name</li>
<p>
名称中包含命名空间分隔符，并以命名空间分隔符开始的标识符，例如 \App\Lib1\MYCONST, \App\Lib2\MyFunction(),。 namespace\Foo 也是一个完全限定名称。
完全限定名称没有歧义，第一个'\'标示'root'，即全局命名空间，
</p>

<li>限定名称Qualified name</li>
<p>名称中含有命名空间分隔符的标识符，例如 Lib1\MyFunction().</p>

<li>非限定名称Unqualified name</li>
<p>名称中不包含命名空间分隔符的标识符，例如 MyFunction</p>
</ul>


<h2>使用相同的命名空间</h2>

<p>我们先来看看下面的代码</p>


nsapp1.php
```php?start_inline=1
<?php
namespace App\Lib1;

require_once('lib1.php');
require_once('lib2.php');

header('Content-type: text/plain');
echo MYCONST . "\n";
echo MyFunction() . "\n";
echo MyClass::WhoAmI() . "\n";
?>
```
<p>虽然我们同时引入了lib1.php 和lib2.php,但是 MYCONST, MyFunction, MyClass只会引用lib1.php中的代码
这是因为nsapp1.php 自身的命名空间和lib1.php相同</p>

结果:
```html
App\Lib1\MYCONST
App\Lib1\MyFunction
App\Lib1\MyClass::WhoAmI
```
<h2>命名空间导入</h2>

<p>我们使用'use' 关键字导入命名空间</p>

```php?start_inline=1
<?php
use App\Lib2;

require_once('lib1.php');
require_once('lib2.php');

header('Content-type: text/plain');
echo Lib2\MYCONST . "\n";
echo Lib2\MyFunction() . "\n";
echo Lib2\MyClass::WhoAmI() . "\n";
?>
```
<p>我们可以在一个或多个'use'并使用'分号'隔开。在这个例子中我们导入了命名空间App\Lib1，现在我们无法直接引用MYCONST, MyFunction or MyClass，因为我们的代码在全局的空间无法查找到它们。然而我们添加了前缀'Lib2'后，此时变成了限定名称；php会搜索所有导入的命名空间查找，直到找到它们</p>

<h2>命名空间的别名</h2>

<p>命名空间别名相当有用，我们可以将长的命名空间映射为短的命名空间</p>

```php?start_inline=1
<?php
use App\Lib1 as L;
use App\Lib2\MyClass as Obj;

header('Content-type: text/plain');
require_once('lib1.php');
require_once('lib2.php');

echo L\MYCONST . "\n";
echo L\MyFunction() . "\n";
echo L\MyClass::WhoAmI() . "\n";
echo Obj::WhoAmI() . "\n";
?>
```

<p>第一个'use'将 App\Lib1定义为'L'.当使用'L'作为限定名称时，php会在编译时将'L'翻译为App\Lib1，因此我们可以使用L\MYCONST和L\MyFunction而非完全限定名称</p>

<p>第二个'use'则将App\Lib2命名空间下的MyClass映射为 Ojb,而这个用法仅对类有效。我们现在可以使用 new Obj() 创建实例 或者运行静态方法</p>

结果：
```html
App\Lib1\MYCONST
App\Lib1\MyFunction
App\Lib1\MyClass::WhoAmI
App\Lib2\MyClass::WhoAmI
```

<h2>名称解析遵循下列规则：</h2>


<p>1. 对完全限定名称的函数，类和常量的调用在编译时解析。例如 new \A\B 解析为类 A\B。</p>
<p>2. 所有的非限定名称和限定名称（非完全限定名称）根据当前的导入规则在编译时进行转换。例如，如果命名空间 A\B\C 被导入为 C，那么对 C\D\e() 的调用就会被转换为 A\B\C\D\e()。</p>
<p>3. 在命名空间内部，所有的没有根据导入规则转换的限定名称均会在其前面加上当前的命名空间名称。例如，在命名空间 A\B 内部调用 C\D\e()，则 C\D\e() 会被转换为 A\B\C\D\e() 。</p>
<p>4. 非限定类名根据当前的导入规则在编译时转换（用全名代替短的导入名称）。例如，如果命名空间 A\B\C 导入为C，则 new C() 被转换为 new A\B\C() 。</p>
<p>5. 在命名空间内部（例如A\B），对非限定名称的函数调用是在运行时解析的。例如对函数 foo() 的调用是这样解析的：</p>
	1.在当前命名空间中查找名为 A\B\foo() 的函数
	2.尝试查找并调用 全局(global) 空间中的函数 foo()。
<p>6. 在命名空间（例如A\B）内部对非限定名称或限定名称类（非完全限定名称）的调用是在运行时解析的。下面是调用 new C() 及 new D\E() 的解析过程： new C()的解析:</p>
	1.在当前命名空间中查找A\B\C类。
	2.尝试自动装载类A\B\C。

<p>new D\E()的解析:</p>
	1.在类名称前面加上当前命名空间名称变成：A\B\D\E，然后查找该类。
	2.尝试自动装载类 A\B\D\E。

<p>为了引用全局命名空间中的全局类，必须使用完全限定名称 new \C()。</p>