---
title: AngularJS动态加载模块和依赖
date: 2016-02-24 22:23:54
tags: 
- AngularJS
- 动态加载
---
由于AngularJS是单页面应用框架，在正常的情况下，会在访问页面的时候将所有的CSS、JavaScript文件都加载进来。文件不多的时候，页面启动速度倒不会影响太多。但是一旦文件数太多或者加载的第三方库比较大的时候，就会影响页面启动速度。因此对于应用规模大、文件数比较多或者加载的第三方库比较大的时候，采用动态加载JS或者动态加载模块会极大提升页面的启动速度。本文将介绍如何利用ocLazyLoad实现动态加载。
<!-- more -->
AngularJS动态加载依赖第三方库：[ocLazyLoad](https://oclazyload.readme.io/docs)。ocLazyLoad是一个第三方库，支持AngularJS动态加载`module`、`service`、`directive`以及静态文件。
### 安装ocLazyLoad
可通过npm或者bower进行安装
```bash
npm install oclazyload
bower install oclazyload
```
### 将`ocLazyLoad` module 添加到你的应用中
```javascript
angular.module('myApp',['oc.lazyLoad']);
```
### 配置 `ocLazyLoad`
你可以在 `config`函数中配置 `$ocLazyLoadProvider`,配置文件如下
```bash
.config(['$ocLazyLoadProvider', function($ocLazyLoadProvider){
    $ocLazyLoadProvider.config({
        debug: true,
        events: true,
        modules: [
            {
                name: 'TestModule',
                files: ['test.js']
            }
        ]
    })
}])
```
* debug: 用来开启debug模式。布尔值，默认是false。当开启debug模式时，$ocLazyLoad会打印出所有的错误到console控制台上。
* events：当你动态加载了module的时候，$ocLazyLoad会广播相应的事件。布尔值，默认为false。
* modules：用于定义你需要动态加载的模块。定义每个模块的名字需要唯一。
modules必须要用**数组**的形式，其中files也必须以**数组**的形式存在，哪怕只需要加载一个文件

### 在路由当中加载module
```bash
    .config(['$routeProvider', function($routeProvider) {
        $routeProvider.otherwise('/index');
        $routeProvider.when('/index', {
            templateUrl: 'index.html',
            controller: 'IndexController',
            resolve: { //  resolve 里的属性如果返回的是 promise对象，那么resolve将会在view加载之前执行
                loadMyCtrl: ['$ocLazyLoad', function($ocLazyLoad) {
                    // 在这里可以延迟加载任何文件或者刚才预定义的modules
                    return $ocLazyLoad.load('TestModule'); //加载刚才定义的TestModule
                    /*return $ocLazyLoad.load([   // 如果要加载多个module，需要写成数组的形式
                        'TestModule',
                        'MainModule'
                        ]);*/
                }]
            }
        })
    }])
```
resolve设置的属性可以被注入到Controller当中。如果resolve返回的是promise对象的话，那么它们将在控制器加载以及$routeChangeSuccess被触发之前执行。
**$ocLazyLoad就是利用这个原理hack，进行动态加载**。
`resolve`的值可以是：
* key，the value of key 是会被注入到Controller的依赖的名字；
* factory，即可以是一个service的名字，也可以是一个返回值，它是会被注入到控制器中的函
数或可以被resolve的promise对象。

通过这样的配置，就可以实现了AngularJS动态加载模块和依赖。但是$ocLazyLoad提供的功能更加丰富，不止动态加载模块和依赖，还能动态加载service，diretive等。更多的功能，可以访问[$ocLazyLoad官网](https://oclazyload.readme.io)
