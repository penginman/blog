---
title: Chevereto-搭建一个自己的图床
date: 2020-09-20 11:09:02
categories: ['瞎折腾']
tags: ['图床']
cover: https://cdn.jsdelivr.net/gh/penginman/PicBed@master/cover/20201111120851.jpg
---

​	博客搭完了，自己选择的这个博客主题又是以图片为主要元素的，当然要考虑图片的使用问题了，原来打算直接放在服务器上，但是后来想了想，以后如果文章~~越写越多~~用的图太多服务器的流量也不够用，想到了图床这一托管图片的服务，但是放在别人的上面总怕会受到~~限制~~，正好发现了`Chevereto`这一图床系统，可以自己搭建一个图床服务器，正好自己还有一个阿里云学生云，那就开工。

### Chevereto 说明

>Chevereto是一个可以帮助你建立自己的图像分享网站（图床）的应用程序，我们的目标是可以让世界上的任何一个人都可以建立自己的图像共享平台。我们坚定不移的为那些想要可定制的白标图像共享服务的人建立一个真正的替代品。

`Chevereto`分为免费版和付费版，区别肯定就是大小什么的，我这个搭在学生云上肯定就不用想我会选择哪个了吧🤣

### 环境说明

因为`Chevereto`所需要的环境为

* Apache/Nginx

* PHP 5.5+

* MySQL 5.0+

~~我太懒了不想动~~所以我选择使用宝塔面板为我们搭建web环境。

宝塔面板官网：https://www.bt.cn/

安装完成以后进入管理面板应该会直接提示你安装需要的环境

我的环境如下图

![BJ9TVP.png](https://s1.ax1x.com/2020/10/29/BJ9TVP.png)

### 总结安装步骤

1. 去github下载Chevereto的压缩包。
2. 在宝塔面板上新建网站目录，创建一个存图片的数据库(自行创建)。
3. 将Chevereto的压缩包上传到网站目录解压。
4. 访问新建的网站就是Chevereto的管理面板，并进行初始配置。
5. 无了。

### 开始

* [github下载地址](https://github.com/Chevereto/Chevereto-Free/releases)

* 创建网站目录和数据库用户

![BJ9H58.png](https://s1.ax1x.com/2020/10/29/BJ9H58.png)

因为我域名所以域名留空，提示默认使用`80`端口，访问地址就是服务器ip地址，剩下的自己随机发挥。

* 创建完成后需要配置一下网站配置文件才可以访问到配置页面。

![BJ9Ibt.png](https://s1.ax1x.com/2020/10/29/BJ9Ibt.png)

![BJ9qPS.png](https://s1.ax1x.com/2020/10/29/BJ9qPS.png)

在`server{...}`中添加

```nginx
location / {
try_files $uri $uri/ /index.php?$query_string;
}
```

配置完成以后应该会自动保存并重启`Nginx`。

* 将在github上下载的`Chevereto`压缩包上传到刚刚创建的网站目录中（上图是`/www/wwwroot`）并解压。

之后就可以直接访问`服务器ip:80`（80端口可以省略），然后一步一步的进行配置。

**可能会出现的错误**

> `Chevereto can’t create the app/settings.php file. You must manually create this file`

解决方法：这个错误就是没有找到`setting.php`配置文件，压缩包内似乎没有创建该文件，我们可以自行创建，在`Chevereto`的网站目录下的`/app`目录下执行命令创建文件，并修改文件权限

```shell
touch settings.php
chmod +x settings.php
```

> 我自己还遇到了第二个错误，大概的意思就是访问权限不足blahbalhblahbla，我改了好久都不行。最后直接把整个网站目录的权限给改了访问成功。知道这样做不对，希望大佬能指点。

* 访问网站进行网站的初始化配置，大概就是填写数据库名称、数据库账号密码、管理员的账户和密码和一些信息。


![BJ97Uf.jpg](https://s1.ax1x.com/2020/10/29/BJ97Uf.jpg)

![BJ9O2Q.jpg](https://s1.ax1x.com/2020/10/29/BJ9O2Q.jpg)

完成以后就可以登陆管理员账号进入管理面板，我是首先去设置里找到语言设置把面板改成了中文。

### 完工

管理面板还有好多其他功能，我都还没研究过，大伙可以以后可以自己慢慢学习

然后我的图床地址：http://47.97.231.10/ (已失效)    ~~🈚👇👻来丶se兔~~

