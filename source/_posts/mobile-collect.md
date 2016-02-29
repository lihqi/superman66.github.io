---
title: 收集移动端代码
date: 2016-02-29 21:44:05
tags: 移动端
---
收集了移动端的一些代码
<!-- more -->
## 1、input输入框更改placeholder样式
```cpp
/* webkit solution*/
input :: -webkit-input-placeholder {
    text-align: center;
    color: #333;
}
/* mozilla solution*/
input:-moz-placeholder{
    text-align: center;
    color: #333;
}
```
## 2、iOS系统去除input输入框自带的内阴影
```cpp
input{
    -webkit-appearance: none;
}
```
### 3、去除数字输入框的小三角
```cpp
/* webkit*/
input::-webkit-outer-spin-button,
input:: -webkit-inner-spin-button{
   -webikt-appearance: none;
}
/*mozilla */
input[type="numer"]{
    -moz-appearance: textfield;
}
```
