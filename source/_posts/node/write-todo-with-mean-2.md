---
title: MEAN：AngularJS+Node.js的REST API开发教程（二）：项目开发
date: 2016-05-28 10:12:27
tags:
- Nodjs
- MEAN
---
# 前言
经过前面那篇文章的准备，现在已经搭建好MEAN架构的开发环境了。这篇文章将介绍如何实现TODO的项目。
创建一个TODO项目的步骤如下：
* 创建一个Express工程项目
* 创建一个SPA，用于前台创建TODO事项，编辑和删除TODO事项以及显示TODO列表
* 保存TODO到数据库（本文采用Mongoose）
* 使用Node.js创建REST API
* 使用AngularJS作为前端调用REST API

Github地址：https://github.com/superman66/node-express-todo
项目效果图：
![text](/images/write-a-todo-6.gif)

## Express项目结构
在第一篇文章，我们已经创建了一个名字为todo的Express项目了，创建成功后的目录结构如下：
![text](/images/write-a-todo-7.png)
接下来就介绍下express项目下的文件夹的作用
* node_modules：通过npm安装依赖包所存放的路径；
* public：存在公用的文件如CSS文件，图片文件等；
* routers：项目路由文件夹，项目所有的路由文件都放在该文件夹下；
* views：页面文件，可以是html文件，也可以是模板引擎文件；
* app.js：app.js是项目的主要入口文件；
* db.js：db.js存放着数据库连接信息，是自己创建的，具体后面会介绍。

## TODO项目开发
介绍了Express项目的文件夹结构之后，就可以开始进入代码开发了。
## 安装依赖包
在刚创建Express项目后，我们已经通过`npm install`安装了必要的依赖。但这还不够，我们还需要额外安装以下的npm：
* mongoose
* angular

使用以下命令，即可安装：
```bash
npm install mongoose angular
```
我们使用的是MongoDB作为数据库，由于直接使用MongoDB操作连接数据库一点都不方便，因为在这里我们采用了`mongoose`。
>Mongoose是基于node-mongodb-native开发的MongoDB nodejs驱动，可以在异步的环境下执行。Mongoose和MondoDB的关系就好比jQuery和原生JavaScript。使用Mongoose操作数据数据库更加方便 。


## 项目开发步骤
### 第一步：创建Express工程
这一步我们已经完成了。
### 第二步：后台配置
这一步包括数据库配置，静态文件夹服务器配置以及路由配置。
*数据库配置

首先在根目录下创建一个`db.js`，用于存放数据库连接信息。我们还需要创建一个`Todo Model`，该model有id，content,update_at三个属性。在mongoose中，model是可以操作数据库的。代码如下：
```javascript
/**
* Created by superman on 2016/5/22.
*/
var mongoose = require('mongoose');
var Schema = mongoose.Schema;
 
var Todo = new Schema({
    id : String,
    content : String,
    update_at : Date
});
 
mongoose.model('Todo', Todo);
mongoose.connect('mongodb://localhost/express-todo');    //此处的express-todo是mongodb数据库的名字，所以你需要先创建这个数据库
```
接下来在`app.js`顶部中添加引用
```javascript
var db = require('./db');
```
* 配置静态文件服务器

Express项目默认public文件夹如静态文件的位置，我们也可以再添加文件夹作为静态文件服务器，代码如下：
```javascript
app.use(express.static(path.join(__dirname, 'public')));  //配置静态文件服务器
app.use(express.static(path.join(__dirname, 'node_modules')));  //可以配置多个静态文件服务器
```
* 路由配置

Express项目使用了路由中间件，将路由信息从app.js中剥离出来，单独放在router文件夹中。但是我们需要在`app.js`中配置，代码如下：
```
app.use('/', routes);
```
完整的`app.js`代码如下：
```javascript
var express = require('express');
var db = require('./db');
var path = require('path');
var favicon = require('serve-favicon');
var logger = require('morgan');
var cookieParser = require('cookie-parser');
var bodyParser = require('body-parser');
var partials = require('express-partials');
 
var routes = require('./routes/index');
var users = require('./routes/users');
 
var app = express();
 
// view engine setup
app.set('views', path.join(__dirname, 'views'));
//app.set('view engine', 'ejs');
//app.set(partials());  //启动模板文件配置
 
// uncomment after placing your favicon in /public
//app.use(favicon(path.join(__dirname, 'public', 'favicon.ico')));
app.use(logger('dev'));
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: false }));
app.use(cookieParser());
app.use(express.static(path.join(__dirname, 'public')));  //配置静态文件服务器
app.use(express.static(path.join(__dirname, 'node_modules')));  //可以配置多个静态文件服务器
app.use('/', routes);
 
// catch 404 and forward to error handler
app.use(function(req, res, next) {
  var err = new Error('Not Found');
  err.status = 404;
  next(err);
});
 
// error handlers
 
// development error handler
// will print stacktrace
if (app.get('env') === 'development') {
  app.use(function(err, req, res, next) {
    res.status(err.status || 500);
    res.render('error', {
      message: err.message,
      error: err
    });
  });
}
 
// production error handler
// no stacktraces leaked to user
app.use(function(err, req, res, next) {
  res.status(err.status || 500);
  res.render('error', {
    message: err.message,
    error: {}
  });
});
 
 
module.exports = app;
```
### 第三步：创建REST API 路由
在创建之前，我们可以先规划下，我们都需要用到哪些API:
* 返回所有的todo列表： `/api/todos`、`GET`
* 创建todo： `/api/todo`、`POST`
* 更新todo：`/api/todo/:id`、`PUT`
* 删除todo：`/api/todo/:id`、`DELETE`


这个四个API的代码如下：
```javascript
/* restful api */
//get all todo
router.get('/api/todos', function(req, res, next){
    Todo
        .find()
        .sort('-update_at')
        .exec(function(err, todos){
            if(err){
                res.send(err);
            }
            res.json(todos);
        });
});
 
//POST that create todo and return all todos
router.post('/api/todo', function(req, res, next){
    new Todo({
        content : req.body.content,
        update_at : new Date()
    }).save(function(err, todo, count){
        if(err){
            res.send(err);
        }
        Todo
            .find()
            .sort('-update_at') //更加日期排序
            .exec(function(err, todos){
                if(err){
                    res.send(err);
                }
                res.json(todos);
            });
    })
});
 
//删除后返回所有
router.delete('/api/todo/:id', function(req, res, next){
    Todo.findById(req.params.id, function(err, todo){
        todo.remove(function(err, todo){
            if(err){
                res.send(err);
            }
            Todo
                .find()
                .sort('-update_at') //更加日期排序
                .exec(function(err, todos){
                    if(err){
                        res.send(err);
                    }
                    res.json(todos);
                });
        })
    })
});
//Update
router.put('/api/todo/:id', function(req, res, next){
    //更新操作
    Todo.findById(req.params.id, function(err, todo){
            if(err){
                res.send(err);
            }
        todo.content = req.body.content;
        todo.save(function(err){
            console.log('update success');
        });
    });
            Todo
                .find()
                .sort('-update_at') //更加日期排序
                .exec(function(err, todos){
                    if(err){
                        res.send(err);
                    }
                    res.json(todos);
                });
});
```
最后我们还需要在添加一个api，用于显示前端页面：
```javascript
router.get('/', function(req, res){
    res.sendfile('./views/index.html');    //angular应用首页的位置
});
```
## 第四步：创建AngularJS应用
新建`index.module.js`，用于定于AngularJS应用，代码如下：
```javascript
/**
* Created by superman on 2016/5/25.
*/
(function () {
    'use strict';
 
    angular.module('TodoApp', [])
        .factory('TodoService', ['$http', function ($http) {
            var _getTodoList = function () {
                return $http.get('/api/todos');
            };
            var _createTodo = function (todo) {
                return $http({
                    method: 'POST',
                    url: '/api/todo',
                    data: todo
                })
            };
            var _deleteTodo = function (id) {
                return $http({
                    method: 'DELETE',
                    url: '/api/todo/' + id
                })
            };
            var _updateTodo = function (todo) {
                return $http({
                    method: 'PUT',
                    url: '/api/todo/' + todo._id,
                    data: todo
                })
            };
            return {
                todoList: _getTodoList,
                createTodo: _createTodo,
                deleteTodo: _deleteTodo,
                updateTodo: _updateTodo
            }
        }])
        .controller('TodoController', ['TodoService', function (TodoService) {
            var vm = this;
            vm.todos = [];
            vm.todo = {
                content: ''
            };
            TodoService.todoList().success(function (data) {
                vm.todos = data;
            }).error(function (err) {
                console.log(err);
            })
 
            vm.createTodo = function (todo) {
                TodoService.createTodo(todo).success(function (data) {
                    vm.todos = data;
                    vm.todo.content = '';
                });
            };
 
            vm.delTodo = function (id) {
                TodoService.deleteTodo(id).success(function (data) {
                    vm.todos = data;
                })
            };
 
            vm.updateTodo = function(todo){
                // console.log(todo);
                TodoService.updateTodo(todo).success(function(data){
                    //vm.todos = data;
                })
            }
 
        }]);
})();
```
在这个文件中，我们把所有的http请求都放在了`TodoService `中，然后在Controller中注入。这样就可以在Controller中使用了。
## 第五步：创建HTML页面
这里就不对AngularJS的语法做介绍了，HTML代码如下：
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>TODO</title>
    <link rel='stylesheet' href='/stylesheets/style.css'>
</head>
<body ng-app="TodoApp">
<div class="container" ng-controller="TodoController as vm">
    <h1 class="title">TODO</h1>
    <section class="form-content">
        <form  ng-submit="vm.createTodo(vm.todo)">
            <input class="form-control" type="text" name="content" ng-model="vm.todo.content" placeholder="添加待办事项"
                   required/>
            <!--<input type="submit" value="添加">-->
        </form>
        <ul class="todo-list">
            <li ng-repeat="todo in vm.todos">
                <a>
                    <input type="text" class="form-control btn-primary"  ng-model="todo.content" ng-blur="vm.updateTodo(todo)"/>
                </a>
                <a class="btn-del" ng-click="vm.delTodo(todo._id)" title="Delete this todo item"></a>

            </li>
        </ul>
    </section>
</div>
<footer>
    输入回车键进行提交
</footer>
</body>
<script src="/jquery/dist/jquery.min.js"></script>
<script src="/angular/angular.min.js"></script>
<script src="/index.module.js"></script>
</html>
```
到这里，一个基于MEAN的TODO项目就完成了。你可以运行`npm start`运行项目，在浏览器中输入`localhost:3000`就可以看到页面了。

# 总结
这个项目一开始只是想学习如何构建一个Express+Nodejs的项目，在完成之后开始思考，如何将其变成一个前后端分离的项目。想到我对AngularJS比较擅长，于是前端就采用AngularJS，代替了项目中自带的ejs模板引擎。然后这个技术栈就组成了简易型的MEAN架构了。突然想起来最开始学Express+Node项目的初衷就是为了掌握MEAN Stack，通过这么一个过程，虽然不能说已经掌握了MEAN Stack，但是已经对其有了结构性的了解，因为MEAN最基础的组成部分就是`MongoDB`、`Express`、`AngularJS`、`Nodejs`。对MEAN Stack有兴趣的，可以前往官网http://mean.io/了解更多。

参考资料：
http://www.jdon.com/idea/nodejs/web-app-with-angularjs-and-rest-api-with-node.html
http://dreamerslab.com/blog/en/write-a-todo-list-with-express-and-mongodb/
https://cnodejs.org/topic/504b4924e2b84515770103dd

