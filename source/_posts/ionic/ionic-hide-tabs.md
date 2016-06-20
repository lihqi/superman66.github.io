---
title: ionic隐藏底部菜单栏ion-tabs
date: 2016-06-16 21:48:32
tags:
- ionic
---
# 前言
ionic中没有提供属性来控制底部菜单栏ion-tabs在页面中的显示与隐藏，只有一个类`.tabs-item-hide`可以控制，所以我们可以通过控制添加`.tabs-item-hide`来实现隐藏菜单栏的目的。

# 实现方式
三个步骤就可以实现了
### 第一步，在ion-tabs指令上添加`hideTabs`
```html
<ion-tabs class="tabs-icon-top tabs-assertive {{hideTabs}}" >
</ion-tabs>
```
### 第二步：写`hideTabs`指令
下面的指令监听了ionicView广播的`$ionicView.beforeEnter`、`$ionicView.beforeLeave`和`$destroy`，当View在进入之前和离开之前，监听`hideTabs`属性，并添加`.tabs-item-hide`。
并在View被销毁的时候即页面返回的时候，将`hideTabs`置为false。指令代码如下：
```javascript
/**
   * 隐藏tabs指令
   * 
   */
  .directive('hideTabs', function ($rootScope) {
    return {
      restrict: 'A',
      link: function (scope, element, attributes) {
        scope.$on('$ionicView.beforeEnter', function () {
          scope.$watch(attributes.hideTabs, function (value) {
            $rootScope.hideTabs = 'tabs-item-hide';
          });
        });
        scope.$on('$ionicView.beforeLeave', function () {
          scope.$watch(attributes.hideTabs, function (value) {
            $rootScope.hideTabs = 'tabs-item-hide';
          });
          scope.$watch('$destroy', function () {
            $rootScope.hideTabs = false;
          })
        });
      }
    };
  })
```

### 第三步：在tab页面添加指令`hide-tabs`
然后在需要隐藏菜单栏的页面加上·hide-tabs·指令，如下面代码:
```html
<ion-view hide-tabs>
//页面内容
</ion-view>
```
这样就可以实现了隐藏底部菜单栏。



