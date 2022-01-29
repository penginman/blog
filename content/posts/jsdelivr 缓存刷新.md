---
title: jsdelivr 缓存刷新
date: 2020-11-20 22:02:46
categories: ['瞎折腾']
tags: ['jsdelivr']
cover: https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/cover/20201116160039.png
---

## 前言

​	上一篇修改了黑幕，但是使用了jsdelivr加速的静态资源，所以照常更新下github上的资源，但是github上查看已经上传成功了，jsdelivr访问的依然是之前的资源，说白了就是缓存的问题。即使本地浏览器端的缓存已经清理，也会因为CDN周围的节点没有同步数据而导致用户端未能及时更新。

## 缓存刷新

把原来访问的链接

`https://cdn.jsdelivr.net/...`

改为

`https://purge.jsdelivr.net/...`

访问资源就会进行刷新，然后页面会返回刷新信息：

![](https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/artical/20201120222419.png)

<span class='heimu'>划水</span>收工。