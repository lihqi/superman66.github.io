---
title: 【读书笔记】JavaScript闭包的理解
date: 2016-03-20 00:41:34
tags:
- JavaScript学习笔记
---
以前一直都不能完全理解闭包的概念，于是就专门学习了下闭包，整理成这篇文章。
<!-- more -->
在学习JavaScript中，有一个概念一直困扰着我，它就是**闭包**。今天就彻底学习学习什么是闭包。
参考资料：
阮一峰老师的闭包教程: http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html)
# 为什么会有闭包？
由于JavaScript的变量作用域的特点，函数内部作用域可以访问外部作用域的变量，而外部作用域却无权访问内部作用域的局部变量。那如果我们想要在函数作用域外访问到函数内部的变量时，该怎么办。
因此解决的办法就是在函数的内部再创建一个函数，这样就可以在内部函数中访问函数的内部变量，这时候再将内部函数作为返回值的话，那么就可以在函数外去访问到函数的内部变量。这就是**函数的闭包**。
# 什么是闭包？
先来看看对于函数的闭包，经典的书籍都是怎么定义的：
**闭包是指有权访问另外一个函数作用域中的变量的函数。**-----《JavaScript高级程序设计（第3版）》
**函数对象可以通过作用域链相互关联起来，函数体内部的变量都可以保存在函数作用域内，这种特性在计算机科学文献中被称为闭包。**-----《JavaScript权威指南（第6版）》
**当函数可以记住并访问所有的词法作用域时，就产生了闭包，即使函数是在当前词法作用域之外执行。**-----《你不知道的JavaScript（上）》
发现了没有，上面三个关于**闭包**虽然描述的方式不同，但是对闭包定义的本质是相同的：**闭包就是能够读取其他函数内部变量的函数**。
阮老师将定义更加简洁得定义为：**定义在一个函数内部的函数**（由于在JavaScript中，只有函数内部的子函数才可以读取函数内部的变量）。
先来看一个简单的例子，清晰地展示了闭包：
```javascript
function foo(){
    var a = 2;
    function bar(){
        console.log(a);
    }
    return bar;
}
var baz = foo();
baz.bar();    //2 ——这就是闭包的效果，在foo()函数作用域却能访问到函数内部变量a的值
```
# 闭包有什么用途
1. 匿名自执行函数
2. 缓存
3. 实现封装
4. 实现面向对象中的对象

_关于闭包的用途，下一篇文章将详细介绍。_
# 闭包注意事项
## 1、闭包性能问题 
由于闭包会携带包含它的函数的作用域，因此会比其他函数占用更多的内存。过度使用闭包可能导致内存占用过多。
虽然V8等优化后的JavaScript引擎会尝试回收被闭包占用的内存。但也应避免大量使用闭包。
## 2、闭包与变量
JavaScript作用域导致了一个副作用，即闭包只能取得包含函数中的任何变量的最后一个值。闭包保存的是整个变量对象，而不是某个特殊的变量。如下面的例子：
```javascript
function createFunctions(){
	var result = new Array();
	for(var i = 0; i< 10; i ++){
		result[i] = function(){
			return i;
		}
	}
	return result;
}
console.log(createFunctions()[1]());    //10，其实数组的每个值都是输出10
```
这个函数返回了一个数组，从表面上看，每个函数都应该返回自己的索引值，但实际上却是每个函数都返回了10。这是为什么呢？
因为每个函数的作用域链中都包含着createFunctions()函数的活动对象，所以它们引用的都是同一个变量i。当createFunctions()函数返回后，变量i的值是10，此时每个函数都引用这保存变量i的统一变量对象，所以每个函数内部i的值都是10。
我们可以通过创建另外一个匿名函数来解决这个问题，如下所示
```javascript
function createFunctions() {
    var result = new Array();
    for (var i = 0; i < 10; i++) {
        result[i] = function(num) {
            return function(){
            	return num;
            };
        }(i);
    }
    return result;
}
```
改进之后，我们并没有直接将闭包赋值给数组，而是定义了一个匿名函数，并将立即执行该函数的结果赋值给数组。而这个匿名函数有一个参数，也就是最终函数要返回的值。在调用每个匿名函数时，我们传入变量i。在这个匿名函数内部，又创建并返回了一个访问num的闭包，这样一来，result数组中的每个函数都有自己的num变量的一个副本，因此就可以各自不同的值。
## 3、闭包与this对象
我们都知道`this`对象是基于函数运行时的执行环境绑定的，在全局函数中，`this`等于`window`。但是在匿名函数中，其执行环境具有全局局限性，因此'this'通常指的是'window'。来比较下下面两个例子：
```javascript
var name = 'The window';
var object = {
    name: 'My Object',
    getNameFunc: function() {
        return function() {
            return this.name;
        }
    }
}
console.log(object.getNameFunc()()); //The window
```
```javascript
var name = 'The window';
var object = {
    name: 'My Object',
    getNameFunc: function() {
        var self = this;
        return function() {
            return self.name;
        }
    }
}
console.log(object.getNameFunc()()); //My Object
```

# 闭包小测

```javascript
    function fun(n, o) {
        console.log(o);
        return {
            fun: function(m) {
                return fun(m, n);
            }
        };
    }
var a = fun(0);  a.fun(1);  a.fun(2);  a.fun(3);//undefined,?,?,?
var b = fun(0).fun(1).fun(2).fun(3);//undefined,?,?,?
var c = fun(0).fun(1);  c.fun(2);  c.fun(3);//undefined,?,?,?
问着三行，a、b、c分别输出什么
```
答案请[猛戳](http://www.cnblogs.com/xxcanghai/p/4991870.html)
参考资料：大部分人都会做错的经典JS闭包面试题 http://www.cnblogs.com/xxcanghai/p/4991870.html



