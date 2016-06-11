---
title: JavaScript的数组详解
date: 2016-05-08 00:12:54
tags:
- Javascript
- Javascript学习笔记
---
# 前言
在一篇文章中看到学习技术一般两种方法论，一是项目驱动，狼吞虎咽般吸收知识，不过多关注细节；另外一种是求知驱动，反刍过程，在这个过程中，不应该放过任何你有疑问的知识。深究下去，搞明白具体是怎么回事，然后总结，分享。
本着这个行为准则，上个礼拜在写代码中遇到了两个关于数组方法的问题，一时没有想起来，于是重新翻开犀牛书，重新复习、查缺补漏。
# 学习前的提问
1. 如何创建数组
2. 数组的读和写
3. 什么是稀疏数组、稠密数组
4. 如何添加和删除数组元素
5. 如何遍历数组
6. ECMAScript3中的数组方法
7. ECMAScript5中的数组方法

## 创建数组
##### 数组字面量
通过数组字面量创建数组，只需要在方括号中将数组元素用逗号隔开即可。
```javascript
var empty = [];
var array1 = [1,2,3,4,5];
var array2 =[,,];    //数组有2个元素，都是undefined，逗号数等于数组个数
```
##### 调用Array()构造函数
调用构造函数可传入参数或者不传入参数
* 不传入参数
```javascript
var arr1 = new Array();
```
* 传入一个数值，指定数组长度
```javascript
var arr2 = new Array(5);
```
* 显示指定两个或者多个数组元素或者数组的一个非数值元素
```javascript
var arr2 = new Array(1,2,2,3,'test');
```
以这种方式，构造函数的参数将会成为数组的元素。使用数组字面量创建函数会比这种方式来得简单。
## 数组元素的读和写
数组可以通过**数组下标索引**来获取和设置数组元素的值。
```javascript
;
(function() {
    var underf = ['name', 2, 4];
    console.log(underf[0]);    //name
    console.log(underf[1]);    //2
    console.log(underf[2]);    //4
    console.log(underf[3]);    //undefined
})();
```
其中当我们试图去访问一个不存在的下标的时候，数组并不会报错，而只是返回undefined。出现这种情况的原因是什么呢？让我们再来看个例子：
```javascript
;
(function() {
    var underf = ['name', 2, 4];
    underf[-1.23] = 'name';
    console.log(underf[-1.23]);    //name
    console.log(underf.length);    //3    
})();
```
当我们使用了不是非负整数的下标时，数组会将其当做一个常规的属性，而不是一个数组的元素。这是因为数组本身就是特殊的对象。下标索引本质也是一种对象的属性名。只不过当下标索引处于0~2的32次方-2的非负整数时，它才是数组的索引，通过下标索引设置数组的值的时候，数组才会自动更新其长度length的值。其余数字的都会被当做属性来对待。
上面的这个例子中，
`underf[-1.23] = 'name';`由于不是索引，所以将会创建一个名为"-1.23"值为"name"的属性。而此时数组的长度是不会发生变化的，因此
`underf.lenght`的值依然为3
之所以会出现这种情况，是因为数组本身就是对象的特殊形式。对象可以通过方括号的方式来访问属性，数组也可以。事实上数组的索引仅仅是对象属性名的一种特殊类型，这意味着JavaScript数组没有其他语言如Java中“越界”的概念。当试图查询任何对象中不存在的属性时，不会报错，只会得到undefined的值。

## 稀疏数组
**稀疏数组就是包含从0开始的不连续的索引的数组**。通常数组的length属性值代表数组中元素的个数。如果数组是稀疏的，那么length的属性值一个大于数组中元素的个数。
可以通过Array(）构造函数和delete操作符创建或者生成稀疏数组。如下面的例子：
```javascript
    var arr1 = new Array(100);    //创建一个没有元素的数组，但是length为100
    var arr2 = [];    //创建一个空的数组，length = 0
    arr2[100] = 10;    //添加一个元素，但是设置length = 101
    console.log(arr1.length);    //100
    console.log(arr2.length);    //101
```
相反的稠密数组就是指从0开始索引是连续的数组。
## 如何添加和删除数组元素
##### 添加元素
前面已经提到最简单的添加数组元素的方法：为新索引赋值。
还有其他很多更高效的方法，将在下面详细介绍。
##### 删除元素
1、使用delete操作符删除数组元素
```javascript
(function() {
    var underf = [1, 2, 4];
    delete underf[0];
    console.log(underf);    //[ , 2, 4 ]
    console.log(underf.length);    //3
})();
```
使用delete操作符将会产生稀疏数组。也就是原有的下标不变，数组的长度也不变。
2、为属性赋值undefined
```javascript
;
(function() {
    var underf = [1, 2, 4];
    underf[0] = undefined;
    console.log(underf);    //[ undefined, 2, 4 ]
    console.log(underf.length);    //3
})();
```
其效果与delete相似，原来的下标依然不变，数组的长度也不会因此改变。
**推荐使用赋值undefined的做法**，因为这种做法比delete高效。

## 如何遍历数组
1、for循环。最常用的遍历方法
```javascript
    var underf = [1, 2, 4,5,6];
    for (var i = 0; i < underf.length; i++) {
        console.log(underf[i]);
    }
```
对于上述的遍历方式有个需要优化的地方，每次循环都要取数组的长度，这样不合理。因此改进了下，将数组长度保存在变量中
```javascript
    var underf = [1, 2, 4,5,6];
    for (var i = 0， len = underf.length; i < len; i++) {
        console.log(underf[i]);
    }
```
上面的方法是基于数组是稠密数组以及所有数据都是合法数据。如果不是的话，则应该先检测、排除不合法数据。
如果想要排除null、undefined和不存在的元素
```javascript
    var underf = [1, 2, 4,5,6];
    for (var i = 0， len = underf.length; i < len; i++) {
        if(!underf[i]) continue;
        console.log(underf[i]);
    }
```
如果想要排除undefined和不存在的元素
```javascript
    var underf = [1, 2, 4,5,6];
    for (var i = 0， len = underf.length; i < len; i++) {
        if(a[i] === undefined) continue;
        console.log(underf[i]);
    }
```
如果只想要排除不存在的元素，可以通过`in`操作符来判断对应的索引是否有元素
```javascript
    var underf = [1, 2, 4,5,6];
    for (var i = 0， len = underf.length; i < len; i++) {
        if(a[i] in underf) continue;
        console.log(underf[i]);
    }
```
2、使用ECMAScript5中提供的`forEach()`方法，下面将详细说明

## ECMAScript3定义的数组的方法
### join()
Array.join()是数组中的所有元素转化为字符串并连接在一起，**最后返回生成的字符串**。
```
(function() {
    var underf = [1, 2, 4,5,6]; 
   console.log(underf.join(" "));	//1 2 4 5 6
    console.log(underf.join("-"));	//1-2-4-5-6
})();
```
### reverse()
Array.reverse()是将数组的元素颠倒顺序，返回逆序的数组。
```javascript
var arr = [1,2,3];
arr.reverse(arr);
console.log(arr);    //[3,2,1]
```
### sort()
Array.sort()是将数组中的元素排序并返回排序后的数组。当不带参数调用sort()时，数组以字母表顺序排序。
```javascript
var underf = [1, 6, 1,2,3];
    console.log(underf.sort());    //[ 1, 1, 2, 3, 6 ]
})();
```
ps：如果数组包含undefined元素，它们将会被排到数组的尾部。
如果要自定义排序，需要传入一个比较函数。比较函数的规则是：如果第一个参数应该在前，比较函数应该返回一个小于0的数值。反之，则返回大于0的数值。如果两个值相等，则返回0；
```
var underf = [111, 226, 11,22,33]; 
    console.log(underf.sort());	//[ 11, 111, 22, 226, 33 ]
    console.log(underf.sort(function(a,b){
    	return a-b
    }))		//[ 11, 22, 33, 111, 226 ]
```

### concat()
Array.concat()创建并返回一个新的数组，这个数组连接了原来的数组和参数。下面是示例：
```javascript
        var arr1 = [111, 226, 11,22,33]; 
    	var arr2 = arr1.concat('a');
    	console.log(arr2);	//[ 111, 226, 11, 22, 33, 'a' ]
```
### slice()
Array.slice()方法是返回指定数组中的一个片段或子数组。它有两个参数，分别指定了所要截取的片段的开始位置和结束位置（注意这里说的位置都是指下标）。返回的数组包含**第一个参数指定的位置和所有到但不含第二个参数指定的位置之间的所有数组元素**，也就是高数中所说的左闭右开的区间[start, end)。slice()存在以下三种情况
**1、只有一个参数**
返回的数组包括从开始位置到数组结尾的所有元素
```javascript
    var arr1 = [1, 2, 3, 4, 5, 6, 7];
    var arr2 = arr1.slice(2);
    console.log(arr2); //[ 3, 4, 5, 6, 7 ]
```
**2、两个参数都是整数**
正常返回开始位置到结束位置所有的数组元素。如果第二个参数的位置超出数组长度，则效果等同于**只有一个参数**的情况，返回的是从开始位置到数组结尾的所有元素。
```javascript
    var arr1 = [1, 2, 3, 4, 5, 6, 7];
    var arr2 = arr1.slice(2, 9);
    var arr3 = arr1.slice(0,2);    //[1, 2]
    console.log(arr2); //[ 3, 4, 5, 6, 7 ]
```
**3、参数带有负数**
如果参数出现负数了，它表示**相对于数组中最后一个元素的位置**。比如-1指定了最后一个元素，-2指定了倒数第二个元素，-3指定了倒数第三个元素。
```javascript
    var arr1 = [1, 2, 3, 4, 5, 6, 7];
    var arr2 = arr1.slice(0,-4);
    console.log(arr2); //[ 1, 2, 3 ]

    var arr3 = arr1.slice(0,-1);
    console.log(arr3); //[ 1, 2, 3, 4, 5, 6 ]

    var arr4 = arr1.slice(-3,-1);
    console.log(arr4); //[ 5, 6]
```

### splice()
Array.splice()是在数组中插入或删除的通用方法，这个方法可能是最强大的数组方法。它是会修改原来的数组，在插入或者删除点之后的数组元素会根据需要增加或减小它们的索引值，因此数组的其他部分仍然保持连续的。splice()返回的是包含删除元素的数组，如果没有删除任何元素，则返回一个空数组。
spilce(index, num, aruments)，它有三个参数。分别是删除或插入的起始位置，删除的个数，要插入的数组元素。这么说似乎体现不出它的强大。我们可以从它的三个功能来学习它：
##### 删除
可以删除任意数量的项，只需要提供2个参数：要删除的第一项的位置和要删除的项数。
例如：splice(0, 2)就会删除数组前两项。
##### 插入
可以向指定位置插入任意数量的项，只需提供3个参数：起始位置，0（要删除的项为0）和要插入的项。如果要插入多项，可以再传入第4个、第5个参数，以至任意多项。
例如：splice(1, 0, 1,1,1,1)，就会往数组的第一个位置开始，插入4个1
##### 替换
可以向指定位置插入任意数量的项，且同时删除任意数量的项，实现替换的功能。只需要传入3个参数，起始位置，要删除的项数和要插入的任意数量的项。

### push()和pop()
JavaScript提供了push()和pop()两个方法，让数组拥有类似栈的功能（先入先出）。
push()方法是在数组尾部添加一个或者多个元素，**并返回数组的新长度**。
pop()方法则相反，它删除数组的最后一个元素，**减小数组的长度并返回删除的值**。
这两个方法都修改原来的数组。组合使用push()和pop()方法，就能够用JavaScript数组实现先进先出的栈。

### shift()和unshift()
这两个方法与push()和pop()类似，只不过它们是在数组的头部进行添加和删除操作。
unshift()在数组的头部添加一个或多个元素，并将已存在的元素移动到更高索引的位置来获得足够的空间，**最后返回数组新长度**。
shift()删除数组第一个元素并返回，然后把所有随后的元素下移一个位置，来填补数组头部的空缺。
需要注意的是，当unshift()拥有多个参数时，参数时一次性插入到数组中，而非一个个插入的。所以插入的元素在元素中的顺序和它们在参数列表中的顺序是一样的。
## toString()和toLocalString()
这两个都是将数组的每个元素转换为字符串输出，保留逗号。和不加参数的join()一样。
区别的是toLocalString()是toString()的本地化版本。
## ECMAScript5中的数组方法 
ECMAScript5定义了9个新的数组方法来遍历、映射、过滤、检测、简化和搜索数据。
这9个方法的第一个参数接收一个函数，并且对数组的每个元素调用一次该函数。在大多数情况下这个函数有三个参数：数组元素(value), 索引(index), 数组本身(array)。这些方法本身不会修改原始数组，但传递给这些方法的函数时可以修改原始数组。
##### forEach()
forEach()方法从头到尾遍历数组，为每个函数调用指定的函数。该方法没有返回值。
```javascript
    var arr1 = [1, 2, 3, 4, 5, 6, 7];
    var sum = 0;
    arr1.forEach(function(value) { //遍历数组将每个元素的值累加到sum
        sum += value;
    });
    console.log(sum); //28

    arr1.forEach(function(value, index, arry){	//对数组的每个元素累加
    	arry[index] = value+2;
    })
    console.log(arr1);	//[ 3, 4, 5, 6, 7, 8, 9 ]
```
##### map()
map()方法将调用的数组的每个元素传递给指定的函数，**并返回每次函数调用的结果组成的数组**，并不会修改原始数组。如果原始数组是稀疏数组，那么map()方法返回的也是稀疏数组，返回的数组具有相同的长度，相同的缺失元素。
```javascript
    var arr2 = [1, 2, 3, 4, 5, 6];
    var arr3 = arr2.map(function(value) {
        return value + value;
    })
    console.log(arr3);    //[ 2, 4, 6, 8, 10, 12 ]
```

##### filter()
这个方法顾名思义，就是起到类似过滤器的作用。它用来对数组的每一项元素运行给定的函数（这个函数是用来执行逻辑判断的，返回true或者false），然后返回符合该函数判断逻辑的元素所构成的数组。
```javascript
    var arr4 = [1,2,3,4,5,6,7];
    var arr4New = arr4.filter(function(value){
    	return value > 4;
    })
    console.log(arr4New);	//[ 5, 6, 7 ]    //返回数组中所有大于4的元素
```
##### every()和some()
这两个方法是数组的逻辑判定：它们对数组元素应用指定的函数进行判定，返回true或者false。它们就像数学中的所有量词和存在量词。every()指数组所有的元素都满足判定函数才返回true。而some()指当数组中至少存在一个元素满足判定函数时返回true，而且当且仅当数组中的所有元素都不满足判定函数时才返回false；

##### reduce()和reduceRight()
这两个缩小数组的方法都会遍历数组的所有项，然后构建一个最终返回的值。区别的是，reduce()方法**从数组的第一个项开始，逐个遍历到最后**，而reduceRight()**从数组的最后一个项开始，向前遍历到第一项**。
reduce()和reduceRight()都需要两个参数。第一个参数是 每个项执行化简操作的函数。第二个参数为可选项，是传递一个给函数的初始值。**对于非空数组，如果第二个参数没有指定，那么将使用数组的第一个元素作为初始值**
比如想要对一个数组的元素求和与求积：
```javascript
var arry = [1,2,3,4,5];
//对数组的元素进行求和，第二个参数初始值可以为0
var sum = arrr.reduce(function(x, y){
	return x+y;
},0)

//对数组元素进行求积，所以第二个参数要为1
var sum1 = arry.reduce(function(x, y){
	return x*y;
}, 1)
```
其中第一个参数也就是每个项都调用的函数接受4个参数：
* 第一个参数：到目前为止的化简操作累积的结果。当第一次调用函数时，第一个函数是**初始值**
* 第二个参数：当前值
* 第三个参数：项的索引
* 第四个参数：数组对象
##### indexOf()和lastIndexOf()
这两个方法是搜索给定的元素在数组中的位置，如果找到则返回该元素在数组中的位置，如果找不到则返回-1。如果存在多个相同值的元素，则返回第一次找到的元素的位置。
这两个方法都接受两个参数：
* 要查找的项
* 查找起点位置的索引（可选）
两个方法的区别在于：indexOf()是从数组的开头向后查找，而lastIndexOf()则是从数组的末尾向前开始查找。
```javascript
var arry = [1,2,3,4,5,1];
console.log(arry.indexOf(1));		//a[0]=1
console.log(arry.lastIndexOf(1));	//a[5]=1
console.log(arry.indexOf(8));		//-1 没有该元素
```
那么问题来了，如果我们想要知道查找所有指定值在数组中的位置而不仅仅是第一个元素的位置，可以怎么做呢？
我们可以利用indexOf()的第二个参数（查找起点位置的索引）来完成。
```javascript
function findAll(arry, x){
	var results = [],
	length = arry.length,
	pos=0;
	while(pos<length){
		pos = arry.indexOf(x, pos);
		if(pos == -1){
			break;
		}
		results.push(pos);
		pos++;
	}
	return results;
}
var arry = [1,2,3,1,1,4,5,1];
console.log(findAll(arry,1));	//[ 0, 3, 4, 7 ]
```
我们通过遍历数组，每个将找到元素的位置存入到数组中，下一次查找的起点位置就从上次的元素的位置开始。这样就记录下所有符号条件的元素的位置了。

## 总结
JavaScript中除了Object之外，Array引用类型恐怕是最常用的类型了。在没有深入学习之前，总是觉得数组不就那么一回事，so easy，根本不值得我花时间去仔细认真地学习。然而事实并不是这样的。这篇文章结束之后才认识到，之前对数组的认识是有多皮毛，很多关于数组各种方法的使用，注意点我一点都不知道。幸亏，我花时间学了下来了。以后要引以为戒，学习是个循序渐进的过程，是一步一个脚印的过程。戒骄戒躁，静心学习。