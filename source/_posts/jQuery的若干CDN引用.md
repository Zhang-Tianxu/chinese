---
title: jQuery的若干CDN引用
tags:
  - web
  - JavaScript
categories:
  - web
date: 2018-11-22 14:23:23
---


![jQuery](https://codecondo.com/wp-content/uploads/2016/12/jQuery.png)
&emsp;许多人依据直觉认为把*jQuery*库放在服务器本地会更快，实际上从*CDN（Content Distribution Network）*引用*jQuery*往往会有更快的响应。原因有二：

1. 同样是要将*jQuery*发送到用户的浏览器，我们的服务器未必有大公司*CDN*服务器快。
2. 由于许多网站都引用了大公司*CDN*提供的*jQuery*，用户的浏览器中很可能已经缓存了*jQuery*，当再访问我们的网站中，浏览器从缓存中加载*jQuery*当然要更快。

&emsp;所以*CDN*引用*jQuery*不失为一种好的选择，而且各大网络公司都有提供该服务，稳定性也比较高。以下是知名公司的*jQuery*引用地址，可以根据需求选择适合我们网站的。
<!--more-->

* 首先是官方*jQuery CDN*，可以通过下面网址查询最新版本:[官方jQuery CDN](https://code.jquery.com)
* 然后是微软的
```javascript
<script src="https://ajax.aspnetcdn.com/ajax/jquery/jquery-1.9.0.min.js"></script>
```
* 接着是度娘的*jQuery CDN*。
```javascript
<script src="https://apps.bdimg.com/libs/jquery/2.1.4/jquery.min.js">
```
* 再来是Google的，这个在国内不太好用，除非你的用户里国外居多，否则慎选：
```javascript
<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.10.2/jquery.min.js">
```
* 当然还有几个不错的版本，一起写出来了：
```javascript
#又拍云的
<script src="https://upcdn.b0.upaiyun.com/libs/jquery/jquery-2.0.2.min.js">
#新浪的
<script src="https://lib.sinaapp.com/js/jquery/2.0.2/jquery-2.0.2.min.js">
#Staticfile CDN
<script src="https://cdn.staticfile.org/jquery/1.10.2/jquery.min.js">
```
&emsp;当然如果基于某种考虑要把*jQuery*库放在服务器本地的话，我也准备了*jQuery*[官方下载地址](http://jquery.com/download/)。
