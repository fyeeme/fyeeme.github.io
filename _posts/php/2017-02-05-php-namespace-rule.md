---
layout:     post
title:      "php命名空间解析规则, 基础篇"
subtitle:   "命名空间是php5.3中最重要的改进之一，对于C#和Java的开发者可能对他们非常熟悉。而这将很可能是的php的应用结构更加合理"
date:       2017-01-03 12:00:00
author:     "Simon Lau"
header-img: "img/post-bg-01.jpg"
keywords: "php namespace"
tags: namespace
categories: php
---

<p>首先你可能会问，为什么我们需要命名空间所以，什么是命名空间？</p>

<p>随着项目中php库代码量的增加，我们很可能需要重用一些我们已经声明方法或者类，而这个问题在我们引入
第三的组建或者插件的时候也同样会碰到，试想一下，我们需要两个或者多个类实现 'Datebase' 要如何做呢</p>

<p>php不允许两个函数或者类出现相同的名字，否则会产生一个致命错误。为了解决这个问题WordPress使用前缀来处理这个问题，给每一个名字添加前缀 'PW_'。所以说命名空间最明确的目的就是为了解决重名问题。</p>

<h2>如何定义命名空间</h2>


<p>默认的所有的常量、类和函数都放在全局的命名空间，命名空间使用关键字'namespace'来定义,php不允许嵌套定义命名空间，或者为同一个代码块声明多个命名空间（只有最后一个才会别识别）,但是我们可以在同一个文件中定义不同的命名空间</p>


```php?start_inline=1
<?php 
//below fn is under the global namespace
function fn(){}

namespace Allen;
//below fn1 is under the Allen\Blog namespace
function fn1(){}
```

<h2>子命名空间</h2>

<p>php允许你定义命名空间的层次结构，因此代码可以根据细分层次，子命名空间使用'\'分割</p>

<ul>
<li>Allen\User</li>
<li>Allen\Blog</li>
<li>Allen\Blog\Manager</li>
</ul>

<h2>调用命名空间</h2>

<p>下面的代码我们将演示如何调用某个命名空间下的常亮、方法和类</p>

test1.php

```php?start_inline=1
<?php
namespace Allen\Lib1;

const MYCONST = 'App\Lib1\MYCONST';

function fn()
{
    return __FUNCTION__;
}

class Test
{
    static function fn()
    {
        return __METHOD__;
    }
}
```

<p>为了调用这些代码，我们创建test2.php</p>

```php?start_inline=1
<?php

header('Content-type: text/plain');

require_once('test4.php');

echo Allen\Lib1\MYCONST ."\n";

echo Allen\Lib1\fn() ."\n";

echo Allen\Lib1\Test::fn() ."\n";
```

结果
```html
App\Lib1\MYCONST
Allen\Lib1\fn
Allen\Lib1\Test::fn
```


<p> 全路径名称（带有命名空间的觉得路径类、函数、常亮）显然很长，但仍然好于App-Lib1-MYCONST,下一节我们将讨论别名(aliasing),以及命名空间是如何解析的</p>
