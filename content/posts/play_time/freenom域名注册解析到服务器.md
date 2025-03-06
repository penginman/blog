---
title: Freenom域名注册解析到服务器
date: 2020-09-02 12:25:37
categories: ['瞎折腾']
tag: ['免费域名']
---

### **1.官网**

[Freenom - 人人都熟悉的名字](https://www.freenom.com/zh/index.html?lang=zh)

官网右上角可以切换中文，简直了。

然后觉得我讲的有点啰嗦的还可以看油管上的[freenom教学视频](https://www.youtube.com/watch?v=IAZDrtmQeh0)

![BJ9Xvj.png](https://s1.ax1x.com/2020/10/29/BJ9Xvj.png)

### **2.搜索想要的域名**

![BJCFGF.png](https://s1.ax1x.com/2020/10/29/BJCFGF.png)

​	搜索以后会列出来可以选择的域名列表，上面的是免费域名，下面的则是一些付费域名。

​	我在这里还遇到了一个坑提前说。freenom提供的有自己的域名解析服务，国内也可以访问的到，我遇到了一个问题有回答说换个DNS解析就行，推荐的是[Hurricane Electric Hosted *DNS*](https://dns.he.net/)，但这个网站禁止了.tk域名的解析。类似的问题请自行先考虑

![BJ9xrn.png](https://s1.ax1x.com/2020/10/29/BJ9xrn.png)接下来选中域名以后点击Get it now！以后只是添加到购物车，随后点击界面中的Checkout跳转到结算界面，这里只用选择期限即可。

![BJ9vKs.png](https://s1.ax1x.com/2020/10/29/BJ9vKs.png)点击continue按钮会提示注册，这里又有坑。

> 由于蝗虫一般涌入试图薅羊毛的中国人让 `freenom` 极度不爽, `freenom` 的免费域名注册对中国人并不友好, 极大概率注册会失败. 为了顺利注册免费域名, 请首先用美国 IP 翻着墙, 使用 Google 账号直接登录该站, 不必自主注册账号, 在填写个人资料时, 用 [fakenamegenerator.com](https://www.fakenamegenerator.com/) 胡诌个美国住址, 就可以随意注册免费域名了. 



### **3.域名解析**

​	完成以后点击Services-->My Domains进入域名管理页面。

![BJCk24.png](https://s1.ax1x.com/2020/10/29/BJCk24.png)



点击域名后面的Manage Domain进入域名解析



![](https://raw.githubusercontent.com/penginman/PicBed/master/20201029160037.png)

![BJCpV0.png](https://s1.ax1x.com/2020/10/29/BJCpV0.png)



​	Nameservers就是域名解析服务器进入后两个选项，第一个是使用freenom的域名解析服务器，第二个是使用其他域名解析商的服务器。

![BJC9aV.png](https://s1.ax1x.com/2020/10/29/BJC9aV.png)



之后点击Manage Freenom DNS进行域名解析就可以使用注册的域名访问了，示例：

![BJCiPU.png](https://s1.ax1x.com/2020/10/29/BJCiPU.png)