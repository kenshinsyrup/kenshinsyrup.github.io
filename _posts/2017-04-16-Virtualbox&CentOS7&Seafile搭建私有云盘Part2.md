---
layout: post #post
title: VirtualBox & CentOS7 & SSH & Seafile & ngrok 搭建私有云盘Part2 #post title
categories: Project #post category, seperated by space
tags: VirtualBox CentOS7 SSH Seafile  ngrok #post tag, seperated by space
---

### 在服务器安装Seafile

[Seafile](https://www.seafile.com/en/home/)官网介绍自己为一个企业级的，高可靠，高性能的文件同步和共享平台。实际上类似的平台有很多，比如还有广为人知的[ownCloud](https://owncloud.org/)等

- 服务器端下载Seafile

在Seafile的官网是只有网页下载按钮没有适合无GUI服务器操作系统的下载方式的，但是我们能够从官网获知Seafile的最新版本，然后在[Bintray](https://bintray.com/)网站下载

当前Linux Server端最新版本

![当前Linux Server端最新版本](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2021_zps7cza6mnb.png)

首先，在宿主机SSH连接到服务器后，使用命令``sudo yum install wget``为服务器安装``wget``

![安装wget](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2022-1_zpsalctobxm.png)

之后，即可以使用``wget``的方式下载指定版本的Seafile，版本号在官网查询替换即可

![下载Seafile](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2022_zpsx3v8d923.png)

### 配置Seafile

本节的配置Seafile的内容，是达拉然巨坑，我是根据官方文档中[Deploying Seafile with MySQL](https://manual.seafile.com/deploy/using_mysql.html)一节来做的，遇到的问题会在下面详述

**按照文档要求**，解压文件，建立文件夹

![解压文件建立文件夹](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2023_zps9bbxcriw.png)

这里有个小工具``tree``很有用，我们安装一下，命令为``sudo yum install tree -y``

![安装tree](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2024_zpszjtuq1iu.png)

作用看下图就知道了

![tree查看目录结构](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2025_zpsixdyidra.png)

**按照官方文档要求**，安装依赖

![安装依赖](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2026_zpsnpzwnsfb.png)

**安装官方文档要求**，运行``setup-seafile-mysql.sh``脚本

![运行setup-seafile-mysql.sh脚本](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2027_zpsn45ldjoy.png)

这里提示了缺少python的``setuptools``模块，**注意**，这里虽然高亮提醒的是缺少``setuptools``，但是需要安装的东西在其下面的非高亮log部分，在CentOS中，执行``yum install python-distribute``进行安装

![安装缺少的setuptools模块](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2028_zps1hy5begc.png)

**按照官方文档要求**，再次运行``setup-seafile-mysql.sh``脚本

![再次运行``setup-seafile-mysql.sh``脚本](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2029_zpsm9rjsikl.png)

这里成功的开始运行脚本，需要点击``ENTER``按键开始，开始之后的内容会要求设置一系列的相关内容，具体的内容表达的信息基本可以猜到，猜不到的也罗列的很清楚在[Deploying Seafile with MySQL](https://manual.seafile.com/deploy/using_mysql.html)文档中

但是

在运行到mysql相关的登入内容时，出现了错误

![mysql登入错误](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2030_zpsg4sugtkh.png)

这个坑花了很多时间，Google出来的答案也是五花八门，最后的解决方式是使用[MariaDB](https://mariadb.org/)来完成数据库的创建

关于MariaDB和MySQL的关系太复杂这里不说了先

- 终止脚本，安装并启动MariaDB

``ctrl + C``终止脚本，使用命令``sudo yum install mariadb mariadb-server``安装MariaDB及其服务

![安装MariaDB]http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2031_zpsnsswzcdu.png()

启动MariaDB服务``sudo systemctl start mariadb.service``
并设置开机自启动``sudo systemctl enable mariadb.service``

![启动并设置自启动](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2032_zps3lvn9s7m.png)

数据库安全性设置，推荐全部选择``Y``

![secure installation](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2033_zpsxaka2v9w.png)

至此，数据库的问题解决了，可以再次执行``setup-seafile-mysql.sh``脚本，顺利完成

- 启动Seafile服务

**按照官方文档**，运行``seafile.sh``脚本及``seahub.sh``脚本，启动seafile及seahub服务，过程中需要设置邮箱密码等账户信息，关于seafile和seahub，可以理解为：seafile是文件服务的后台，seahub是服务的前端，在服务器端8000端口可以访问seahub页面，从而查看到文件信息

![启动seafile及seahub服务](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2036_zpsyfr06vec.png)

最后，我们需要从外部访问到seafile和seahub服务，因此我们需要为seafile和seahub的端口设置防火墙为public

![设置防火墙](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2037_zpsylqwwgip.png)

至此，服务器Seafile服务配置启动完毕，我们拥有了自己的私有云盘

## 从局域网内访问私有云盘

还记得我们之前已经将服务器的``192.168.99.101:8000``转发到了宿主机的``localhost:9988``，那么我们现在直接访问``localhost:9988``就可以登录到seahub网页，邮箱密码就是前面设置的那个

![登陆seahub](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2038_zpsfhzkax10.png)

登陆后，我们上传一张图片作为实验，成功

![成功上传图片](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2039_zpsnos7qcxr.png)

Bingo!!!

现在只要在我们局域网内的任何人只要用这个账密登入，就可以同步和共享文件了

理论上，如果我们不是在VirtualBox里面，而是在真实的线上的一台服务器上面做我们上面这些操作，现在已经可以通过服务器的IP地址来完成从互联网到私有云盘的访问了，我们的私有云盘项目就搞定了

但是，既然我们在VirtualBox里面做的，那么就把这个项目做到底，让她能够被公网访问

- 从公网访问VirtualBox内的虚拟机服务器上的私有云盘

这里我们需要利用一个神器[ngrok](https://ngrok.com/)，这里我要实现的只是将私有云盘能够从公网访问，爽一把，所以只需要使用最基础的ngrok的端口映射功能就好了，这个神器还是第一次开发微信公众号后台的时候认识的

ngrok的使用很简单，下载，运行，运行方式如下，如果提示``ngrok``命令为发现，那么将ngrok移入Mac的bin目录或者制作软链接到bin目录就好

使用``ngrok http 9988``将``9988``端口映射到随机公网地址(想不随机要花钱，暂时穷)

![ngrok映射端口到公网](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2040_zps46maujd7.png)

图上``http://....ngrok.io``就是现在``9988``端口映射到公网后的地址，也就是说，从这个在公网可以访问的地址，访问时，会访问到宿主机的``localhost:9988``，而记得我们之前已经将``localhost:9988``与虚拟机服务器的``192.168.99.101:8000``绑定，那么就达到了从公网直接穿透过来访问我们的VirtubalBox上面服务的目的

最后一张图，使用手机访问ngrok提供的地址，成功访问到了自己在VirtualBox上的私有云服务，注意图片左上角，使用的是手机网络而非宿主机Mac所连接的WiFi，同时地址栏也能看到访问的地址是ngrok给映射的地址

![公网访问VirtualBox私有云](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2041_zps80uubho8.jpeg)











