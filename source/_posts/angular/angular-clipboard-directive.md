---
title: 复制内容到剪贴板——基于clipboard的AngularJS指令
date: 2016-06-11 11:10:09
tags:
- clipboard.js
- AngularJS
---
# 前言
在其他网站上，经常会看到网址复制到剪贴板这种功能，刚好前些天公司在做分享活动时也遇到了这种需求，于是就自己做了一个基于clipboard.js的AngularJS的指令。
# 关于clipboard的解决方案
目前比较常用的有两种解决方案，一种是使用ZeroClipboard.js实现，另一种则是使用clipboard.js实现。
**clipboard.js**最大的优点就是
>A modern approach to copy text to clipboard.No Flash. No frameworks. Just 3kb gzipped

clipboard.js浏览器支持如下，仅支持IE9+：
![text](/images/angular-clipboard-browser.jpg)
更多详情的信息可前往clipboard.js官网（https://clipboardjs.com/）。
所以如果想要支持低版本的IE（IE8及以下）的话，肯定就无法使用clipboard.js，所以我们还可以选择ZeroClipboard.js。
>The ZeroClipboard library provides an easy way to copy text to the clipboard using an invisible Adobe Flash movie and a JavaScript interface.

ZeroClipboard.js是基于Flash实现复制内容到剪贴板，最大的优点就是支持较低版本的浏览器，包括低版本的IE。
但这同时也是它最大的弊端，如果用户的浏览器没有装Flash，那么它就只能歇菜了。

更多关于ZeroClipboard的信息可前往官网（http://zeroclipboard.org/）

# 如何使用基于clipboard.js实现的AngularJS指令
既然采用了AngularJS，那就已经勇敢得抛弃了老掉牙的IE8了。所以我自然就选择了clipboard.js来实现。
## 加载依赖
下载（https://github.com/superman66/angular-clipboard）代码到本地，将`angular-clipboard`添加到你的应用的依赖中
```javascript
angular.module('myApp', ['angular-clipboard']);
```
## 使用方法
```html
<!-- Target -->
    <textarea id="bar" ng-model='text'></textarea>
    <!-- Trigger -->
    <button class="btn"  clip-action="copy" clip-model="text" clip-alert='true' clip-success-text="地址已经复制到剪贴板!" clipboard>
        复制到剪贴板
    </button>
```
其中button按钮拥有4个属性分别为：
* clip-action：设置行为，默认是copy，还可以设置为cut；
* clip-model：需要复制的model，这里的model要等于前面的`ng-model='text'`;
* clip-alert：是否需要弹出框提醒，true是弹出框提醒，false则不弹出；
* clip-success-text：弹出框提醒文字。

到这里就可以愉快地使用复制到剪贴板的功能了。
最后来看下非常简单的源代码：
```javascript
angular.module('angular-clipboard', [])
.directive('clipboard', [function(){
    return{
        restrict: 'A',
        scope: {
            action: '@clipAction',
            text: '=clipModel',
            clipAlert: '@clipAlert',    //if alert the tip after clip success
            successText: '@clipSuccessText'    //the alert text
        },
        link: function(scope, ele, attr){
            angular.element(ele).attr('data.clipboard-action', scope.action);    //set action: copy or cut. default: copy
            angular.element(ele).attr("data-clipboard-text",scope.text);    //set text that you want to copy
            scope.$watch('text', function(){
                angular.element(ele).attr("data-clipboard-text",scope.text);
            })
            var clipboard = new Clipboard('.'+attr.class);
            //after clip success
            clipboard.on('success', function(e) {
                if(scope.clipAlert =='true'){
                    alert(scope.successText);
                }
                e.clearSelection();
                //clipboard.destroy();    //destory
            });
            //when error occured
            clipboard.on('error', function(e) {
                console.error('Action:', e.action);
                console.error('Trigger:', e.trigger);
            });
        }
    }
}])
```
# 总结
既然源码都这么简单，那我为什么还会选择将其封装成一个可复用的directive呢？因为本着Don't Repeat Yourself的准则，虽然功能非常简单，但是如果我不将其抽象成一个组件的话，下次再遇到同样的需求的时候，我还得再写一遍代码。而我把它封装成组件的话，下次需要再用的时候，我只需要在代码中引入即可。做到write once， use anywhere。

