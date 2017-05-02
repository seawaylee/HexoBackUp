---
title: Curl实现Get、Post 
date: 2017-03-09 22:01
tags: [Linux]
---


**Curl是Linux下一个很强大的http命令行工具，其功能十分强大。**

<!--more-->

## 一 CURL对HTTP的常规访问

### 1. 访问网站

$ curl http://www.linuxidc.com

回车之后，www.linuxidc.com 的html 显示在屏幕上了 

### 2. 保存页面

用curl option: -o

$ curl -o page.html http://www.linuxidc.com

可以看到屏幕上出现一个下载页面进度指示，等到100%，就保存完成了。使用wget 也可以的~。

## 二 GET模式

GET模式什么option都不用，只需要把变量写在url里面就可以了

例如：

$ curl http://www.linuxidc.com/test.cgi?param1=nickwolfe&param2=12345

## 三 POST模式

使用 option -d，

例如:

$ curl -d "param2=nickwolfe&param2=12345" http://www.linuxidc.com/login.cgi

## 四 POST+Json
curl -l -H "Content-type: application/json" -X POST -d '{"phone":"13521389587","password":"test"}' http://domain/apis/users.json

