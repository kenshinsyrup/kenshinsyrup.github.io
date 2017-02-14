---
layout: post #post
title: Hello World! #post title
categories: Other #post category, seperated by spcace
tags: Other #post tag, seperated by spcace
---

## 博客页面搭建完毕

最近<del>又</del>比较浮躁，刚好发现自己去年测试Jekyll时候搭建的Github个人页面不怎么好看，于是想重新弄一个，以后也算有个地方写东西

期间<del>又</del>踩到了坑，这里随便记录下

使用Jekyll搭建Github个人页面的帖子满世界都是，肯定不用我重复了，毕竟我也是菜

OK，直接说遇到的问题

### Could not locate Gemfile or .bundle/ directory

这个问题的来历比较神奇

首先，我前端内容并不是特别熟悉，所以博客模版也是直接从[Jekyll Themes](http://jekyllthemes.org/)找的，这里我使用的是[Codinfox Lanyon
](http://jekyllthemes.org/themes/codinfox-lanyon/)模版，其源码仓库地址是[这里](https://github.com/codinfox/codinfox-lanyon)

由于种种原因，我没有直接fork上面的仓库，而是donwload之后自行修改成自己喜欢的样子，改完之后push到了我自己的Github[仓库](https://github.com/kenshinsyrup/kenshinsyrup.github.io)

但是，转折来了，在我把模版改成喜欢的样子之后，本地试图查看效果时，``bundle exec jekyll serve``命令提示错误``Could not locate Gemfile or .bundle/ directory``

Google了半天，遇到了很多同样有该错误的人，但各种解决方案都不适合我这里的情况。最后终于发现，原因是模版代码里没有``Gemfile``和``Gemfile.lock``两个文件ORZ

解决办法就是自行创建这两个文件，当然最简单的就是让Jekyll给我创建了，执行``jekyll new myblog``，创建一个全新的博客，然后在``myblog``里面把``Gemfile``和``Gemfile.lock``文件拷贝到我自己的模版根目录

博客模版使用到了``jekyll-paginate``插件，因此手动修改默认的``Gemfile``文件plugins相关内容，增加一行``gem "jekyll-paginate"``，如下:
```
# If you have any plugins, put them here!
group :jekyll_plugins do
   gem "jekyll-feed", "~> 0.6"
   gem "jekyll-paginate"
end
```

那么，就祝所有人新年大吉<del>吧</del>!