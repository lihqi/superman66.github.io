---
title: 关于AngularJS的一些坑
date: 2016-02-29 21:45:39
tags: AngularJS
---
本文是搜集学习AngularJS和开发中收集的一些坑。
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
