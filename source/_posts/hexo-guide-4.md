---
title: Hexo博客搭建教程（三）：Hexo优化与个性化
date: 2016-02-23 22:24:11
tags: Hexo
---
前面的文章介绍了如何打造一个属于自己的博客。但是这个博客还只是拥有基本的功能。对于一个站点而言，我们还需要很多功能来完善它，比如需要网站访问统计数据，网站评论功能等。这篇文章将从以下几个方面介绍如何自定义你的博客。
* 添加统计代码
* 添加多说评论功能
<!-- more -->
# 添加统计代码
无数据，不运营。对于一个站点而言，网站的访问数据十分重要，数据分析是网站运营的一部分。虽然这只是一个博客，但我们也可以将其当做一个网站来运营。
要想统计网站的访问数据，一般通过第三方数据分析网站，添加相应的统计代码来进行数据统计。由于google analytics会出现被墙的原因以及统计数据不够及时（一般需要第二天才能看到报表），因此我采用了[CNZZ数据专家](http://www.cnzz.com/)的数据统计功能。（至于为啥不用百度统计，由于百度全家桶实在是呵呵。。。）。
## 获得CNZZ统计代码
没有账号的自行去注册。注册完，填写你的博客站点信息之后，拿到CNZZ提供的统计代码。
## 编辑 `themes/yilia/_config.yml`文件
```yml
cnzz_tongji: true  # 开启cnzz统计
google_analytics: ''
```
## 新建`cnzz_tongji.ejs`文件
在`themes/yilia/layout/_partial/`文件夹下新建文件`cnzz_tongji.ejs`，文件内容如下:
```ejs
<% if(theme.cnzz_tongji) {%>
	<script type="text/javascript"> 
	//cnzz analytics code
	</script>
<%}%>
```
## 将`cnzz_tongji.ejs`添加到`head.ejs`
打开位于`themes/yilia/layout/_partial/`文件夹下的`head.ejs`，在`</head>`之前添加以下代码，将统计代码添加到页面中
```ejs
<% _partial/cnzz_tongji.ejs%>
```
添加后便可以去[CNZZ数据专家](http://www.cnzz.com/)查看博客的访问数据统计了。