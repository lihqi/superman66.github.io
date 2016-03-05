---
title: 【JavaScript高程学习笔记】面向对象的程序设计之继承
date: 2016-03-06 01:27:43
tags:
- JavaScript高级程序设计学习笔记
- 面向对象
- 继承
---
本篇文章是学习《Javascript高级程序设计》中关于继承这一部分的学习笔记。
<!-- more -->
#继承
许多OO语言都支持两种继承方式：接口继承和实现继承。而ECMAScript只支持**实现继承**，而且实现继承主要是依靠原型链来实现的。
## 一、原型链
ECMAScript中继承的基本思想是利用原型让一个引用类型继承另外一个引用类型的属性和方法。
让我们简单回顾一下原型、构造函数和实例之间的关系。
每个构造函数都有一个原型对象`Prototype`，原型对象包含一个指向构造函数的指针，即`constructor`属性，这个属性指向的是`prototype`属性所在的函数（构造函数）。而构造函数的实例则包含一个指向原型对象的内部指针。
那么如果让原型对象等于另一个类型的实例，则此时原型对象将包含一个指向另一个原型的指针，相应地，另一个原型中也包含着一个指向另外一个构造函数的指针。假如另外一个原型又是另外一个类型的实例，那么上述关系依然成立，如此层层推进，就构成原型链。如下面的例子：
```javascript
	function Animal(){ //采用构造函数模式创建对象
		this.live = true;
		this.run = function(){
			console.log('I can run');
		}
	}
	Animal.prototype = {
		constructor: Animal,
		eat: function(){
			console.log('I can eat');
		}
	}
	function Human(){
		this.isHuman = true;
	}
	Human.prototype = new Animal(); //将父类Animal的实例赋给子类Human的原型对象实现继承

	function Boy(){
		this.sex = 'boy';
	}
	Boy.prototype = new Human();	//将父类Human的实例赋给子类Boy的原型对象实现继承,子类 Boy就继承了Human以及Animal
	var human = new Human();
	var boy = new Boy();
	console.log(human)
	console.log(boy);	// 
	boy.run();    	// I can run
```
**1、原型链搜索机制**
从打印结果，如下图所示，可以看出，Boy的继承原型链，Human->Animal->Object（因为所有的对象都继承于Object）。所以Boy对象在执行run()方法的时候，能输出“I can run”的结果。而boy能找到run方法是基于`原型链搜索机制`。当boy调用run()方法时，首先会先在Boy的实例去寻找该方法或者属性。如果找不到该方法或属性，则会继续搜索实例的原型。如果再找不到，就继续往原型的原型寻找。在找不到的情况下，会一直搜索到原型链的末端才会停止，一般是Object对象。

**2、确定原型和实例的关系**
有两种方式可以确定原型和实例之间的关系。
* instanceof操作符
* isPrototypeOf()方法
```javascript
        console.log(boy instanceof Object); //true
	console.log(boy instanceof Animal); //true
	console.log(boy instanceof Human); //true

	console.log(Object.prototype.isPrototypeOf(boy)); //true
	console.log(Animal.prototype.isPrototypeOf(boy)); //true
	console.log(Human.prototype.isPrototypeOf(boy)); //true
```
**3、重写父类方法或者添加新方法**
如果需要重写父类方法或者添加新方法，一定要放在**替换原型的语句之后**，而且在重写父类方法或者添加新方法的时候，**不能使用对象字面量创建**。因为如果使用对象字面量创建，就相当于重新创建了原型链，会导致原有的原型链失效。
```javascript
    function Animal(){ //采用构造函数模式创建对象
		this.live = true;
		this.run = function(){
			console.log('I can run');
		}
	}
	Animal.prototype = {
		constructor: Animal,
		eat: function(){
			console.log('I can eat');
		}
	}
	function Human(){
		this.isHuman = true;
	}
	Human.prototype = new Animal(); //将父类Animal的实例赋给子类Human的原型对象实现继承

	Human.prototype.eat = function(){	//重写父类方法，语句要放在原型替换之后。
		console.log("I can eat override");
	}

	Human.prototype.speak = function(){  //添加新方法，且不能使用对象字面量添加方法
		console.log('Human can speak');
	}
	Human.prototype = {	 //不能使用对象字面量添加
		speak: function(){  
		console.log('Human can speak');
	}
```
**4、原型链存在的问题**
跟之前创建对象的原型模式一样，正是由于原型共享被所有实例共享的特性，导致了在听过原型继承的时候，父类的属性是子类的原型，因此所有子类的实例对父类的属性操作时，都会影响到子类所有实例。如下面例子：
```javascript
    function Animal(){ //采用构造函数模式创建对象
		this.live = true;
		this.gender = [''];
		this.run = function(){
			console.log('I can run');
		}
	}
    	Animal.prototype = {
		constructor: Animal,
		eat: function(){
			console.log('I can eat');
		}
	}
	function Human(){
		this.isHuman = true;
	}
	Human.prototype = new Animal(); //将父类Animal的实例赋给子类Human的原型对象实现继承
        var human1 = new Human();
        var human1 = new Human();
        human1.gender.push('male');
        console.log(human2.gender);     //此时输出 male
```
这个例子中，human1对父类（即Human的原型）的gender属性添加了一个值“male”,由于原型的共享性，导致了Human的另外一个实例human2调用gender属性输出的值是刚刚human1添加过值的。
原型链的第二个问题：在创建子类实例的时候，无法在不影响所有子类实例的情况下，向父类的构造函数传递参数。
## 2、继承的方法
### 1、借用构造函数
实现思想：在子类内部调用超类构造函数，使用apply（）和call()
```
function SuperType(name){
    this.color = ['red', 'blue', 'white'];
    this.getName = function(){
        console.log(name);
    }
}
function SubType(){
    //继承了SuperType
    SuperType.call(this, 'Superman');
}
```
使用借用构造函数继承可以保证了父类的属性和方法对每个子类而言都是独立的。同时还支持传递参数。
但是该方法依然存在着问题，由于是通过在子类内部实例化父类构造函数的方式来实现继承，应该函数的复用就无从谈起。而且在父类原型中定义的方法，对于子类而言也是不可见的，结果导致所有类型都只能使用构造函数模式。因此很少单独使用借用构造函数模式。

### 2、组合继承
组合继承也称伪经典继承，是将借用构造函数和原型链的技术组合到一块。这种方式是最常用的继承模式。
实现思路：使用原型链对**原型属性**和**原型方法**的继承，而通过借用构造函数来实现对**实例属性**的继承。如下面例子
```javascript
function SuperType(name){
    this.name = name;
    this.colors = ['red', 'blue', 'green'];
}
SuperType.prototype.sayName = function(){
    alert(this.name);
}
function SubType(name, age){
    SuperType.call(this, name);
    this.age = age;
}
//继承方法
SubType.prototype = new SuperType();

SubType.prototype.sayAge = function(){
    console.log(this.age);
}
var sub1 = new SubType('superman1', '1');
var sub2 = new SubType('superman2', '2');
sub1.colors.push('black');
console.log(sub1.colors);    // ['red', 'blue', 'green', 'black']
sub1.sayName();    //superman1

sub2.colors.push('green');
console.log(sub2.colors);    // ['red', 'blue', 'green', 'green']
sub2.sayName();    //superman2
```
通过这种方式，即可以让子类的实例分别拥有自己的属性，又可以使用同样的方法。
### 3、原型式继承
实现思路：借助原型可以基于已有的对象创建新对象。如下面例子：
```javascript
function object(o){
    function F(){
        F.prototype = o;
        return new F();
    }
}
var person = {
    name: 'superman',
    age: 22,
    friends: ['1', '2', '3', '4']
}
var anotherPerson = object(person);
```
### 4、寄生式继承
### 5、寄生组合式继承


