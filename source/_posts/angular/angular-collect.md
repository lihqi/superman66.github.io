---
title: 关于AngularJS的一些坑
date: 2016-04-04 16:45:39
tags: AngularJS
---
本文是搜集学习AngularJS和开发中收集的一些坑。持续更新。
<!-- more -->
## 1、select第一行出现空白
在使用带有angular数据绑定功能的HTML SELECT 元素时，如果不指定default value的话，第一个option会出现空白，如下图：

解决办法：
`为select添加 一个 默认的option`
```javascript
    <select ng-model="myColor" ng-options="d.id for d in data">
            <option value="">请选择</option>
     </select>
```
## 2、AngularJS在IE的XHR请求存在Bug。
IE只会在第一次才会从服务器中去请求XHR数据，之后的XHR请求都是从缓存中取。 
解决办法：禁用IE对ajax的缓存
http://stackoverflow.com/questions/16098430/angular-ie-caching-issue-for-http
具体代码如下：
```javascript
myModule.config(['$httpProvider', function($httpProvider) {
    //initialize get if not there
    if (!$httpProvider.defaults.headers.get) {
        $httpProvider.defaults.headers.get = {};  
    }  
 
    // Answer edited to include suggestions from comments
    // because previous version of code introduced browser-related errors
 
    //disable IE ajax request caching
    $httpProvider.defaults.headers.get['If-Modified-Since'] = 'Mon, 26 Jul 1997 05:00:00 GMT';
    // extra
    $httpProvider.defaults.headers.get['Cache-Control'] = 'no-cache';
    $httpProvider.defaults.headers.get['Pragma'] = 'no-cache';}]);
```
## 3、使用ui-bootstrap中的$modal出现编译错误
问题描述:在页面一加载的时候就去调用`$modal`服务的话，会报错对象的模板页面找不到。
解决办法：使用`$timeout`，延迟100ms之后再去调用`$modal`服务。
```javascript
 $timeout(function () {
            if($scope.isDated){
                $modal.open({
                    backdrop: 'static',
                    keyboard: false,
                    animation: true,
                    templateUrl: 'activity_reward_dated_modal.html',
                    controller: 'ActivityRewardDatedModalCtrl'
                });
            }
        },100)
```
## 4、页面出现表达式闪烁

### 1、为什么会出现表达式{{express}}闪烁？
　　因为我们利用JavaScript操作DOM，都需要等待DOM Ready完成。AngularJS也会在DOM Ready完成后才会去解析页面上的视图模板。在AngularJS开始工作之前，用户会看到表达式本身，之后才会被替换为表达式的求值结果。
### 2、表达式闪烁解决方案：主要有两种解决方案
* 把{{express}}替换为ng-bind因为ng-bind只是html节点的拓展属性，浏览器并不理解这个未知的属性，因为浏览器不会显示它。
```
<div ng-bind='express'></div>
```
* 利用AngularJS提供的ngCloak来标记AngularJS出现的节点。ngCloak主要利用CSSdisplay属性来控制显示和隐藏表达式模块。
```
<div ng-cloak>{{express}}</div>
```
#### ngCloak实现原理
　　AngularJS会在文件加载的同时向HTML的head元素添加ng-hide、ng-cloak的样式定义。这样在浏览器初始化页面的时候，添加了ngCloak指令的节点会被`ng-cloak`这个样式隐藏掉，因此在AngularJS解析视图模板之前，我们看不到ng-cloak指令的节点。
那么被隐藏的节点又是怎么样在视图解析完成后显示出来的呢？
　　complie函数会在AngularJS开始解析模板指令的时候被执行，它会移除在DOM节点上的ngCloak属性和ngCloak样式，这样带有ng-cloak指令的的DOM节点就会被正常显示出来。

## 5、使用第三方插件的时候，会出现无法动态绑定值
　　AngularJS是通过“脏检查”来实现动态绑定的。然而这种"脏检查"机制只能适用于AngularJS内部的行为触发方式比如ng-clcik、ng-change等，而不能涵盖所有的AngularJS操作场景。典型的例子就是我们在封装第三方jQuery插件时，不能自动更新视图，而需要我们手动调用`$scope.$apply`。但是往往我们在集成jQuery插件时候调用`$scope.$apply`会出现`digest in progress`错误，那么这时候可以考虑使用`$timeout`来代替.
　　那为什么手动触发`$scope.$apply`会报`digest in progess`的错误？
　　AngularJS在任何时候只允许一个`$digest`或者`$apply`操作存在于应用中。因此当应用中已经有`$digest`或者`$apply`操作的时候，如果再手动去触发的话，就会报错`$digest in progress`。
   

参考资料：
> https://docs.angularjs.org/error/$rootScope/inprog
>《AngularJS深度剖析与最佳实践》


