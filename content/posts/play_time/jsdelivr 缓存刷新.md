---
title: jsdelivr 缓存刷新
date: 2020-11-20 22:02:46
categories: ["Play"]
tags: ["jsdelivr"]
---

## 前言

​ 上一篇修改了黑幕，但是使用了 jsdelivr 加速的静态资源，所以照常更新下 github 上的资源，但是 github 上查看已经上传成功了，jsdelivr 访问的依然是之前的资源，说白了就是缓存的问题。即使本地浏览器端的缓存已经清理，也会因为 CDN 周围的节点没有同步数据而导致用户端未能及时更新。

## 缓存刷新

把原来访问的链接

`https://cdn.jsdelivr.net/...`

改为

`https://purge.jsdelivr.net/...`

访问资源就会进行刷新，然后页面会返回刷新信息：

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201120222419.png)

划水收工。
