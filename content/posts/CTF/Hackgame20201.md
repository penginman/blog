---
title: Hackgame2020(一)
categories: ['CTF']
tags: ['web','Hackergame 2020','代码审计']
date: 2020-11-01 18:33:40
cover: https://cdn.jsdelivr.net/gh/penginman/PicBed@master/cover/20201111120831.png
---

## 前言

比赛地址：https://hack.lug.ustc.edu.cn/

# web

### 签到题

​	第一题是flag提取器，打开题目以后页面是一个提取器下面一个`进度条`和一个`提取`按钮。

![BwJbJs.png](https://s1.ax1x.com/2020/11/01/BwJbJs.png)

​	随便拉了拉进度条发现初始是`0`末尾是`1.5`，然后中间的数值都是小数

![BwYAQx.png](https://s1.ax1x.com/2020/11/01/BwYAQx.png)

​	我猜的题目可能是想让数值为`1`时能提取到flag。

​	F12查看源代码，定位到进度条的标签，查看属性

![](https://s1.ax1x.com/2020/11/01/BwYoX6.png)

​	确实和猜测一样最大最小值分别是`0`、`1.5`，注意到`step`值为`0.00001`，就是这个属性让我们拖动进度条时，递增的值是小数，想精准得到`1`个flag，就把网页上的`step`值改为`1`。然后随便拖动一下进度条得到`1`，点击`提取`按钮，完工。

![](https://s1.ax1x.com/2020/11/01/BwNVaD.png)



### 2048

​	打开题目是一个`2048`的小游戏，第一个想法就是玩`2048`达到一定分数以后会获得flag（~~可恶这个比赛怎么知道最近我天天在玩2048的~~）,但是想比赛不可能让选手在线玩游戏吧，尤其对于我这种~~逃课废物~~是不可能努力哒！

F12开始翻网页资源

![](https://s1.ax1x.com/2020/11/01/BwUONF.png)

​	还好上次~~摸鱼~~课题做了一个网页版的2048，略微能看懂一点点结构，第一个`animframe_polyfill`应该就是管动画效果的，`keynpard...`这个肯定是相应按键的，`local_storage_manager`应该是保存游戏的，`game_manager`感觉就是管理整个游戏的主要文件。

​	打开`game_manager`粗略的看了下变量，`score` 分数、`over、won、keepPlaying`游戏状态，想赢游戏肯定是和`score、won`有关，直接Ctrl+F搜索整个文档这两个变量出现的位置。

发现了

![](https://s1.ax1x.com/2020/11/01/BwdGdK.png)



​	只有这里修改了`won`的状态，前面还有个`16384`这个应该就是获得胜利需要得到的分数，这个分数对于我来说还是简简单单（~~小声bb~~[截图为证](https://s1.ax1x.com/2020/11/01/BwUdte.jpg)），直接让`if`里面的语句为真就可以获胜，直接修改`if(1) self.won = true`，保存文件，按一个方向键让语句执行到，完工。

![](https://s1.ax1x.com/2020/11/01/BwwpTK.png)

# general

### 猫咪问题++

​	秉着前面题都是简单题，试着做了一下，看到题目提示的有往年的问题题解

![](https://s1.ax1x.com/2020/11/01/BwLtDs.png)

​	题意思很明显的书考验同学的搜题技巧。那就开工。

![](https://s1.ax1x.com/2020/11/01/BwLD8U.png)



#### 第一题

> 1. 以下编程语言、软件或组织对应标志是哺乳动物的有几个？
>
> Docker，Golang，Python，Plan 9，PHP，GNU，LLVM，Swift，Perl，GitHub，TortoiseSVN，FireFox，MySQL，PostgreSQL，MariaDB，Linux，OpenBSD，FreeDOS，Apache Tomcat，Squid，openSUSE，Kali，Xfce.
>
> 提示：学术上一般认为龙不属于哺乳动物。

​	本人只认识几个，那就一个一个搜呗。我的模式是：百度`xxx标志`然后第二条就是百度图片的搜索结果，大致辨别一下，然后再百度`xxx是哺乳动物吗`，妥妥的~~胎儿~~教学。

​	一个比较有印象就是`FreeDOS`这个标志就离谱，什么玩意

![](https://src.onlinedown.net/supply/sup_logo/logo-1122/46778_g.jpg)



**参考答案 ：12** 



#### 第二题

>2. 第一个以信鸽为载体的 IP 网络标准的 RFC 文档中推荐使用的 MTU (Maximum Transmission Unit) 是多少毫克？

​	卡了我好一会，没听过信鸽传输，贴上最后找到答案的[博客](https://blog.csdn.net/qq_31621387/article/details/77690642)，以及一篇信鸽传输的[历史发展](http://sohu.com/a/309403082_354973)，长见识了。

**参考答案：256**



#### 第三题

>3. USTC Linux 用户协会在 2019 年 9 月 21 日自由软件日活动中介绍的开源游戏的名称共有几个字母？
>
>   提示：活动记录会在哪里？

​	搜索`USTC Linux 用户协会`发现这个协会就是科大爱好者们创建的。那么直接摸到他们官网的[新闻版块](https://lug.ustc.edu.cn/news/)（百度搜出来的是旧站，里面有新站的网址）。题目中还写道`2019年9月21日自由软件日活动`，那么官网肯定有那天的新闻。

​	找到一篇当天的新闻[2019 软件自由日中国科大站](https://lug.ustc.edu.cn/news/2019/09/2019-sfd-ustc/)，进取直接找，文章末尾有

>最后一项是李文睿同学介绍了开源游戏 Teeworlds，由于底层代码开源，开发者可以做出自己的定制，可玩性非常高。

​	答案就是`Teeworlds`

​	我还摸到了他们当天活动的记录资料：[点我](https://ftp.lug.ustc.edu.cn/%E6%B4%BB%E5%8A%A8/2019.09.21_SFD/)

​	在`slides\闪电演讲\Teeworlds`文件夹下有应该作者演讲的PPT和游戏的视频演示，有点心动了🤣

**参考答案：9**



#### 第四题

> 4. 中国科学技术大学西校区图书馆正前方（西南方向） 50 米 L 型灌木处共有几个连通的划线停车位？

​	直接百度地图搜图书馆，然后全景地图房门口，视野拉到L型灌木那。

![](https://s1.ax1x.com/2020/11/01/BwvkdA.png)

**参考答案：9**



#### 第五题

> 5. 中国科学技术大学第六届信息安全大赛所有人合计提交了多少次 flag？

​	百度`中国科学技术大学第六届信息安全大赛`有个`...圆满结束`，就他了。点开第二行就是`经统计，在本次比赛中，总共有 2682 人注册，1904 人至少完成了一题。比赛期间所有人合计提交了 17098 次 flag`。（看这个网站的标志似乎还是第三题搜的那个`USTC Linux 用户协会`的官网新闻。嗷原来题目上已经说了是举办方👀）

**参考答案：17098**

完工。