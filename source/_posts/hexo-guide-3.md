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
每次在部署前，先把代码提交到`source`分支，再执行部署命令。
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
这样就建立了代码控制分支。因此，博客在部署到github之前，需要先把代码push到`source`分支，再执行部署命令。如下：
push 代码到`souce`分支
```bash
git add *
git commit -m "msg"
git push origin source
```
## 部署到github
```bash
hexo generate -d
```
#Bug记录
如果你使用了第三方主题，那么进行代码提交的时候，是无法将第三方主题提交到你的github repository中。