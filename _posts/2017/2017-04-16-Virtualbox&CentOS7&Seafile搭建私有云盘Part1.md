---
layout: post #post
title: VirtualBox & CentOS7 & SSH & Seafile & ngrok 搭建私有云盘Part1 #post title
categories: "2017" # archive
tags: VirtualBox CentOS7 SSH Seafile ngrok #post tag, seperated by space
---

Build your own private cloud disk using VirtualBox & CentOS7 & SSH & Seafile & ngrok Part1

这个是个人的一篇经验总结，或者说花样踩坑集锦

先说下最终完成的效果：宿主机为Mac，在VirtualBox中安装CentOS7 minimal做为服务器，通过SSH由宿主机连接到CentOS7并在其中构建Seafile服务，通过外网或内网访问Seafile服务以完成同步或共享文件的功能，达到拥有私有云盘的目的

## 在VirtualBox中配置用于安装CentOS7的虚拟机

[VirtualBox](https://www.virtualbox.org/wiki/VirtualBox)是一款Mac上很好用的虚拟机软件，之所以选择VirtualBox而不是其他的虚拟机软件，是因为我司项目开发就用的VirtualBox，所以我最初做这个项目的目的之一就是用来练手加强对VirtualBox的理解的

[CentOS7](https://www.centos.org/)是RedHat之下的一款社区版Linux，目前最高级版本为7， 可以在[这里](https://www.centos.org/download/)下载。在下载页面会看到有``DVD ISO``，``Everything ISO``，``Minimal ISO``三个选项，一般来说，如果需要GUI页面，选择``DVD ISO``，如果只需要最小安装尤其是作为服务器操作系统时，选择``Minimal ISO``，而``Everything ISO``对一般用户来说很少选择，其包含了以上两种ISO文件所有的内容，并额外包括开发CentOS功能所需要的内容

下面开始比较通用的在VirtualBox中安装虚拟机的步骤，这个步骤理论上是适合于创建任何系统虚拟机的，毕竟只是打造一台"裸机"

- 命名虚拟机及指定操作系统，命名随意，Type选Linux，Version选Red Hat

![命名虚拟机及指定操作系统](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot_zpsecmyufzi.png)

- 指定虚拟机内存空间，随意更改，我采用了默认设置

![指定虚拟机内存空间](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot_zpsecmyufzi.png)

- 虚拟硬盘文件格式，随意更改，我采用了默认设置

![硬盘文件格式](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%203_zpsx5c10u1t.png)

- 虚拟硬盘文件存储位置及大小，随意更改，我采用了默认设置

![虚拟硬盘文件存储位置及大小](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%204_zps5jzzeqdi.png)

- 虚拟硬盘文件是否可动态增长，随意更改，我采用了默认设置

![虚拟硬盘文件是否可动态增长](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%205_zpskvyylbnt.png)

- 裸机搞定

![裸机搞定](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%206_zps0arqwzpz.png)

## 在虚拟机中安装CentOS7操作系统

- 载入操作系统镜像 CentOS7 Minimal ISO

启动刚刚设置成功的裸机，会要求安装系统，在系统文件加载的地方指定到之前下载的CentOS7 Minimal ISO后点击start即可

![安装系统](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%207_zps06iiysmz.png)

进入CentOS系统安装页面后，鼠标被VirtualBox的虚拟机窗口捕获后会提示是否允许捕获，当然选是，然后虚拟机中按``cmd + 鼠标左键``的方式可以退出捕获，让鼠标回归宿主机

- 安装CentOS7

进入安装选择界面，键盘方向键上下调节选项，``Enter``键选中选项，这里选择最简单的安装方式即可，即执行``Install CentOS Linux 7``选项

![Install CentOS Linux 7](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%208_zps4gpb02kt.png)

**安装时需要注意，不要随便调节窗口大小，一般VirtualBox会自动缩放窗口以让用户能最明显和方便的点击各个按钮，如果你像我一样手贱的缩放了窗口，那么记得上下左右滚动窗口以找到各种下一步的按钮，不要像我一样轻易以为VirtualBox傻缺**

选择系统语言，推荐英语，没啥可说的

![选择语言](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%209_zpsgihrcns6.png)

系统安装设定，这个我们不需用在CentOS安装时设定，因为我们之前在裸机设置时已经搞定了，这里点开这个设定窗口后直接左上角``done``即可

![系统安装设定-1](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2010_zpsd2omagbz.png)

![系统安装设定-2](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2011_zps6ixumbux.png)

点击``Begin Installation``即可，喝水

![开始安装](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2012_zps5xlief58.png)

开始安装后，在读条的过程中，会发现有两个设定需要完成，这里不用着急，着急也没用，进度图跑完前没法点击这两个按钮，安心喝水

![读条](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2013_zps37bj6soz.png)

读条结束后，开始设置两个黄色感叹号内容

root用户的密码设定，很重要，不用我多说

![root密码](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2014_zpsol2pfvvh.png)

注意看图，知识点又来了，设置的root密码强度差不说，还居然包含了root用户名，被系统吐槽，而且这样的密码需要点两次左上角的``Done``才能设置成功

这里主要是在本地虚拟环境，不想设置太复杂的密码，会忘 -_-#

接下来设置一个登陆用户，在你设置完root密码后，这个设定已经变成可选了，但是还是推荐在这里设置好

![设置登录用户](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2015_zpsq4rt97rc.png)

我这里直接给自己的登陆用户作为管理员了，以后的操作都会使用该用户<strike>为了加深印象我又把密码设置的很差劲让系统提醒一次 [doge] </strike>

OK，这里设置完毕后，左上角``Done``*2，系统就开始启动了，在VirtualBox中安装CentOS7的操作就完成了

## 通过SSH由Mac宿主机访问CentOS7虚拟机

- 登陆虚拟机，查看网络配置

在CentOS7启动后，会停留在``localhost login: ``处等待，这里使用管理员``kenshin``的账密登入

![登入系统](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2015-1_zpsfhfnflbj.png)

图中重点，``ip addr``查看CentOS7的网络配置，注意``2: enp0s3:``，这里指示的是虚拟机的internet连接，能够发现，没有ip地址

这是因为CentOS7的网络服务并不是默认开机启动的，需要手动配置

首先进入到CentOS7的网络配置文件所在目录

```
cd /etc/sysconfig/network-scripts/
```

![配置网络-1](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2016-1_zpsqves4lcb.png)

然后打开``ifcfg-enp0s3``文件，这里需要``sudo``权限来编辑该文件

```
sudo vi ifcfg-enp0s3
```

![打开ifcfg-enp0s3](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2016-2_zpsidzamajs.png)

将``ONBOOT``设置为``yes``后``wq``保存并退出

![ONBOOT](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2016-3_zps9tnq5rud.png)

- 关闭虚拟机，配置网卡

经过上面的操作，理论上是只需要重启网络服务就可以了，但是我们需要退出来为虚拟机额外添加网卡，所以索性先关机

而更实际上，我们在创建裸机的时候就可以添加好这个额外的网卡的，但是我忘了

其实真实情况是，这里我浪费了很多时间来处理网卡的事情，最初我并没有想到VirtualBox默认的NAT网卡不能完成我后面需要的SSH连接的功能，后来还搞了一阵Bridged Adapter的作用，所以这里是已经折腾了很多东西之后才发现的事情，但是既然是记录步骤，就不说那些错误和解决错误的详情了，最后再统一总结下就好了

另外，关于NAT，Host-only，Bridged的Adapter的区别，后面会提到

总之，现在把虚拟机关机，回到VirtualBox界面后，为虚拟机添加一个Host-only Adapter

![添加Host-only Adapter](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2017_zpszf0m00r6.png)

- 启动虚拟机，再次查看网络信息

网卡添加完毕后，启动虚拟机，登入管理员账户，执行``ip addr``命令查看网络信息

![新的网络信息](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2018_zpsmk3dmykk.png)

重点来了，图上我们有了两个internet的ip地址

图中``2: enp0s3 .... inet 10.0.2.15``是由NAT Adapter提供的，其可以ping通外界网络，也就是外界网络对其来说可见，但是外界不能ping通``10.0.2.15``，也就是其对外界网络来说不可见，**包括宿主机也不能ping通 10.0.2.15**

图中``3: enp0s8 ... inet 192.168.99.101``是由Host-onlyAdapter提供的，其可以ping通外界网络，也就是外界网络对其来说可见，但是外界不能直接ping通``192.168.99.101``，也就是其对外界网络来说不可见，**但是宿主机可以ping通 192.168.99.101**

这就是为什么需要额外添加一张Host-onlyAdapter的原因，如果宿主机都不能ping通虚拟机，那么SSH毛毛啊

- 宿主机SSH连接虚拟机

在虚拟机中执行``service sshd start``命令来开启SSH服务，默认会运行在22端口

在虚拟机中执行``systemctl status sshd``命令来查看SSH服务状态

![SSH服务状态](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2018-1_zpscik19mco.png)

``Loaded: ...enabled...``表示SSH服务开启了开机自启动，这正是我们需要的

``Active: active(running)...``表示SSH服务当前正在运行

现在，想建立SSH连接就只剩下最后一步，设置端口

在NAT Adatper中配置端口转发信息，为什么不在Host-only Adapter中配置呢，因为不能

**Tips, 配置端口转发不需要重启虚拟机的**

打开虚拟机的网络配置界面，在NAT Adapter中选择``Port Forwarding``

![打开NAT Adapter](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2019-0_zpspwegr3jn.png)

``Port Forwarding``配置端口转发信息

![端口转发](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2019_zpskq2kaya7.png)

这里做了两个端口转发

``Guest Port 22``是用来做SSH连接的，默认情况下，服务器22端口用于SSH，``Guest IP``和``Host IP``都正常设置，``Host Port``随意，不要冲突了就好

``Guest Port 8000``是用来做服务转发的，后面会用到，我会把虚拟机8000端口的服务转发到宿主机，然后宿主机可以用来搞点事情，这里``Guest IP``和``Host IP``也是正常设置，``Host Port``同样随意，不冲突就行

保存信息之后，在CentOS7中使用``sudo systemctl restart network``命令重启下网络服务或者直接``reboot``命令重启机器

在宿主机SSH连接虚拟机，``-p``指定端口，后面再跟要登入的用户及服务器地址，我们已经做了端口和地址转发，因此这里的命令如下

![SSH连接](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2017-04-16-post01/screenshot%2020_zpsqgtymite.png)

Bingo!!!

搞定，我们现在可以在Mac宿主机终端经由SSH操作CentOS7虚拟机了，也就是我们的服务器（下面我也会除非万不得已，不然均称呼我们的CentOS7君为服务器），接下来我们就开始在服务器配置Seafile，不过现在内容太多了，尤其是很多图片，所以写在第二部分














