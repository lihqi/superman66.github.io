---
title: 深入理解JavaScript中的call和apply
date: 2016-04-20 23:20:35
tags:
- JavaScript
---
本文利用主动提问的学习方法来学习JavaScript中的call和apply
<!-- more -->
#  学习之前的主动提问
昨天get的新技能，通过主动提问来学习。
![](http://7xr6yj.com1.z0.glb.clouddn.com/aritcle-review-insight-3.png)
## call和apply是什么
`call`和`apply`都是属于Function.prototype的一个方法。它是在JavaScript引擎内实现的，因为是原型上的方法，因此对于每个Function对象的实例，都共享这两个方法。
## call和apply有什么作用
根据MDN上对于JavaScript.call()的定义
> call()方法在使用一个指定的this值和若干个指定的参数值的前提下调用某个函数或方法

`apply`和`call`作用一样。也就是说它们具有动态改变函数运行时的上下文，可以将一个函数的对象上下文从初始的上下文改变为由指定的this值所指定的对象。
## call和apply的用法
`apply`和`call`的作用与用法几乎一样，它们的区别就在于方法**传递的参数不同**。`call()`传递的是参数是任意的，而`apply()`传递的参数必须为数组
### call()语法
> fun.call(thisArg[, arg1[, arg2[, ...]]])

#### thisArg
在fun函数运行时指定的this值，需要注意的是，指定的this值并不一定是该函数执行时真正的this值，如果这个函数处于**非严格模式下**，则指定为null和undefined的this值会指定指向全局对象（在浏览器中则是window对象），同时值为原始值（Number，string，boolean）的this值会指向该原始值的自动包装对象。

#### arg1, arg2, arg3, ...
传递给fun函数的指定的参数列表
### apply()语法
> fun.call(thisArg[ arg1, arg2,...])

## call和apply的用法
talk is cheap，show me the code。让我们先抛开上面复杂的解释，通过代码来解释。写一个hello world:
```javascript
;(function () {
	var  printA = {
		print: function(p1, p2){
		console.log(p1 +" "+ p2);
		}
	}
	
	var printB = {
		print: function(p1,p2){
			printA.print.call(this, p1, p2);	
		}	
	}
	printB.print('hello', 'worldB');	//hello worldB
})();
```
最后的输出结果是hello worldB。很显然，在printB.print方法中，并没有输出结果的console.log()语句，但是在printB.print函数内部，通过printA.print.call()的方式，借用了printA.print方法。
call就是借用别人的方法、对象来调用，就像调用自己的方法一样。
## call和apply的应用场景
这里引用Mozilla的例子
### 使用call方法调用父构造函数
```javascript
function Product(name, price) {
	  this.name = name;
	  this.price = price;

	  if (price < 0) {
	    throw RangeError('Cannot create product ' +
	                      this.name + ' with a negative price');
	  }

	  return this;
	}

	function Food(name, price) {
	  Product.call(this, name, price);
	  this.category = 'food';
	}

	Food.prototype = Object.create(Product.prototype);
	Food.prototype.constructor = Food; // Reset the constructor from Product to Food

	function Toy(name, price) {
	  Product.call(this, name, price);
	  this.category = 'toy';
	}

	Toy.prototype = Object.create(Product.prototype);
	Toy.prototype.constructor = Toy; // Reset the constructor from Product to Toy

	var cheese = new Food('feta', 5);
	var fun = new Toy('robot', 40);
```
### 使用call方法调用匿名函数
在下例中的for循环体内，我们创建了一个匿名函数，然后通过调用该函数的call方法，将每个数组元素作为指定的this值执行了那个匿名函数。这个匿名函数的主要目的是给每个数组元素对象添加一个print方法，这个print方法可以打印出各元素在数组中的正确索引号。当然，这里不是必须得让数组元素作为this值传入那个匿名函数（普通参数就可以），目的是为了演示call的用法。
```javascript
var animals = [
  {species: 'Lion', name: 'King'},
  {species: 'Whale', name: 'Fail'}
];
 
for (var i = 0; i < animals.length; i++) {
  (function (i) {
    this.print = function () {
      console.log('#' + i  + ' ' + this.species + ': ' + this.name);
    }
    this.print();
  }).call(animals[i], i);
}
```

### 使用call方法调用匿名函数并且指定上下文的'this'
在下面的例子中，当调用 greet 方法的时候，该方法的 this 值会绑定到 i 对象。

```javascript
function greet() {
  var reply = [this.person, 'Is An Awesome', this.role].join(' ');
  console.log(reply);
}
 
var i = {
  person: 'Douglas Crockford', role: 'Javascript Developer'
};
 
greet.call(i); // Douglas Crockford Is An Awesome Javascript Developer
```

## 参考资料
>关于javascript中apply()和call()方法的区别：http://www.cnblogs.com/fighting_cp/archive/2010/09/20/1831844.html
>Function.prototype.call():：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call