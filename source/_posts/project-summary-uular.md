---
title: 优乐健身移动端页面制作总结
date: 2016-03-12 21:50:42
tags: 
- 移动端 
- rem布局
---
该项目是优乐健身的移动端页面。刚好利用这个项目用来学习移动端的适配方案：rem布局。 
<!-- more -->

# 项目介绍
该项目是优乐健身的移动端页面。网站要求字体和边距做到自适应。因此采用了rem布局。
rem布局参考资料：
web app变革之rem: https://isux.tencent.com/web-app-rem.html
手机端页面自适应解决方案: http://www.jianshu.com/p/b00cd3506782

# 项目技术点
## 1、rem布局
rem布局非常简单，只需要将下面的JavaScript代码放到你的页面中即可。
```
(function (doc, win) { 
	var docEl = doc.documentElement, resizeEvt = 'orientationchange' in window ? 'orientationchange' : 'resize', 
	recalc = function () { 
	var clientWidth = docEl.clientWidth; 
	if (!clientWidth) return; 
	if(clientWidth>=640){ 
	docEl.style.fontSize = '100px'; 
	}
	else{ 
	docEl.style.fontSize = 100 * (clientWidth / 640) + 'px'; 
	}
	}; 
	if (!doc.addEventListener) return; 
	win.addEventListener(resizeEvt, recalc, false); 
	doc.addEventListener('DOMContentLoaded', recalc, false); 
	})(document, window);
```
### 如何使用
先解释下上面的代码，最核心的意思就是说当页面的宽度超过了640px，html的`font-size`为100px。如果小于640px，则通过`100 * (clientWidth / 640)`计算出页面的`font-size`。这时候便可以在网页中尽情使用rem了。所以涉及到宽度和距离的属性比如`width`、`margin`、`padding`、`font-size`等都可以使用rem作为单位。
那么设计稿中的px值要怎么转换成rem值呢？
我们可以将font-size的值设成100px，这样1rem = 100px。如果设计稿中的宽度是30px的话，那么就可以很方便的转换为0.3rem。
**ps:这里设置宽度为640px是根据设计稿来的，如果你拿到的设计稿是750，那么你就可以将上面的数值改为750。**

### rem布局存在的问题
**存在的问题**：在生成dom树的时候，底部用js改变html的font-size的话，会造成整个页面重新布局，这样的结果就是导致页面元素因为尺寸改变了而闪烁的情况。在页面类容大的时候，加载时页面会有明显的动态变化，
**解决办法**： 
* 使用一个全局loading页面，在fontSize计算之后才展示真正的页面
* 用响应式样式控制最好先用响应式，再用js。具体做法就是用CSS的`@media`根据屏幕初始化一遍html的font-size，然后在再用js计算，这样可以避免页面加载时候出现的闪烁。具体的CSS代码如下面：
```css
@media only screen and (max-width: 320px){html{font-size: 9px;} }
@media only screen and (min-width: 320px) and (max-width: 352px){html{font-size: 10px;} }
@media only screen and (min-width: 352px) and (max-width: 384px){html{font-size: 11px;} }
@media only screen and (min-width: 384px) and (max-width: 416px){html{font-size: 12px;} }
@media only screen and (min-width: 416px) and (max-width: 448px){html{font-size: 13px;} }
@media only screen and (min-width: 448px) and (max-width: 480px){html{font-size: 14px;} }
@media only screen and (min-width: 480px) and (max-width: 512px){html{font-size: 15px;} }
@media only screen and (min-width: 512px) and (max-width: 544px){html{font-size: 16px;} }
@media only screen and (min-width: 544px) and (max-width: 576px){html{font-size: 17px;} }
@media only screen and (min-width: 576px) and (max-width: 608px){html{font-size: 18px;} }
@media only screen and (min-width: 608px) and (max-width: 640px){html{font-size: 19px;} }
@media only screen and (min-width: 640px){html{font-size: 20px;} }

来源： https://isux.tencent.com/web-app-rem.html
```     
* 采取淘宝的[flexible.js](https://github.com/amfe/lib-flexible)方案解决。
```
## 2、自定义select框
如何自定义select框的样式
## 3、移动端直接拨打电话

```html
<a href="tel://110">电话</a>
```
在`a`标签中使用tel://协议，后面跟着电话号码，就可以实现在移动端拨打电话。