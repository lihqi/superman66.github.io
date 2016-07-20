---
title: AngularJS去掉URL里的#号
date: 2016-07-20 22:10:54
tags:
- AngularJS
---
# 前言
用过AngularJS的都知道，AngularJS的路由会带有`#`，最近在考虑AngularJS框架SEO的解决方案，比较普遍的做法是采用`prerender.io`提供的解决方案，（待补充）。
采用该方案的其中一个步骤就是要去掉路由中的`#`号，网上找了很多教程大多数只提到了一部分，就是设置`htm5lMode(true)`，但这只是其中一步，并不完整。只设置了html5Mode项目并不能正确运行。本文就介绍如何完整实现路由去掉`#`。要想完全实现，还需要后端服务器配置，本文web服务器采用的是nginx，所有关于服务器的配置都是在nginx的基础下。
# 步骤一：设置html5Mode
在你的路由配置文件中添加`html5Mode(true)`，代码如下
```javascript
    .config(['$locationProvider', function ($locationProvider) {
        $locationProvider.html5Mode(true);
    }])
```

# 步骤二：添加`<base>`设置根目录
在路由中添加了html5Mode后，需要在首页添加`<base href="xxx/xxx">`，href路径为项目根目录的文件夹。比如我的项目index.html的路径是`tmw/app/index.html`，那么`tmw/app`就是根目录路径。
此时，你访问项目的首页就可以发现，url当中已经没有#号了。但是这时候你如果访问原来的一些带有#号的路径时，发现是不能跳转的，而且#会被转义，如下图所示：
![text](/images/angular-remove-hash.png)
解决办法就是用$location.path()来进行页面跳转。
# 步骤三：配置nginx
之所以要配置nginx是因为，当你按照上面的设置之后，一旦刷新页面，就会报404错误。所以我们可以用nginx来解决这个问题：当出现404时，nginx将请求重定向到项目的首页（这里为index.html），然后再由Angular的路由去解析url，从而跳转到正确的路径。
nginx配置如下：
···bash
location / {
            root   html;
            index  index.html index.htm;
            try_files $uri $uri/ /tmw/app/index.html =404;
        }
···
其中：
* root是配置nginx项目根目录，`html`这个文件夹是nginx默认的根目录。（我的项目路径`/html/tmw/app`）
* index配置服务器首页

所谓只要把/tmw/app/index.html换成你自己的项目首页路径即可。
# 参考资料
>https://gist.github.com/zdwolfe/6721115
>认识与使用$location服务:http://www.alloyteam.com/2015/04/angular-location/


