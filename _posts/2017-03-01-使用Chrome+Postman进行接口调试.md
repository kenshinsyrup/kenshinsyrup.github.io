---
layout: post #post
title: 使用Chrome+Postman进行接口调试 #post title
categories: 工具 #post category, seperated by spcace
tags: Postman Chrome #post tag, seperated by spcace
---

时隔多年，又重新看了一遍闪灵

失眠中 -_-#

开发过程中经常需要进行接口调试，测试完毕才能交接前端使用，调试过程中，发现了[Postman](https://www.getpostman.com/)这个神器，在配合Chrome控制台的情况下，能完美应付大部分情况下的调试工作

在此，记录下调试过程

## 工具

嗯，这节是废话

Chrome: 这个星球最好的浏览器我们都需要有一个

Postman: 这个有[Chrome插件版](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?hl=zh-CN)和[App版](https://www.getpostman.com/products)，我是使用了后者

## 使用方法

### Postman介绍

简单的介绍一下Postman

个人觉得，Postman是一个特别特别好的产品:
- 能模拟常用的HTTP请求，包括GET，POST等等，用于调试现在流行的RESTful接口相当方便
- 支持多种授权认证方式
- 支持历史记录，收藏，分享等功能

（软广在博客内容前部添加，括号里内容记得删掉）

![Postman界面](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-03-01-post01/screenshot_zpsf58vh6me.png)

界面很简洁，常用的功能已经标注，这就是平时最多使用的功能了

### 使用Postman调试GET接口

![使用GET请求获取Applysquare网站主页](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-03-01-post01/screenshot%203_zpsewvhje6o.png)

GET请求很简单，就不说了

### 使用Postman调试POST接口

这里主要是POST过程中会遇到CSRF问题，解决方法就是获取到合法的CSRF Token，然后添加到POST请求的Header中

*这个部分是因网站而异的，很多网站都很难在不熟悉的情况下完成POST请求，不过我们又不是黑客，只是想安心调试下自己的接口而已，那么下面提到的所有信息理论上我们都能拿到的*

我自己遇到的情况是这样：

1、我需要向``localhost:8000/ajax/g``地址POST一段JSON数据，但是会遇到``403 Forbidden, CSRF 验证失败``

2、首先使用合法账户登录到``localhost:8000``，这里有一点需要注意，一般脑子正常的网站，都是在login之后直接跳转到网站首页的，所以，在Chrome控制台中，将``Preserve log``选项打勾，这样在登录后直接跳转到首页的过程中，不会丢失POST请求的日志，否则只会看到最后一个请求也就是一个GET主页的请求的日志
![Chrome控制台](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-03-01-post01/screenshot%205_zpsgbmetpbn.png)

3、在控制台中找到``login/``，将其中``Request Headers``的``Cookie``及``CSRF``等内容添加到Postman的请求头中

4、在Postman中Authorization中选择合适的Auth type，输入对应的合法用户信息

5、再次执行步骤1，理论上能完成POST过程




















