---
title: Hexo博客搭建教程（三）：Hexo博客代码版本控制
date: 2016-02-22 22:03:43
tags: Hexo
---
由于Hexo只会将生成后的public文件夹部署到github上，导致无法对博客进行代码版本控制。同时如果需要备份代码的话，只能通过其他手段来实现。
本文介绍了如何利用github分支对代码进行版本控制，同时起到备份代码的作用。
<!-- more -->
# 解决思路
实现Hexo博客代码版本控制以及备份的思路如下：
通过新建一个`source`分支用于专门存放hexo代码。原先的`master`分支依然不变，作为hexo 部署的分支。
每次在部署后，再把代码提交到`source`分支。
# 实现步骤
## 1、本地创建git 仓库
```bash
git init
```
## 2、添加远程库
```bash
git remote add origin <git repository url>
```
## 3、创建source分支
```bash
git checkout -b source
```
## 4、提交文件及分支，并push到远程仓库
```bash
git add *
git commit -m 'message'
git push origin source 
```
其中`source`为分支名称。

这样就建立了代码版本控制分支。之后只要将博客在部署到github之后，将代码push到`source`分支上。代码如下：
```bash
git add *
git commit -m "udpate site"
git push origin source
```
# 问题记录
如果你使用了第三方主题，在进行代码提交的时候，是无法将第三方主题提交到你的github repository中，会出现 `untracked content`的提示。
这是因为第三方主题本身也是一个git项目。你无法将别人的git项目直接通过add 和commit的方式提交到你自己的git repository。
也就说，你无法提交处于 `untracked`状态的文件。
解决办法：
* 添加 `submodule`的方式，将主题作为submodule提交到你的git repository
* 删除主题文件夹下的`.git`文件夹。如果这时候还不能提交，可以新建个文件夹，随便命名，将主题文件夹内的东西复制到新建的文件夹。再通过`git add`提交就可以了。