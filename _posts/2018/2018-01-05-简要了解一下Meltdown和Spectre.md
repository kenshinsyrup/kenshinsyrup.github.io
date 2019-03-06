---
layout: post #post
title: 简要了解一下Meltdown和Spectre #post title
categories: "2018" #post archive
tags: Meltdown Spectre #post tag, seperated by space
---


凑个热闹，简单了解一下[Meltdown and Spectre](https://meltdownattack.com/)

作为非专业人士，就本着看热闹不嫌事儿大的态度Google看看好了，以下内容本质上都是来自搜索引擎，我自己理解或者翻译了一下

## 总体介绍
Meltdown和Spectre是两种不同的bug，但都可以被攻击者用来在现代电脑处理器上非法窃取密码等隐私数据。这里的现代电脑处理器目前被定义为来近20年来生产的所有处理器，包括Intel，AMD，以及基于ARM架构的处理器

因为是比较严重的bug，当然作为处理器生产厂家第一反应就是站出来稳定军心，当然牙膏厂没啥好说的来，毕竟全线崩溃，Meltdown和Spectre都对其有影响，于是先站出来放了个大炸弹说不止Intel有这个危险，所有的处理器都有。于是AMD不开心了，站出来回怼说AMD是不会收到Meltdown影响，同时'near zero risk'被Spectre影响。这就是新闻热闹而众说纷纭的原因

#### Meltdown
![Meltdown.png](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2018-01-05/meltdown_zps0qkfzfnc.png)

Meltdown会利用Intel处理器的**预测执行**设计，破坏应用程序和操作系统间的界限，攻击程序有机会访问到操作系统所使用的内存空间，也就有机会从中获取操作系统级别的数据

但是这个bug是可以进行补丁修复的，而且各方已经开始放出软件进行修复。虽然修复后对系统性能会有影响，但是具体有多大影响暂时不明，听说对个人设备来说使用的影响不大，看了几个测评，打完补丁前后玩瞎子信条以及跑大型Adobe或者Office软件都没有很大差别

#### Spectre
![Spectre.png](http://i32.photobucket.com/albums/d1/kenshinsyrup/Kenshinsyrup/2018-01-05/spectre_zpsebzeyy5i.png)

不得不说这Logo还挺萌。。。

Spectre会破坏应用程序之间的隔离性，攻击程序有机会访问到其他应用程序所使用的内存空间，也就有机会获取其他应用程序的数据

Spectre相对于Meltdown更难以被利用，但是也更难以被通过软件补丁的方式修复，现在普遍认为会在接下来很长时间内作为一个随时可能引爆的炸弹存在

重点，Spectre目前有两种变体，AMD官方宣称的对Spectre的'near zero risk'是指的Spectre的一种变体而非全部，这也是在Hacker News上面被嘲讽的一个点

## 危害群体
其实这两个bug对普通用户对影响不大，安装下各自厂商发布的系统升级，继续安全的使用自己的设备就好了。因为虽然这两个bug在个人设备上也可以被利用，但是个人设备一般都是单人使用，也有比较迅速的系统更新和权限系统管理，以及杀毒软件等，危害不会很大

重大危害主要针对的是云服务平台，云平台理论上可以视作一台巨大的电脑，同时有无数的用户在自己的实例上进行操作，云平台发展至今，无论个人还是公司或是政府都有自己的内容在上面，其上存储的隐私数据的重要程度大家都清楚，所以一旦发生用户隐私数据泄露，影响要严重的多，而且攻击者完全可以是一个正当的云平台用户，在掌握了bug的利用方式之后，其只需要在该平台上运行攻击程序就可以窃取其他用户甚至操作系统级别信息的

## 利用方式

新闻说完了，Meltdown和Spectre这两个bug是可以利用的，这个是**概念验证**已经通过的

利用的思路如下:
> The core of this attack is extremely simple and elegant; use a value from a speculated-but-not-completed instruction as an index, causing one of a set of cachelines to be touched; then use a timing measurement on those cachelines to see which one was accessed, thus determining the value.

作为非专业人员，我觉得用形象生动的例子来理解一下这些高大上的内容的原理就可以了:)

所以我们举个例子

当然先贴来源[[Reading privileged memory with a side-channel](https://googleprojectzero.blogspot.com/2018/01/reading-privileged-memory-with-side.html)
](https://news.ycombinator.com/item?id=16065845)

想象有一家图书馆，里面的书都只有一个字母

现在图书馆有一些书你无权借出比如BookX。有一些书你有权借出比如BookY，BookY+1，。。。以此类推

你来到图书馆前台，向图书馆管理员说你想借出BookX，但图书馆管理员不知道你有没有权利借出BookX，于是让查询员帮自己去系统里查一下你是否有权利借出这本书。在你们等待查询员回复的时候，你说“实际上，我并不大感兴趣借出BookX，我实际想做的事情是：如果BookX里的内容是字母'a'，我就借出BookY；如果BookX里的内容是字母'b'，我想借出BookY+1。。。以此类推”

图书馆管理员现在正在等待查询员回复你是否有权利借出BookX，但是闲着也是闲着，所以他就翻开BookX看了一下，然后发现里面的内容是字母'b'，于是就去书架上去了BookY+1到前台来，以便一旦查询员回复说你有权利借出BookX，那么图书馆管理员可以立刻把BookY+1给你，以提高效率（**Intel预测执行**）

然后，查询员回来了，告诉图书馆管理员说你无权利借出BookX，于是图书馆管理员对你说“对不起，你不能借出BookX。（然后我也不是傻子）我也不能把我刚刚拿到的那本你有权利借出的书给你因为如果这样的话你就反推出来了你无权借出的BookX的内容。”

你当然表示理解，然后说你想借出BookY，图书管理员去书架**找了一会儿**，比如花了5分钟，然后把BookY拿给了你。然后你说你还想借出BookY+1，图书馆管理员**立刻**把手头的BookY+1拿给了你。然后你说你还想借出BookY+2，图书管理员去书架**找了一会儿**，比如花了5分钟，然后把BookY+2拿给了你

于是，通过时间差，你推断出，BookY+1极大可能是在你想借出你无权借出的BookX时，图书馆管理员拿到手里的那本书，也就推断出BookX的内容是字母'b'

完工，BookX就是系统进程内存或者其他程序所在内存，其中的内容就是内存中的内容，图书馆管理员就是处理器，你就是攻击程序，图书馆管理员的手头或者说前台就是缓存

最后，如果BookX指的是系统进程所在内存，系统有乱序映射的处理来让其他应用不能轻易的知道自己的内存地址，但是各种应用程序所在的内存地址可能就没那么难获得了:)


