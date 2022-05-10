---
title: "获取京东CK教程"
date: 2022-05-09T21:40:49+08:00
categories: ['瞎折腾']
tags: ['京东薅羊毛']
draft: false
---

## 获取京东CK

获取京东CK(cookie)并提交到上车面板。教程分为三部分分别是，PC端获取CK，接受一对一薅羊毛通知，最后是安装APP获取CK。

> cookie的格式为：pt_key=XXX; pt_pin=XXX;
>
> XXX为占位符

CK提交地址：[http://42.192.83.222:2332](http://42.192.83.222:2332)

## PC端获取CK

登陆地址：[多快好省，购物上京东！](http://plogin.m.jd.com/login/login)

> 首先告知。
>
> 如果你有多个号登陆，请使用浏览器的无痕浏览，或是更换浏览器抓取，因为京东的cookie在登出以后就会失效。
>
> 使用无痕浏览登陆抓完直接关闭，再打开新的无痕浏览抓下一个。
>
> 京东cookie手机登陆方式有效期为一个月左右。

![](https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/artical/16521042625541652104262122.png)

输入手机号，获取验证码登陆。如果是手势验证码，失败很多次别灰心，~~手稳一点~~，是可以验证通过的。

![](https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/artical/1652104507214QQ%E5%9B%BE%E7%89%8720220509215454.png)

登陆成功以后

按F12打开开发者工具选择**Network**（网络），请注意要拦截所有的请求（只要能找到cookie就可），在下方**Name**（名称）里随便选一个找有完整cookie的。在右边**header**（标头）下的cookie中找到如`pt_key=XXX; pt_pin=XXX;`格式的就是需要的cookie。

> **如果你的名称列表里面是空的**，随便点一个页面中的其他按钮，浏览器就会捕获请求

![](https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/artical/1652105174078QQ%E5%9B%BE%E7%89%8720220509220548.png)

然后在[http://42.192.83.222:2332](http://42.192.83.222:2332)中提交CK，服务器建议`腾讯云`，变量组为`JD_COOKIE`，备注可以选择下方的[一对一通知](#额外服务(一对一通知))方式，CK失效的时候我会根据这个通知你，或者你自己记得一个月要来提交一次。

![](https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/artical/16521053935521652105392592.png)

完工。

补充：如果不想在一堆cookie里找需要的那两个，可以把cookie完全复制下来。然后按F12开发者工具的控制台（console）中执行以下代码：

```js
var CV = '这里面放你拿到的完整cookie';
var CookieValue = CV.match(/pt_key=.+?;/) + CV.match(/pt_pin=.+?;/);
console.log(CookieValue)
copy(CookieValue);
```

控制台里有输出就算成功了，会直接复制到剪切板。

如果看不懂我说的，看看这篇文章方法更全：[https://w37fhy.cn/2379.html](https://w37fhy.cn/2379.html)

## 额外服务(一对一通知)

可以每天看自己的资产，有商品领取提醒，cookie失效的时候可以及时收到通知。

以下内容使用微信打开，或者复制到浏览器打开（虽然是二维码，但是会动态刷新，所以自行打开）

[//wxpusher.zjiecode.com/api/qrcode/cLY05MLHiHj7JXD7sXMjISKV5C85yhgGDupknmwqFevgvPGzS1xpiiRj0LCX8Xtj.jpg](//wxpusher.zjiecode.com/api/qrcode/cLY05MLHiHj7JXD7sXMjISKV5C85yhgGDupknmwqFevgvPGzS1xpiiRj0LCX8Xtj.jpg)

关注微信公众号

![](https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/artical/1652154472985E}{9OJS7T2454WO88}[9KK6.png)

扫码关注以后，提示已经**订阅应用京东**，然后点击下面我的--->我的UID，会返回你唯一的UID，我只能通过这个UID来通知你你的账号信息。请提交的时候附上备注，格式如`你的备注@@你的UID`。

## 手机端获取CK(Android)

下载apk：[https://www.aliyundrive.com/s/rbjgkpBY9eb](https://www.aliyundrive.com/s/rbjgkpBY9eb)

打开以后直接登陆，登陆完成以后会直接弹出抓取结果，然后复制CK提交到网页[http://42.192.83.222:2332](http://42.192.83.222:2332)中即可。

![](https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/artical/1652107556707QQ%E6%88%AA%E5%9B%BE20220509224405.png)

完工。

>  如果觉得我说的很啰嗦就是因为最近在写毕业论文，废话文学了属于是。

