---
title: sql-labs(一)
date: 2020-09-22 10:50:08
categories: ['CTF']
tags: ['sql注入']
---

## 前言

在线靶机地址：https://buuoj.cn/challenges#sqli-labs

## less-1

​	首先说明sql注入的大致步骤：

* 判断注入类型。如整型字符型注入等。
* 判断列数
* 判断数据的回显位
* 构造sql语句



​	根据题目提示，说明是一个单引号注入题目，构造一个带单引号的语句`?id=1'`，发现数据库报错

![](https://s1.ax1x.com/2020/11/04/BgERNd.png)

​	通过后面的报错语句`1'' LIMIT 0,1`的分析，我们的单引号被数据库解析，那么说明我们也可以使用连接查询`union`插入我们想要查询的语句。

​	推出数据库的查询的部分语句可能为`where id = '$id'LIMIT 0,1`，那么通过构造闭合`?id=1' [这里添加语句] --+`添加自己想要语句，语句后面的`--+`作用是将后面的其他语句注释掉。

​	首先是判断字段个数：`?id=1' order by 1 --+`，页面显示正常，直到尝试`?id=1' order by 4 --+`发现数据库报错

![](https://s1.ax1x.com/2020/11/04/BgVsGn.png)

​	说明数据库的字段值只有四个。

​	接下来测试数据的回显位，构造语句`?id=' union select 1,2,3 --+`，这里需要注意的有，前面`id`的查询一定是要不存在的，因为数据库只会回显第一条查询的数据，如果第一条语句查询成功则后面`union`构造的语句就不会显示；`union`连接查询语句后面查询的字段数需要和前面的字段数相等，详细用法可以自行查询。

​	执行后页面显示。

![](https://s1.ax1x.com/2020/11/04/BgZAeS.png)

​	说明查询语句的`2,3`是回显位，之后就可以将查询的语句进行替换。如：

​	获取数据库版本，数据库路径，当前用户，当前数据库：
​	`?id=' union select 1,concat_ws('_',user(),version(),database()),@@basedir --+` 

页面显示

![](https://s1.ax1x.com/2020/11/04/BgZbfs.png)

​	利用元数据库来爆表、爆数据

​	`?id=' union select 1,group_concat(table_name),3 from information_schema.tables where table_schema=database() --+`

​	之后大家可以自行发挥。

​	`flag`的话我做的题是在`ctftraining.flag`表中，答案在：`?id=' union select 1,flag,3 from ctftraining.flag  --+`

## less-2

​	第二题看题目名称`intiger based`知大意，是id的数据类型由字符型变成了数字类型，这次就不使用单引号直接构造语句，和第一题差不多。

## less-3

​	看标题`Single quotes with twist`，是在前面题的基础上加上了括号包裹，所以语句就成了`where id = ('id')`所以我们闭合的方式也要改变。附源码

![](https://s1.ax1x.com/2020/11/04/BgnO1A.png)

## less-4

​	标题`Double Quotes`，说明是个引号注入，把前面题的单引号改成双引号构成闭合即可。

## less-5

​	标题`Double Injection-Single Quotes`，很明显提示是单引号，然后套用前面的方法， 发现这次题目变了，不管输入啥页面只显示一个`You are in...........`，但是数据库报错还是会显示，只要数据库错误还能显示，我们就可以是用一个新的技术把数据显示在错误信息上。

​	双查询注入也是我第一次听，贴一个讲大致原理的帖子：[点我](https://blog.csdn.net/Leep0rt/article/details/78556440)。

​	构造语句：

`?id=' union select 1,2,3 from (select 1,count(*),concat_ws('____________',floor(rand()*2),concat_ws('********',version(),database()))a from information_schema.tables group by a)b --+`

​	讲一下`CONCAT_WS(separator,str1,str2,…)`函数的用法：把`str1`、`str2`连接起来，并使用`separator`做分隔符。

