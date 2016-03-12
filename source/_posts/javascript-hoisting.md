---
title: 深入理解变量声明提升和函数声明提升
date: 2016-03-05 15:08:35
tags:
- JavaScript学习笔记
---
最近做题目遇到了关于变量声明提升和函数声明提升的知识点，觉得掌握得不是非常好，因此特地又翻开了犀牛书，重新深入学习，并整理成文章。
<!-- more -->
# 变量声明提升
## 1、变量定义
可以使用var定义变量，变量如果没有赋值，那变量的初始值为`undefined`。
## 2、变量作用域
变量作用域指变量起作用的范围。变量分为全局变量和局部变量。全局变量在全局都拥有定义；而局部变量只能在函数内有效。
在函数体内，同名的局部变量或者参数的优先级会高于全局变量。也就是说，如果函数内存在和全局变量同名的局部变量或者参数，那么全局变量将会被局部变量覆盖。
**所有不使用var定义的变量都视为全局变量**
## 3、函数作用域和声明提前
JavaScript的函数作用是指在函数内声明的所有变量在函数体内始终是有定义的，也就是说**变量在声明之前已经可用**，所有这特性称为`声明提前（hoisting）`，即JavaScript函数里的所有声明（只是声明，但不涉及赋值）都被提前到函数体的顶部，而变量赋值操作留在原来的位置。如下面例子：
_注释：`声明提前`是在JavaScript引擎的预编译时进行，是在代码开始运行之前。_
```javascript
var scope = 'global';
function f(){
    console.log(scope);
    var scope = 'local';
    console.log('scope');
}
```
由于函数内声明提升，所以上面的代码实际上是这样的
```javascript
var scope = 'global';
function f(){
    var scope;    //变量声明提升到函数顶部
    console.log(scope);
    scope = 'local';    //变量初始化依然保留在原来的位置
    console.log(scope);
}
```
经过这样变形之后，答案就就非常明显了。由于scope在第一个console.log(scope)语句之前就已经定义了，但是并没有赋值，因此此时scope的指是`undefined`.第二个console.log(scope)语句之前，scope已经完成赋值为'local'，所以输出的结果是`local`。

# 函数声明提升
## 1、函数的两种创建方式
* 函数声明
* 函数表达式

**函数声明语法**
```javascript
f('superman');
function f(name){
    console.log(name);
}
```
运行上面的程序，控制台能打印出`supemran`。
**函数表达式语法**
```javascript
f('superman');
var f= function(name){
    console.log(name);
}
```

运行上面的代码，会报错`Uncaught ReferenceError: f is not defined(…)`,错误信息显示说f没有被定义。
 为什么同样的代码，函数声明和函数表达式存在着差异呢？
这是因为，函数声明有一个非常重要的特征：`函数声明提升`，函数声明语句将会被提升到外部脚本或者外部函数作用域的顶部（是不是跟变量提升非常类似）。正是因为这个特征，所以可以把函数声明放在调用它的语句后面。如下面例子，最终的输出结果应该是什么？：
```javascript    
var getName = function(){
    console.log(2);
}
function getName (){
    console.log(1);
}
getName();
```
可能会有人觉得最后输出的结果是`1`。让我们来分析一下，这个例子涉及到了`变量声明提升`和`函数声明提升`。正如前面说到的函数声明提升，函数声明`function getName(){}`的声明会被提前到顶部。而函数表达式`var getName = function(){}`则表现出变量声明提升。因此在这种情况下，getName也是一个变量，因此这个变量的声明也将提升到底部，而变量的赋值依然保留在原来的位置。需要注意的是，**函数优先，虽然函数声明和变量声明都会被提升，但是函数会首先被提升，然后才是变量。**因此上面的函数可以转换成下面的样子:
```javascript
function getName(){    //函数声明提升到顶部
    console.log(1);
}
var getName;    //变量声明提升

getName = function(){    //变量赋值依然保留在原来的位置
    console.log(2);
}
getName();    // 最终输出：2
```
所以最终的输出结果是：`2`。在原来的例子中，函数声明虽然是在函数表达式后面，但由于函数声明提升到顶部，因此后面getName又被函数表达式的赋值操作给覆盖了，所以输出`2`。









