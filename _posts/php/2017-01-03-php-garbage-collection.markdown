---
layout:     post
title:      "php内存回收机制"
subtitle:   "垃圾回收机制是一种动态存储分配方案。它会自动释放程序不再需要的已分配的内存块。自动回收内存的过程叫垃圾收集。垃圾回收机制可以让程序员不必过分关心程序内存分配，从而将更多的精力投入到业务逻辑"
date:       2016-01-03 12:00:00
author:     "Simon Lau"
header-img: "img/post-bg-01.jpg"
tags: gc
categories: php
---

* 引用计数基本知识
* 回收周期(Collecting Cycles)
* 性能方面考虑的因素

<h2 class="section-heading">引用计数基本知识</h2>

<p>每一个php变量存在一个叫做"zval"的变量容器中。 一个zval的变量容器，除了包含变量的类型和值，还包括了两个字节的额外信息。第一个是"is_ref"，是一个bool值，用来表示这个变量是否属于音容集合（reference set）。 通过这个字节，php引擎才能把普通变量和引用变量区分开来，由于php允许用户通过&来使用自定义的引用，zval变量容器的中还有一个内部引用计数机制，来优化内存使用。 第二个额外字节是"refcount", 用以表示只想这个zval标量容器的变量(symbal)的个数，所有的符号内存在一个字附表中，其中每一个符号都有作用域(scope),那些主脚本和每个函数或者方法也都有作用域。</p>

<p>当一个变量被赋值时.就会产生一个zval变量容器，例如下面这样:</p>

```php?start_inline=1
$val = "new string";
<?php
class User extends Model 
{
    protected $variable = null;

    public function getVariable()
    {
        return $this->variable;
    }
}
```

<p>在上例中，新的变量val,就是在当前作用域中生成的。并且生成了类型为 <a href="http://php.net/manual/zh/features.gc.refcounting-basics.php">string</a>和值为 new string的变量容器。再额外的两个字节中， “is_ref”被默认设置为false, 因为没有任何自定义的引用生成。 “refcount”被设定为1，因为这里只有一个变量使用了这变量容器。注意到当“refcount”的值是1时，“is_ref”的值总是false, 如果你已经安装了<a href="https://xdebug.org/">Xdebug</a>,你能通过调用函数xdebug_debug_zval()显示“refcount”和“is_ref”的值。</p>

{% highlight php  startinline=true %}
xdebug_debug_zval('val');

//print result
val: (refcount=1, is_ref=0)='new string'
{% endhighlight  %}
<p>A Chinese tale tells of some men sent to harm a young girl who, upon seeing her beauty, become her protectors rather than her violators. That's how I felt seeing the Earth for the first time. I could not help but love and cherish her.</p>

<p>For those who have seen the Earth from space, and for the hundreds and perhaps thousands more who will, the experience most certainly changes your perspective. The things that we share in our world are far more valuable than those which divide us.</p>

<h2 class="section-heading">The Final Frontier</h2>

<p>There can be no thought of finishing for ‘aiming for the stars.’ Both figuratively and literally, it is a task to occupy the generations. And no matter how much progress one makes, there is always the thrill of just beginning.</p>

<p>There can be no thought of finishing for ‘aiming for the stars.’ Both figuratively and literally, it is a task to occupy the generations. And no matter how much progress one makes, there is always the thrill of just beginning.</p>

<blockquote>The dreams of yesterday are the hopes of today and the reality of tomorrow. Science has not yet mastered prophecy. We predict too much for the next year and yet far too little for the next ten.</blockquote>

<p>Spaceflights cannot be stopped. This is not the work of any one man or even a group of men. It is a historical process which mankind is carrying out in accordance with the natural laws of human development.</p>

<h2 class="section-heading">Reaching for the Stars</h2>

<p>As we got further and further away, it [the Earth] diminished in size. Finally it shrank to the size of a marble, the most beautiful you can imagine. That beautiful, warm, living object looked so fragile, so delicate, that if you touched it with a finger it would crumble and fall apart. Seeing this has to change a man.</p>

<a href="#">
    <img src="{{ site.baseurl }}/img/post-sample-image.jpg" alt="Post Sample Image">
</a>
<span class="caption text-muted">To go places and do things that have never been done before – that’s what living is all about.</span>

<p>Space, the final frontier. These are the voyages of the Starship Enterprise. Its five-year mission: to explore strange new worlds, to seek out new life and new civilizations, to boldly go where no man has gone before.</p>

<p>As I stand out here in the wonders of the unknown at Hadley, I sort of realize there’s a fundamental truth to our nature, Man must explore, and this is exploration at its greatest.</p>

<p>Placeholder text by <a href="http://spaceipsum.com/">Space Ipsum</a>. Photographs by <a href="https://www.flickr.com/photos/nasacommons/">NASA on The Commons</a>.</p>
