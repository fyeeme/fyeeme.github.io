---
layout:     post
title:      "php内存回收机制 - 引用计数基本知识"
subtitle:   "垃圾回收机制是一种动态存储分配方案。它会自动释放程序不再需要的已分配的内存块。自动回收内存的过程叫垃圾收集。垃圾回收机制可以让程序员不必过分关心程序内存分配，从而将更多的精力投入到业务逻辑"
date:       2017-01-03 12:00:00
author:     "Allen Lau"
header-img: "img/post-bg-01.jpg"
keywords: "php gc php内存回收"
tags: gc
categories: php
---

### 垃圾回收机制

* <a href="/php/2017/01/03/php-gc-refcount/">引用计数基本知识</a>
* <a href="/php/2017/01/03/php-gc-collecting-cycles/">回收周期(Collecting Cycles)</a>
* 性能方面考虑的因素

<h3 class="section-heading">1.1 引用计数基本知识</h3>

<p>每一个php变量存在一个叫做"zval"的变量容器中。 一个zval的变量容器，除了包含变量的类型和值，还包括了两个字节的额外信息。第一个是"is_ref"，是一个bool值，用来表示这个变量是否属于音容集合（reference set）。 通过这个字节，php引擎才能把普通变量和引用变量区分开来，由于php允许用户通过&来使用自定义的引用，zval变量容器的中还有一个内部引用计数机制，来优化内存使用。 第二个额外字节是"refcount", 用以表示只想这个zval标量容器的变量(symbal)的个数，所有的符号内存在一个字附表中，其中每一个符号都有作用域(scope),那些主脚本和每个函数或者方法也都有作用域。</p>

<p>当一个变量被赋值时.就会产生一个zval变量容器，例如下面这样:</p>

```php?start_inline=1
$val = "new string";
```

<p>在上例中，新的变量val,就是在当前作用域中生成的。并且生成了类型为 <a href="http://php.net/manual/zh/features.gc.refcounting-basics.php">string</a>和值为 new string的变量容器。再额外的两个字节中， “is_ref”被默认设置为false, 因为没有任何自定义的引用生成。 “refcount”被设定为1，因为这里只有一个变量使用了这变量容器。注意到当“refcount”的值是1时，“is_ref”的值总是false, 如果你已经安装了<a href="https://xdebug.org/">Xdebug</a>,你能通过调用函数xdebug_debug_zval()显示“refcount”和“is_ref”的值。</p>

```php?start_inline=1
xdebug_debug_zval('val');

//print result
val: (refcount=1, is_ref=0)='new string'
```

<p>把一个变量赋值给另一个变量将增加引用次数（refcount）.</p>

```php?start_inline=1
$a = "hello, fyee";
$b = $a;
xdebug_debug_zval('a');

//print result
a: (refcount=2, is_ref=0)='hello, fyee'
```

<p>这时， 引用次数变成了2， 因为同一个变量的容器被变量a 和变量 b 关联。 当没必要时，php不会自己复制已经生成的变量容器。变量容器在"refcount"变成0时，就会被销毁。当任何关联到某个变量的变量离开他的作用域（比如函数执行结束），或者对变量调用了函数unset()时， "refcount"就会减一， 下面的离子就能说明：</p>

```php?start_inline=1
$a = "hello, fyee";
$c = $b = $a;
xdebug_debug_zval('a');
unset($b, $c);
xdebug_debug_zval('a');

//print result
a: (refcount=3, is_ref=0)='hello, fyee'
a: (refcount=1, is_ref=0)='hello, fyee'
```
<p>如果我们在执行 unset($a);, 包含的类型和值的这边两容器就会从内存中删除。</p>

<h3 class="section-heading">1.2 复合类型（compound Types）</h3>

<p>当考虑像array 和object这样的复合类型时， 事情就稍微有点复杂了。 与标量类型的值不同， array和object类型的变量把他们的成员或者属性存在自己的符号中。这意味着下面的例子将会生成三个zval变量容器。</p>

```php?start_inline=1
$fruits = array('apple'=>'apple', 'banana'=>'banana');
xdebug_debug_zval('fruits');

//print result
fruits: (refcount=1, is_ref=0)=array (
	'apple' => (refcount=1, is_ref=0)='apple', 
	'banana' => (refcount=1, is_ref=0)='banana'
)
```

<p>这桑格变量的容器是： fruits, apple, banana. 增加和减少“refcount”的规则和上面提到的一样，下面我们在数组中再添加一个元素，并且把他的值设置成为数组中已经存在的值：</p>

```php?start_inline=1
$fruits = array('apple'=>'apple', 'banana'=>'banana');
$furits['orange'] = $fruits['apple'];
xdebug_debug_zval('fruits');

//print result
	fruits: (refcount=1, is_ref=0)=array (
	'apple' => (refcount=2, is_ref=0)='apple', 
	'banana' => (refcount=1, is_ref=0)='banana', 
	'orange' => (refcount=2, is_ref=0)='apple'
)
```
<a href="#">
    <img src="{{ site.baseurl }}/img/php/php-simple-array2.png" alt="php simple array">
</a>

<p>从以上的xdebug输出信息，我们可以看到原有的数组元素和新添加的的数组元素关联到同一个"refcount"2的zval变量容器。尽管Xdebug的输出显示两个值为'apple'的 zval变量容器,其实是同一个，函数xdebug_debug_zval()不显示这个信息，但可以通过显示内存指针信息来看到</p>

<p>删除数组的一个元素，就是类似的从作用域中删除一个变量，删除后，书祖宗的这个元素所在的容器的"refcount"值减少，同样当"refcount"为0时， 这个变量容器就从内从删除，下面的例子可以说明：</p>

```php?start_inline=1
$fruits = array('apple'=>'apple', 'banana'=>'banana');
$fruits['orange'] = $fruits['apple'];
unset($fruits['apple'], $fruits['banana']);
xdebug_debug_zval('fruits');

//print result
	fruits: (refcount=1, is_ref=0)=array ('orange' => (refcount=1, is_ref=0)='apple')
```

<p>现在，当我们添加一个数组本身作为这个元素时，事情将变得有趣，下个例子说明这个。例中加入了引用操作符，否则php将生成一个复制。</p>


```php?start_inline=1
$fruits = array('apple');
$fruits[] = &$fruits;
xdebug_debug_zval('fruits');

//print result
	fruits: (refcount=2, is_ref=1)=array (
		0 => (refcount=1, is_ref=0)='apple',
	 	1 => (refcount=2, is_ref=1)=...
	 )
```
<a href=":;">
    <img src="{{ site.baseurl }}/img/php/php-loop-array.png" alt="php simple array">
</a>

<p>能看到数组变量（fruit）同时也是这个数组的第二个元素指向变量容器的"refcount"为2，上面的输出结果中"..."说明发生了递归操作，显然这总情况下意味着"..."指向了原始数组。</p>

<p>跟刚刚的一样，对一个变量调用unset,将喊出这个符号，且它指向的变量容器的引用次数也减1.所以，如果我们在执行完上面的代码后，对变量$fruits调用unset,那么对变量$fruits 和数组元素”1“所指向的变量容器的引用次数减1，从2变成了1.下面的例子可以说明：
</p>

<h3 class="section-heading">1.3 清理变量容器的问题(Cleanup Problems) </h3>

<p>尽管不再有某个作用域中的任何符号指向这个结构(就是变量容器)，由于数组元素“1”仍然指向数组本身，所以这个容器不能被清除 。因为没有另外的符号指向它，用户没有办法清除这个结构，结果就会导致内存泄漏。庆幸的是，php将在脚本执行结束时清除这个数据结构，但是在php清除之前，将耗费不少内存。如果你要实现分析算法，或者要做其他像一个子元素指向它的父元素这样的事情，这种情况就会经常发生。当然，同样的情况也会发生在对象上，实际上对象更有可能出现这种情况，因为对象总是隐式的被引用。</p>

<p>如果上面的情况发生仅仅一两次倒没什么，但是如果出现几千次，甚至几十万次的内存泄漏，这显然是个大问题。这样的问题往往发生在长时间运行的脚本中，比如请求基本上不会结束的守护进程(deamons)或者单元测试中的大的套件(sets)中。后者的例子：在给巨大的eZ(一个知名的PHP Library) 组件库的模板组件做单元测试时，就可能会出现问题。有时测试可能需要耗用2GB的内存，而测试服务器很可能没有这么大的内存。
</p>

<a href="/php/2017/01/03/php-gc-collecting-cycles">
    
</a>
