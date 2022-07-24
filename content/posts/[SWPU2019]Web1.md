---
title: SWPU2019 Web1
date: 2021-04-15 09:55:17
categories: ['CTF']
tags: ['sql注入','无列名注入']
cover: https://cdn.jsdelivr.net/gh/penginman/PicBed@master/cover/20201116160013.jpg
---

一个登陆界面，再看下url地址为login.php，确认了使用的是php

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20210415091617.png)

直接查看源码，在源码中找到了register.php。先注册一个进去看一看。

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20210415091705.png)

是一个发布广告的信息页，发布广告时需要输入广告的标题和内容，联想一下之前做过的发布文章的，应该是sql注入，输入广告标题输入一个单引号`'`试一试

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20210415091924.png)

广告详情中出现了数据库报错

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20210415091959.png)猜测应该是二次注入，发布时加上一些转义字符没有出现错误，但是查看详情时再次从数据库中取出带有payload的数据，可以执行sql语句。而且上面的单引号测试出来了是**字符型单引号闭合**。

接下来是尝试过滤。我试出来的有空格（使用`/**/`绕过）、or，and（可以使用&&，||），同时or被过滤，就说明保存数据库表名的库**information_schema**没有办法查询，并且**orderby**也不能使用，需要使用其他办法获取表内容。

测试一下当前表的字段数，从1开始加，一直会报字段数不匹配，直到尝试到22。还需要主义的是执行的sql语句后面的 `LIMIT 0,1`需要闭合，所以最后添加了一个单引号

```sql
'/**/union/**/select/**/1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22'
```

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20210415092716.png)

说明回显位是2和3。

由于没有办法查询表名，所以需要我们自己猜测，CTF比赛中常见的表名就是flag、users、举办方的缩写啥的。~~这种没有办法查询表名的题目表名应该都不会设置太难~~

测试的语句 

```sql
'/**/union/**/select/**/1,(select/**/*/**/from/**/flag),3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22'
```

当测到`users`表时返回了当前字段数不匹配的错误，说名users里有多个字段，肯定没有办法显示在一列中。所以接下来是猜users表的字段数。

首先讲解一波**无列名注入**。

先来个正常表（flag）的查询

![](D:\DROPS\CTF比赛题解\BUUCTF\Web\[SWPU2019]Web1\20210415094632.png)

我们知道在sql语句查询的时候，可以给列名起别名形如

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20210415094059.png)

使用联合查询的时候，只要前后表的字段数相同，前面查询的就会成为表名

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20210415094238.png)

所以当我们不知道flag表的字段，并且想要查询里面的内容时，比如我想查询flag表的flag字段，可以构造

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20210415094814.png)



需要为子查询的结果再起一个别名（图中为`x`），这样我们就不用列名查询到了字段。总结一下思路就是：自己构造一个表名并且联合查询想要查询的表名，再使用`group_concat`函数输出自己构造的表名。

所以测试users表的字段数时，就通过形如上图的格式，改变联合查询的字段数判断。

最终的payload：

```sql
'/**/union/**/select/**/1,(select/**/group_concat(b)/**/from(select/**/1,2/**/as/**/a,3/**/as/**/b/**/union/**/select/**/*/**/from/**/users)x),3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21,22'
```



