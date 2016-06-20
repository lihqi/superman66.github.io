---
title: ionic登录拦截机制-使用Modal作为登录框
date: 2016-06-20 22:12:27
tags:
- ionic
---
#  前言
先来说说网页和App中交互的不同。在网页中，页面与页面之间的跳转过程较少会采用转场动画，直接就是页面切换。
而在App的交互方式中，页面切换是有转场动画的。页面进入一般都是从右到左的动画切换。这种交互方式通常表示页面正常的切换。还有一种页面进入方式就是从下往上进入，这种交互方式一般用于登录拦截时，弹出登录框，表示得是非正常状态下的页面切换。如下图所示，就是登陆拦截情况下的从下往上的页面进入效果：
![text](/images/ionic-login-modal.gif)

这篇文章就来说说，如何在ionic中实现这种从下往上的登录拦截交互方式。文章目录：
* 实现原理
* 实现步骤
    * 路由添加属性：`isPublic`来判断路由是否为公共路由；    
    * 将Modal包装成通用Service，方便调用；
    * 通过`$stateChangeStart`判断当前路由是否为publicUrl或者用户已经登录，然后做出相应的判断。

# 实现原理
实现原理很简单：
第一步：在写路由的时候，为每个路由添加`isPublic`属性标识该路由是否为匿名路由；
第二步：将`$ionicModal`包装成通用的Service，方便在`run()`方法中调用。
第三步：通过接收事件`$stateChangeStart`监听路由的变化，判断当前路由是否为publicUrl 或者已经是登录状态。如果是的话，则不处理，否的话，使用`$ionicModal`弹出登录框；

# 实现步骤
## 第一步：为路由添加`isPublic`属性。
直接为路由添加一个`data: {isPublic: false}`，`isPublic`有两个值：false表示该路由需要登录状态才能访问，true表示该路由可以匿名访问。这样就可以在监听`$stateChangeStart`事件中获取到这个属性了（如何获取后面会讲到）。
完整的route如下：
```javascript
 .state('footer.account', {    //这里footer.account中的footer是底部菜单栏的state，可以自行更换为你底部菜单栏的state
        url: '/account',
        views: {
          'account': {
            templateUrl: 'main/module/account/account.html',
            controller: 'AccountController'
          }
        },
        data: {isPublic: false}    //为路由添加该属性，判断是否为匿名路由
      })
```

## 第二步：将`$ionicModal`包装成通用的Service
由于监听`$stateChangeStart`事件是在`run()`方法中进行的，因此如果直接在`run()`方法中调用`$ionicModal`将会报错，因为调用该服务是需要传入`$scope`，而在`run()`方法中，只有`$rootScope`，如果直接将`$rootScope`代替`$scope`传入，将会报错。所以我们需要再多做一件事：将`$ionicModal`包装成Service，这样才能在`run()`方法中正确的使用。
代码如下：
```javascript
.factory('ModalService', ['$ionicModal', '$rootScope', '$q', '$injector', '$controller',
    function ($ionicModal, $rootScope, $q, $injector, $controller) {
      return {
        show: show
      };

      function show(templateUrl, controller, parameters) {
        var deferred = $q.defer(),
          ctrlInstance,
          modalScope = $rootScope.$new(),
          thisScopeId = modalScope.$id;
 
        $ionicModal.fromTemplateUrl(templateUrl, {
          scope: modalScope,
          animation: 'slide-in-up'
        }).then(function (modal) {
          modalScope.modal = modal;
 
          modalScope.openModal = function () {
            modalScope.modal = show();
          };
 
          modalScope.closeModal = function (result) {
            deferred.resolve(result);
            modalScope.modal.hide();
          };
 
          modalScope.$on('modal.hidden', function (thisModal) {
            if (thisModal.currentScope) {
              var modalScopeId = thisModal.currentScope.$id;
              if (thisScopeId === modalScopeId) {
                deferred.resolve(null);
                _cleanup(thisModal.currentScope);
              }
            }
          });
 
          //Invoke the controller
          var locals = {'$scope': modalScope, 'parameters': parameters};
          var ctrlEval = _evalController(controller);
          ctrlInstance = $controller(controller, locals);
          if (ctrlEval.isControllerAs) {
            ctrlInstance.openModal = modalScope.openModal;
            ctrlInstance.closeModal = modalScope.closeModal;
          }
 
          modalScope.modal.show();
        }, function (err) {
          deferred.reject(err);
        });
 
        return deferred.promise;
      }
 
      function _cleanup(scope) {
        scope.$destroy();
        if (scope.modal) {
          scope.modal.remove();
        }
      }
 
      function _evalController(ctrlName) {
        var result = {
          isControllerAs: false,
          controllerName: '',
          propName: ''
        };
        var fragments = (ctrlName || '').trim().split(/\s+/);
        result.isControllerAs = fragments.length === 3 && (fragments[1] || '').toLowerCase() === 'as';
        if (result.isControllerAs) {
          result.controllerName = fragments[0];
          result.propName = fragments[2];
        }
        else {
          result.controllerName = ctrlName;
        }
 
        return result;
      }
    }])
```
上面的`ModalService`返回的是一个promise对象，所以可以如下代码所示的用法，其中`show()`方法拥有三个参数：
* templateUrl：modal页面的地址；
* controllerName：controller的名字；
* parametes：传入的参数对象。
```
appModalService
.show('<templateUrl>', '<controllerName> or <controllerName as ..>', <parameters obj>)
.then(function(result) {
    // result 返回调用时传入的parameters
}, function(err) {
    // error
});
```
ps：此处的代码是来自ionic官方论坛上一位大神提供的，下面的参考资料中列出该文章的地址

### 第三步：使用`$stateChangeStart`监听路由变化
`$stateChangeStart`是来自`ui-router`中的广播事件，表示当state变化之前开始监听。用法如下：
```javascript
$rootScope.$on('$stateChangeStart', function(evt, toState, toParams, fromState, fromParams){
    //do somethting
    evt.preventDefault();    //用于阻止路由状态变化。也就是终止了跳转到下一个路由状态
})
```
其中参数如下：
* toState：表示目标state，是一个object，对应着我们所定义的路由对象。如果将toState打印出来，就会是下面的样子：
```javascript
{    
        url: '/account',
        views: {
          'account': {
            templateUrl: 'main/module/account/account.html',
            controller: 'AccountController'
          }
        },
        data: {isPublic: false}    //为路由添加该属性
      }
```
* toParams：表示目标state的参数对象；
* fromState：同`toState`一样，只不过表示的是上一个state；
* fromParams：表示上一个state的参数对象。

我们监听`$stateChangeStart`事件，获取了`isPublic`属性的值和token。如果这两者有一个为true，则路由继续跳转，false的话，则调用`ModalService`弹出登录modal框，用户登录成功后，则跳转到原本要进入的目标路由。如果用户点击关闭modal框，则返回到上一个路由。代码如下：
**路由监听代码**
```javascript
$rootScope.$on("$stateChangeStart", function (evt, toState, toParams, fromState, fromParams) {
        var isPublic = angular.isObject(toState.data) && toState.data.isPublic === true;    //判断当前state的data属性"isPublic" === true
        var token = Passport.getToken();    //这里的getToken()是我自己写的取得当前token的方法，可以换成你自己的方法
        if (isPublic || token ) {     如果该state是匿名访问路由 || token存在 
                 //do nothing
        }
        else {    //表示该state访问需要权限
          ModalService.show('main/common/component/login-modal.html', 'LoginController', {'login': true})    //调用ModalService.show()方法，显示登录modal框，这里还要指定Controller为LoginController，你也可以替换为自己的Controller
            .then(function (data) {
              if (data.login) {    //login 是我自定义的参数，后面会讲到
                $rootScope.$broadcast('login', 'true');        //向下广播 login事件，这样就可以在其他controller中接收到该事件，从而进行相应的操作
              }
              else {
                if(data.state){    //state也是我自定义的参数
                  $state.go(data.state);
                }else{
                  $state.go(fromState.name);
                }
              }
            });
        }
      });
```
**Login.html代码**
```html
<ion-modal-view ng-controller="LoginController" >
  <ion-header-bar>
    <h1 class="title">登录</h1>
    <button class="button button-clear button-assertive " ng-click="closeModal({'login': false})">    <!--这里调用关闭modal的方法，同时传递了参数对象{‘login’:false}-->
      <i class="icon ion-ios-arrow-thin-left"></i>
    </button>
  </ion-header-bar>
  <ion-content scroll="false">
    <form name="myForm"  novalidate>
      <div class="tm-form-item item-text-wrap">
        <label class="item-input tm-item-input">
          <input name="username" type="text" ng-model="user.username" placeholder="请输入手机号码"
                 required ng-maxlength="11" ng-minlength="11" >
        </label>
      </div>
      <div class="tm-form-item item-text-wrap">
        <label class="item-input tm-item-input">
          <input type="password" name="password" ng-model="user.password" placeholder="请输入密码"
                 required>
        </label>
      </div>
    </form>
  </ion-content>
</ion-modal-view>
```

**LoginController代码**
```javascript
function LoginController($scope, LoginService, $rootScope) {
    $scope.user = {};
    $scope.login = function () {
      LoginService.login($scope.user).then(function (data) {
         //login  这个就是写自己的登录逻辑
        $scope.closeModal({'login': true});    //登录成功后，调用关闭modal的方法，同时传递了参数对象{‘login’:true}，所传的参数在上面提到的路由监听方法中会用到
      }, function (err) {
      })
    };
  }
  LoginController.$inject = ['$scope', 'LoginService', '$rootScope'];
```

# 总结
到这里就可以实现App交互方式的登录拦截了。之前没有接触过App的开发无论是原生还是native，所以竟然一点都没有注意到页面切换还存在着这种差异。这次是公司里的iOS开发提醒了我，然后才注意到。想起了前几天看到的一句话**未来不仅存在着已知的未知数，更存在着未知的未知数**，永远记得保持学习的心态，活到老学到老真不是一句口号，更应该是行动。很庆幸自己选择了做程序猿，特别是选择了变化日新月异的前端。正是因为这种变化，会让自己一直都保持着学习的心态。

#参考资料
>https://forum.ionicframework.com/t/ionic-modal-service-with-extras/15357