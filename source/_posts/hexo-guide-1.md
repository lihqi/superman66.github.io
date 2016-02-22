---
title: Hexo博客搭建教程（一）：Hexo介绍及搭建
date: 2016-02-21 21:33:34
tags: Hexo
---
终于有了自己的github博客了。记录下搭建Github.io博客的过程以及当中所遇到的坑。

<!-- more -->
# Hexo是什么
Hexo是一个基于nodejs快速、简洁且高效的博客框架。可以方便的生成静态网页文件托管到github pages。具有超快渲染速度、支持GitHub Flavored Markdown语法、一键部署等优点，同时还拥有强大的插件系统，方便用户进行定制化开发。
# Hexo 安装
本教程只针对windows用户。
## 安装前提
在安装Hexo之前，请先安装
* Node.js
* Git

## 安装 Git
下载[msysgit](https://git-for-windows.github.io/)进行安装即可。
## 安装 Nodejs
在windows下安装nodejs非常简单，直接去[官网](https://nodejs.org/)下载进行安装即可。
## 安装 Hexo
Hexo安装也很简单。在Git和Node.js都安装后，直接使用npm进行安装即可
```bash
npm install -g hexo-cli
```
## 创建Hexo文件夹
在你喜爱的文件夹下，如（`D:\hexo`）,右键选择`git bash`,执行以下命令。便会自动新建所有的文件
```bash
hexo init
```
## 安装所依赖的包
```bash
npm install
```
## 本地查看
到这一步，你已经在本地安装了Hexo博客。你可以通过以下的命令在本地查看博客。
### 生成静态文件
```bash
hexo generate
```
### 运行服务
```bash
hexo server
```
此时在浏览器中输入`localhost:4000`，便可以看到博客了。至此，本地博客已经搭建好了。但是此时博客还只是在本地，别人是无法访问的。

# Github部署
## 注册Github账号
有账号的人跳过，没有的话[注册](https://github.com/)下也很简单。
## 创建 repository
在个人github主页右下角点击 `New repository`,创建一个新的repository。
新的repository的名字应该跟你github账号的名字一样。比如我的github账号是`superman66`，
那么新的repository的名字就应该为`superman66.github.io`。
## 部署
进入博客所在的文件夹(如`D:\hexo`)，找到_config.yml文件，修改以下的配置，将下面的`superman66`都换成你自己的账户名。
```
deploy:
  type: git
  repository: https://github.com/superman66/superman66.github.io.git
  branch: master
```
*注意：hexo 3.0以下，type要写成 `github`*
这个参数是用来配置网站一键部署的。让你只需要一条命令就可以将网站部署到服务器上。
配置文件修改之后，执行一下命令便可以完成部署了。
```bash
hexo generate
hexo deploy
```

### hexo常用命令
* `hexo new "post_name"`用于生成新的文章；
* `hexo generate`用于生成静态文件；
* `hexo server` 用于启动本地服务；
* `hexo deploy`用于将生成的静态文件部署到repository上。

同时这四个命令还支持简写
 `hexo g` === `hexo generate`
 `hexo s` === `hexo server`
 `hexo d` === `hexo deploy`
 `hexo n` === `hexo new`
 至此，你已经将本地的博客部署到github上。你可以通过username.github.io(`username`换成你自己的账户名)来访问你的博客了 

 ## 问题记录
 在部署过程中，执行`hexo d`进行部署的时候，出现以下的错误
 **`hexo bash: /dev/tty: No such device or address`**
 ### 解决办法：
 安装github for Window，点击这里进行[下载](https://github-windows.s3.amazonaws.com/GitHubSetup.exe),使用git shell再执行`hexo d`命令进行部署即可。
 ### 问题的原因：
 google找到的说法是，由于window安装的git版本问题导致的。
