---
title: 多行文本溢出显示省略号
date: 2016-07-10 22:22:56
tags:
- CSS
---
# 前言
这两天在做移动端页面的时候发现，需要文本显示两行，且超出部分要显示省略号。如下图这种效果：
![](/images/webkit-line-clamp.png)

# 解决办法
利用`-webkit-line-clamp`、`-webkit-box-orient`和`-display:-webkit-box`三个属性就可以实现多行文本溢出显示省略号。代码如下：
```
.line-clamp{
    display: -webkit-box;
    overflow: hidden;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
}
```
由于移动端的浏览器内核都是Webkit，所以可以放心大胆的使用。
-webkit-line-clamp属性是一个不规范的属性（WebKit-Specific Unsupported Property），并不是CSS的标准，只能是Webkit内部使用。
它是用来限制一个块元素显示的文本的行数。为了实现效果，通常还需要结合
* ` -webkit-box-orient`：设置伸缩盒对象的子元素的排列方式
* `display:-webkit-box`：将对象作为弹性伸缩盒子模型显示
