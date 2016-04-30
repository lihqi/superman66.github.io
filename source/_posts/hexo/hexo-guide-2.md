---
title: Hexo博客搭建教程（二）：Hexo博客的配置、使用
date: 2016-02-21 22:41:24
tags: Hexo
---
本文主要介绍如何对Hexo博客站点进行个性化的设置、如何安装主题以及主题的设置。

<!-- more -->
经过上一篇文章，我们已经学会如何搭建Hexo博客以及将博客部署到github上了。这篇文章主要讲如何对自己的博客站点进行个性化配置以及如何发表新文章。
博客的配置一个是站点的配置:`d:\hexo\_config.yml`，一个是主题的配置:`d:\hexo\themes\yilia\_config.yml`
## 站点的配置
```yml
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site
title: Superman的博客 #站定的名称
subtitle: 超人不会飞 #站点的副标题
description: 超人前端学习博客 #站点的描述
author: Superman 
email: supermanchc@gmail.com
language: zh-Hans # 语言 使用中文需要使用zh-Hans
timezone:  #默认操作系统的时间

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
##url 在没有绑定域名前，不需要填写
url: http://yoursite.com
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:

# Directory 目录格式，不修改
source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:

# Writing 写作布局，不修改
new_post_name: :title.md # File name of new posts
default_layout: post
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
future: true
highlight:
  enable: true
  line_number: true
  auto_detect: false
  tab_replace:

# Category & Tag
default_category: uncategorized
category_map:
tag_map:
# Archives 默认值为2，这里都修改为1，相应页面就只会列出标题，而非全文
## 2: Enable pagination
## 1: Disable pagination
## 0: Fully Disable
archive: 1
category: 1
tag: 1
# Date / Time format
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD
time_format: HH:mm:ss

# Pagination
## Set per_page to 0 to disable pagination
per_page: 5
pagination_dir: page

# Extensions 配置主题
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: yilia

# Deployment 配置部署github站点，改为自己的github repository
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repository: https://github.com/superman66/superman66.github.io.git
  branch: master
```
## 安装新主题
Hexo此时用的是默认的主题，如果需要更换主题，可以去[主题市场](https://github.com/tommy351/hexo/wiki/Themes)挑选自己喜爱的主题。这里以安装`yilia`主题为例。
#### 安装

``` bash
$ git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia
```

#### 配置

修改hexo根目录下 `_config.yml` 的themes: 
```yml
# Extensions 配置主题
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: yilia
```

#### 更新

``` bash
cd themes/yilia
git pull
```
## 配置主题
这样就为你的博客站点安装了新的主题。接下来对主题进行配置
主题配置文件在主目录下的`_config.yml`：
```
# Header
menu:
  主页: /
  所有文章: /archives
  # 随笔: /tags/随笔

# SubNav
subnav:
  github: "#"
  weibo: "#"
  rss: "#"
  zhihu: "#"
  #douban: "#"
  #mail: "#"
  #facebook: "#"
  #google: "#"
  #twitter: "#"
  #linkedin: "#"

rss: /atom.xml

# Content
excerpt_link: more
fancybox: true
mathjax: true

# Miscellaneous
google_analytics: ''
favicon: /favicon.png

#你的头像url
avatar: ""
#是否开启分享
share: true
#是否开启多说评论，填写你在多说申请的项目名称 duoshuo: duoshuo-key
#若使用disqus，请在博客config文件中填写disqus_shortname，并关闭多说评论
duoshuo: true
#是否开启云标签
tagcloud: true

#是否开启友情链接
#不开启——
#friends: false
#开启——
friends:
  奥巴马的博客: http://localhost:4000/
  卡卡的美丽传说: http://localhost:4000/
  本泽马的博客: http://localhost:4000/
  吉格斯的博客: http://localhost:4000/
  习大大大不同: http://localhost:4000/
  托蒂的博客: http://localhost:4000/

#是否开启“关于我”。
#不开启——
#aboutme: false
#开启——
aboutme: 我是谁，我从哪里来，我到哪里去？我就是我，是颜色不一样的吃货…
```

