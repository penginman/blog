---
title: jsDelivr加速静态博客
categories: ['瞎折腾']
tags: ['jsdelivr']
cover: 'https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/cover/20201111213620.jpg'
date: 2020-11-11 21:37:53
---


## 前言

​	这几天总感觉博客访问特别慢，最先是找到了CDN加速，但是在国内加速的话域名都是要备案的，又看了看境外加速。

>  CDN的全称是Content Delivery Network，即内容分发网络。CDN是构建在网络之上的内容分发网络，依靠部署在各地的边缘服务器，通过中心平台的负载均衡、内容分发、调度等功能模块，使用户就近获取所需内容，降低网络拥塞，提高用户访问响应速度和命中率。CDN的关键技术主要有内容存储和分发技术。——百度百科

在犹豫要不要买的时候，发现了这个东西：[jsdelivr](https://www.jsdelivr.com/)，一个可以加速静态资源的免费CDN，官网上能看的出和WordPress有什么py关系还可以加速github的资源。hexo是静态博客，那么我就把博客要用到的js、css、还有博客用到的一些图片都放github然后引用。开搞

## 1. 新建仓库



![](https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed/artical/20201111202508.png)

​	名字重了是因为我已经创建好了并且使用了以后才来写的博客。

## 2. 克隆Git仓库到本地

​	在自己电脑建个文件夹然后打开git输入`git clone 你仓库的链接`，把刚刚创建的仓库拉倒本地方便上传到仓库。

![](https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed/artical/20201111203733.png)

廖雪峰老师的git教程我当时看了一遍，觉得非常棒，哈哈哈就是自己太菜了又给忘了，帖出来：[Git简介](https://www.liaoxuefeng.com/wiki/896043488029600/896067008724000)。

## 3. 上传需要加速的资源

​	把需要上传的资源整理到刚刚拉下来的本地git仓库，上传。

```
git status                    //查看状态
git add .                     //添加所有文件到暂存区
git commit -m '第一次提交'      //把文件提交到仓库
git push                      //推送至远程仓库
```



​	这里我说一下是怎么加速自己的博客的，因为博客加载的时候需要加载主题的各种js和css文件，然后因为服务器网渣所以加载时间很慢，使用加速的话就会加载的快。

​	接下来是要上传哪些文件，我使用的是`butterfly`这个主题，[主题github](https://github.com/jerryc127/hexo-theme-butterfly)支持一下作者，直接在主题`theme/butterfly`文件夹下面找到资源文件夹`source`发现里面都是一些零碎的文件，但是在发布文件夹`public`下是一个完整的js和css，所以猜测生成的时候会把零碎的文件进行整合，然后主题配置文件里作者也写的很清楚

![](https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed/artical/20201111205045.png)	穷人流下了不争气的泪。传！(真加速还得选好服务器)

​	这里我的分析是：由于引用的不是本地的资源文件，所以可能会产生自己在本地修改了某项配置，但是网页没有生效，这里就需要时刻记着自己引用的是github上的资源，如果本地配置大改的话，github上的文件也要进行重新上传覆盖。

​	做法：配置文件里找到引用的是本地资源的项，然后在生成网站的`public`文件下找到对应的资源文件。

我列一下我在配置文件里修改的项：`main_css`、`main`、`utils`、`local_search`、`algolia_js`、`translate`，因为使用的是`Valine`评论，里面可以设置自定义表情，我也使用这个方法修改了。



## 3. 获取地址

​	官网首页很清楚的写明了如何获取资源链接

![](https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/artical/20201111211357.png)

​	`https://cdn.jsdelivr.net/gh/user/repo@version/file`，`user`就是你的github用户名，`repo@version`，仓库加上版本号，`file`就是仓库下的路径。

​	这里我没有说版本号是因为网上的教程讲到了仓库需要发布，但是我后面无意间发现不用发布直接`reop@分支名`，也可以访问到。并且我一开始也发布仓库了，但是后面想要修改已经上传的文件也出了一些问题，索性直接用简单的。

​	贴一个我博客首页的壁纸链接：https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/top_img/83531406_p0.png

​	这个壁纸有12M大小，而且链接也符合上面的格式，可以参考一下。



## 4. 引用链接

​	配置文件里可以找需要替换的资源，直接贴上链接就可以了，只不过以后别忘了**你现在引用的是之前的上传的静态资源**，别忘啦！别忘啦！别忘啦！

​	我发现github能这样用以后就在上面整了图床，现在博客里的图片都开始在上传，之前用的路过图床，说的全球都有CDN加速，但是还是卡的一。

​	还有我整理的`Valine`评论的自定义表情，大伙可以直接拿去用：[图片地址](https://cdn.jsdelivr.net/gh/guobang-yoo/CDN@master/emoji/)，[emojimap](https://cdn.jsdelivr.net/gh/guobang-yoo/CDN@master/emoji/valine.json)。完工



