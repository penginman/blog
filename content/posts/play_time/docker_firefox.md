---
title: Linux上运行Firefox浏览器
categories: 'Play'
slug: docker-firefox-build
date: 2025-10-13
description: '在Linux服务器上通过Docker运行一个浏览器'
---

## 前言

有一个网页游戏挂机的需求，遂找到了这个东西。项目地址：https://github.com/jlesage/docker-firefox

## 一键启动

```
docker run -d \
    --name=firefox \
    -p 5800:5800 \
    -v $PWD/firefox:/config:rw \
    jlesage/firefox
```

然后访问 `http://ip:port` 即可。

## 其他问题

发现中文网页会乱码。安装中文字体即可

```
docker exec -it firefox sh
wget https://dl-cdn.alpinelinux.org/alpine/v3.19/community/x86_64/font-wqy-zenhei-0.9.45-r3.apk
apk add font-wqy-zenhei-0.9.45-r3.apk
```

水一篇，后续有需求改配置了再来。