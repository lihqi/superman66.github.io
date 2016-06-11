---
title: MEAN：AngularJS+Node.js的REST API开发教程（一）：Express+Node+MongoDB环境搭建
date: 2016-05-25 22:20:01
tags:
- Nodejs
- MEAN
---
# 前言
终于要开始学习Node.js了。学习一门新语言或者框架入门的最好的方式就是采用项目驱动学习，这样能快速入门，有代入感。而对于新手而言，最适合入门的项目就是TODO项目。因为通过TODO项目，你可以了解到该语言或者框架对于增、删、改、查是如何操作。熟悉这些操作将有助于你更快地熟悉和进一步学习语言或者框架。这篇文章将介绍如何使用MEAN架构进行TODO项目的开发。
>　MEAN是一个Javascript平台的现代Web开发框架总称，它是MongoDB + Express +AngularJS + NodeJS 四个框架的第一个字母组合。它与传统LAMP一样是一种全套开发工具的简称。

# 功能列表
* 用户可以添加、编辑以及删除TODO事项
# 环境准备
本文的开发环境均为windows。
## Nodejs安装
>Node.js 是一个基于 Chrome V8 引擎的 JavaScript 运行环境。Node.js 使用了一个事件驱动、非阻塞式 I/O 的模型，使其轻量又高效。

windows下安装Node.js非常简单，到官网下载.msi文件安装即可。安装Node.js的同时会自动安装NPM。地址：https://nodejs.org/en/
## Express 安装
### 第一步：全局安装Express框架
>Express 是一种保持最低程度规模的灵活 Node.js Web 应用程序框架，为 Web 和移动应用程序提供一组强大的功能。

```bash
npm install -g express
```
Express安装成功之后，如果要通过命令查看Express是否安装成功，会提示Express不是内部或外部命令(注意：-V需要大写)
```bash
express -V
```
由于最新Express框架4.0以上将命令工具分离出来了，你需要单独安装一个命令工具（项目地址:https://github.com/expressjs/generator）
```bash
npm install -g express-generator
```
安装完Express和express-generator之后，
便可以成功使用Express命令
### 第二步：创建工程并启动Express服务
首先使用express创建一个名叫todo的工程，`-e`表示使用ejs作为模板引擎
```bash
express -e todo
```
创建工程后，需要进入该工程目录，利用npm install添加依赖
```bash
cdtodo
npm install
```
添加依赖之后，便可以启动该工程，启动成功的工程就已经运行在3000端口上
```bash
npm start
```
这时候打开浏览器访问：localhost:3000,出现下面的界面即表示安装成功
![text](/images/write-a-todo-1.png)
## MongoDB安装
>MongoDB是一个基于分布式文件存储的数据库。由C++语言编写。旨在为WEB应用提供可扩展的高性能数据存储解决方案。是Nosql的一种。

### 第一步：下载
官网下载地址：https://www.mongodb.com/download-center#community。选择与你操作系统对应的版本进行下载，安装即可。
### 第二步：配置环境变量
安装完成后，通过cmd进入到mongodb文件夹（即MongoDB在你的电脑的安装文件夹）下的bin文件夹，就可以执行mongodb的命令。但如果每次执行mongdb的命令都要进入到bin文件夹下，岂不是很麻烦。所以我们可以将bin文件夹设置为环境变量。
下载安装完成后的目录结构如下：将路径`d:/mongodb/bin/`(此路径是你安装mongodb的文件夹路径)，设置为环境变量即可。这样就直接可以命令行使用mongodb的命令，而无需进入到mongodb对应的文件夹。
![text](/images/write-a-todo-2.png)
### 第三步：配置MongoDB数据库
创建一个`mongodb.config`文件，用于配置 MongoDB数据库的dbpath（数据库存储路径）和logpath（日志文件存储路径）。为什么要设置这两个路径呢？因为MongoDB每次启动服务的时候需要指定数据库存储路径和日志文件存储路径，如果没有指定的话，将无法正常启动数据库。MongoDB不仅提供了通过配置文件的方式来设置dbpath和logpath，还可以通过`--dbpath`和`--logpath`的选项来配置路径。我们这里采用config文件的形式。
`mongodb.config`文件内容如下
```
# 这里的路径应该换为你自己的mongodb路径
##store data here
dbpath=E:\mongodb\data
 
##all output go here
logpath=E:\mongodb\log\mongo.log
```
由于安装后的MongoDB并不存在data和log这两个文件夹，因此我们需要在mongodb文件夹下手动创建这两个文件夹。不然在启动mongodb服务器的时候会提示找不到这两个文件夹。一切都设置好了之后，我们就可以通过下面的命令来启动mongodb服务器。
```bash
mongod.exe --config D:/mongo/mongo.config
```
看到下面的输出信息就说明启动MongoDB数据库成功了。
![text](/images/write-a-todo-3.png)
这时候我们再打开新的命令行窗口，输入`mongo`，就可以看到当前连接的数据库信息
![text](/images/write-a-todo-4.png)
### 第四步：添加MongoDB到Windows服务
通过上面的配置和安装我们已经成功安装了mongodb并可以使用了。但是这样还存在着不方便的地方就是，如果你每次要在使用数据库的时候，需要通过`mongod.exe --config D:/mongo/mongo.config`手动启动数据库服务器。不过MongoDB已经考虑到这一点了，已经提供了一个选项`--install`，将MongoDB添加到Windows Service中。这样我们就可以通过设置Windows Service自动启动代替人工启动。
执行下面的命令就可以将MongoDB添加到Windows服务中：
```bash
mongod.exe --config D:/mongo/mongo.config --install
```
添加完成后，你就可以在Windows服务中看到
![text](/images/write-a-todo-5.png)
到这里你就可以开始捣腾MongoDB数据库了。
