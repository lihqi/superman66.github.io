---
title: 【JavaScript高程学习笔记】面向对象的程序设计之对象
date: 2016-02-26 00:45:53
tags: 
- JavaScript高级程序设计学习笔记
- JavaScript
---
本篇文章是学习《Javascript高级程序设计》中关于对象这一部分的学习笔记。
<!-- more -->
# 学习内容
* 理解对象属性
* 理解并创建对象

# 一、理解对象
对象本质是一组散列表由key-value值组成。而value可以是数据或者函数。可以使用**对象字面量**方法创建对象。
```javascript
var person = {
    name: 'superman',
    age: 29,
    sayName: function(){
        alert(this.name);    
    }
}
```
## 1、属性类型
待补充。。

# 二、创建对象
创建对象有以下几种模式：
* 工厂模式
* 构造函数模式
* 原型模式
* 组合使用构造函数模式和原型模式
* 动态原型模式
* 寄生构造函数模式
* 稳妥构造函数模式
**要求能熟练掌握每种模式的写法，以及他们之间的区别。**
## 1、工厂模式
工厂模式是软件工厂领域一种广为人知的设计模式。特点是抽象了创建具体对象的过程。**工厂模式创建函数的例子**：
```javascript
//工厂模式
function createPerson(name,age){
    var obj = new Object(); //需要显式创建对象，并在最后返回该对象
    obj.name = name;
    obj.age = age;
    obj.sayName = function(){
            alert(this.name);
    }
    return obj; //返回该对象
}
var person = createPerson(); //实例化
```
工厂模式解决了创建多个相似对象的问题，却没有解决对象识别的问题(即怎么知道一个对象的类型)。因此出现了构造函数模式。
## 2、构造函数模式
就好比Object和Array这种原生的构造函数，可以使用构造函数来创建对象。因此也可以创建自定义的构造函数，从而定义自定义对象类型的属性和方法。**构造函数模式的例子**：
```javascript
//构造函数模式
function Person(name, age){
    this.name = name;
    this.age = age;
    this.sayName = function(){
        alert(this.name);
    }
}
//实例化
var person = new Person();
```
*构造函数与工厂模式的区别*
* 无需显式创建对象；
* 没有return语句 ；
* 直接将属性和方法赋给了this对象。

*使用 `new`调用构造函数会经历以下四个步骤*：
* 创建一个新对象；
* 将构造函数的作用域赋给新对象（因此this就指向了这个新对象）
* 执行构造函数中的代码（为这个新对象添加属性）
* 返回新对象

*构造函数存在的问题*
使用构造函数时，每个方法都要在每个实例上重新创建一遍。这样会导致创建了多个完成相同任务的function。没有这个必要。因此有了原型模式来解决构造函数所产生的这些问题。
## 3、原型模式
我们创建的每个函数都有一个 `prototype(原型)`属性，这个属性是*一个指针，指向一个对象*，而这个对象的用途是包含可以由特定类型的*所有实例共享的属性和方法*。因此为了解决构造函数模式存在的问题，我们可以将通用的方法添加到prototype中，如下面的例子：
```javascript
//原型模式
function Person(){
}
Person.prototype = {
     name : 'superman',
     age : 29,
     sayName : function(){
        alert(this.name);
    }     
}
var person1 = new Person();
var person2 = new Person();
console.log(person1.sayName == person2.sayName); // true
```
通过原型模式创建的对象，由于`sayName()`函数是加到`Person.prototype`上，所有的Person对象实例的`sayName()`都是共享的，因此都属于同一个实例。
*原型对象存在的问题*：
1、它省略了为构造函数传递初始化参数这一环节，结果所有实例在默认情况下都取得相同的值。
2、最大的问题是由于其共享的本性所导致的。由于原型中的属性都是被实例共享的，在属性为基本值的时候倒不会产生太大的问题。但如果属性是引用类型的时候，问题就凸显了。以下面的例子说明：
```javascript
funtion Person(){
}
Person.prototype = {
    constructor: Person,
    name: 'superman',
    friends: ['superman', 'spiderman'],
    sayName : function(){
        alert(this.name);
    }  
};
var person1 = new Person();
var person2 = new Person();
person1.friends.push('Van'); //由于friends方法被所有实例共享，因此任何一个实例操作这个方法，都会影响到所有的实例调用这个方法。
console.log(person1.friends);     //superman, spiderman, Van
console.log(person2.friends);     //superman, spiderman, Van
console.log(person1.friends === person2.friends);  // true;
```
这就是原型模式最大的问题。任何实例对原型上的方法进行操作都会影响的到所有实例对该方法的调用。
因此引入了下面的这种模式：**组合使用构造函数模式和原型模式**
## 4、组合使用构造函数模式和原型模式
这种模式是创建自定义类型的最常见方式。构造函数用于定义实例属性，原型模式用于定义方法和共享的属性。这样模式的优点便是，每个实例都会有自己的一份**实例属性的副本**，但同时又**共享着对方法的引用**，**最大限度地节省了内存**。同时这种模式还支持向构造函数传递参数，可谓急两种模式之长。如下面例子：
```javascript
funtion Person(name, age){
    this.name = name;
    this.age = age;
    this.friends = ["a", "b"];
}
Person.prototype = {
    constructor: person,
    sayName: function(){
        alert(this.name);
}
}
```
这种构造函数和原型混成的模式，是目前使用最广泛的一种创建自定义类型的方法。

## 5、动态原型模式
所谓动态原型模式就是**把所有的信息都封装在构造函数中，而通过在构造函数中初始化原型，又保持了同时使用构造函数和原型的优点。如下面例子：
```javascript
 function Person(name, age){
    //属性
    this.name = name;
    this.age = age;
    //方法
    if(typeof this.sayName !=  "function"){ //通过判断某个应该存在的方法是否有效，来决定是否需要初始化原型。
        Person.prototype.sayName = function(){
            alert(this.name);
        }
    }
}
var friend = new Person('Nicholas', 29);
friend.sayName();          //Nicholas
```
动态原型模式可谓非常完美！仅在方法不存在时，才会将它添加到原型中。
## 6、寄生构造函数模式
写法与工厂模式一模一样，区别是在创建实例的时候，是通过`new`来创建，与构造函数的实例对象方法一致。这种模式可以在特殊情况下用来为对象创建构造函数。如下面例子：我们可以创建一个具有额外特殊方法的特殊数组。
```javascript
functio SpecialArray(){
    //创建数组
    var values = new Array();
    // 添加值
    values.push.apply(values, arguments);
    //添加方法
    values,toPipedString = funtion(){
        return this.join("|");
    }
    return values;
}
var colors = new SpecialArray("red", "blue", "black");
alert(colors,toPipedString());    //"red|blue|green"
```
## 7、稳妥构造函数模式

