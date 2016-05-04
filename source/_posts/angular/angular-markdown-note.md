---
title: 基于localStorage和AngularJS的markdown简易笔记本
date: 2016-05-04 22:36:21
tags:
- AngularJS
- markdown
---
这是基于AngularJS实现的简易markdown笔记本。具有添加、删除分类，添加、删除笔记本，添加标签，全屏显示等功能。后续将完善更多功能。
<!-- more -->
在之前markdown编辑器的基础上，再演化成一个基于localStorage的markdown简易笔记本。
上一篇基于AngularJS打造的markdown笔记本[点击这里查看](http://superman66.github.io/2016/04/15/angular/angular-markdown/)
先来看效果图：
![angular](/images/angular-markdown-note.gif)
# 已完成功能列表
* 添加和删除分类
* 添加和删除笔记本
* 笔记本编辑与保存
* 支持全屏编辑
* 笔记本可以添加多个标签、删除标签
# 功能待完善列表
* 支持分类和笔记本的右键菜单（菜单包括：重命名、删除、添加）
* 支持多级分类
*  支持移动端布局

# 知识点记录
## CSS布局
##### 1、左边两列固定宽度，右边一列宽度自适应。如下图所示
![angular](/images/angluar-markdown-note-1.png)
代码如下：
```html    
    <style>
        *{
		padding: 0;
		margin: 0;
		box-sizing: border-box;
	}
	.sub{
		float: left;
		width: 200px;
		background: #eee;
		border: 1px solid #ddd;
		height: 200px;
	}
	.extra{
		float: left;
		width: 300px;
		background: #eee;
		border: 1px solid #ddd;	
		height: 200px;
	}
	.main{
		width: 100%;
		height: 200px;
		padding-left: 500px;
		background: #eee;
		border: 1px solid #ddd;	
	}
    </style>

		<div class="sub">sub</div>
		<div class="extra">extra</div>
		<div class="main">main</div>
```
左边两列通过float属性使其固定在左边。最右边一列设置padding值为左边两列的宽度和，同时设置宽度为100%。这样就可以实现了左边两列固定宽度，右边一列宽度自适应。
##### 2、左右两列固定，中间一列自适应，如下图所示
![angular](/images/angluar-markdown-note-2.png)
代码如下：
```html    
    <style>
	*{
		padding: 0;
		margin: 0;
		box-sizing: border-box;
	}
	.sub{
		float: left;
		width: 200px;
		background: #eee;
		border: 1px solid #ddd;
		height: 200px;
	}
	.extra{
		float: right;
		width: 300px;
		background: #444;
		border: 1px solid #ddd;	
		height: 200px;
	}
	.main{
		width: 100%;
		height: 200px;
		background: #888;
		border: 1px solid #ddd;	
	}
    </style>

		<div class="sub">sub</div>
		<div class="extra">extra</div>
		<div class="main">main</div>
```
第二种布局html部分并没有改变，只对CSS进行了改变。去掉`.main`的padding-left属性值，`.extra`的float属性值由left改为right。
##### 3、左边一列自适应，右边两列固定宽度。如下图所示
![angular](/images/angluar-markdown-note-3.png)
代码如下：
```html    
    <style>
        *{
		padding: 0;
		margin: 0;
		box-sizing: border-box;
	}
	.sub{
		float: right;
		width: 200px;
		background: #eee;
		border: 1px solid #ddd;
		height: 200px;
	}
	.extra{
		float: right;
		width: 300px;
		background: #eee;
		border: 1px solid #ddd;	
		height: 200px;
	}
	.main{
		width: 100%;
		height: 200px;
		padding-right: 500px;
		background: #eee;
		border: 1px solid #ddd;	
	}
    </style>

		<div class="sub">sub</div>
		<div class="extra">extra</div>
		<div class="main">main</div>
```
这种布局与第一种布局类似，HTML结构依然保持不变，只不过是`.sub`和`,extra`的float属性改为right，`.main`的padding值改为`padding-right:500px`
## 基于localStorage存储数据
由于没有接入数据库，因此采用的是localStorage本地存储。localStorage是基于key-value的JSON格式进行存储，类似于NoSql数据库。为了模拟后台的增删改查API，写了一个data-Service来封装增删改查的方法。
在写Service的过程中，思维一直受到之前关系型数据库的影响，总是想采用关系型数据库那套增删改查的方法来实现。对比一下，有几个差异点：
* 自增长id
    之前在使用MySQL数据库的时候，每条记录会有一个自增长id的，而且这个自增长id是不可逆的，删除了这条记录后，这个id也不会被回收。后续再添加的记录的id是在之前id的基础上递增的。而在localStorage中，则没有自增长id的机制。因此必须模拟实现自增长id。实现机制：每次要添加数据的时候，先去遍历该列表，获取该列表的长度length。如果length不等于的话，则id=length+1。如果length=0，id=1；
* 读取数据
    基于JSON的存储，在存储列表数据时一般采用数组，而数组里面再包含一个个对象来存储数据。如果需要根据id来获取相对应对象所在的值，就需要通过先通过查找该id在整个数组中的索引值，再通过这个索引值去得到该对象。
```javascript
  {
    "id": 4,    //标识该条记录的唯一id
    "name": "4",
    "noteList": [
      {
        "id": 1,
        "title": "4-1",
        "createTime": "2016-04-26 22:52:59",
        "updateTime": "2016-04-26 22:52:59"
      },
      {
        "id": 2,
        "title": "4-2",
        "createTime": "2016-04-26 22:54:09",
        "updateTime": "2016-04-26 22:54:09"
      }
    ]
  }
```
* 删除数据
    删除数据则是采用了数组的方法splice。因为使用splice()删除数据，数组会根据需要减小它们的索引值，并自动减少数组的长度，可以避免产生稀疏数组。
    
## ECMAScript5的数组方法
在操作localStorage数据的过程中，使用到了ECMAScript5的一些方法，比如forEach()，filter()等。之前也是没有接触过。具体关于JavaScript数组将会在另外一篇文章中详细说明。
* forEach()
* map()
* filter()
* every()和some()
* reduce()和reduceRight()
* indexOf()和lastIndexOf()

项目地址：http://superman66.github.io/angular-markdown/app/index.html#/index
