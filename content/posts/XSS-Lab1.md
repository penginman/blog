---
title: XSS-Lab (一)
date: 2020-11-03 10:13:39
tags: ['XSS']
categories: ['DROPS']
cover: https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/cover/20201111120856.jpg
---

## 头

靶机地址：https://buuoj.cn/challenges#XSS-Lab

![](https://s1.ax1x.com/2020/11/03/BsmJ5n.png)

## level 1

​	![](https://s1.ax1x.com/2020/11/03/BsmfKO.png)

​	观察发现`欢迎用户test`与URL中的`name=test`字段对应，尝试修改发现可行。直接将`name`字段改为`name=<script>alert()</script>`，完工。



## level 2

![](https://s1.ax1x.com/2020/11/03/BsnwWt.png)

​	在搜索栏中的输入会回显到页面，查看网页源代码，发现值在`input`的`value`属性中

![](https://s1.ax1x.com/2020/11/03/BsnLk9.png)

​	构造闭合`"> <script>alert()</script> // `，构造以后的标签会变成`.... value=""> <script>alert()</script> // ">`。完工



## level 3

![](https://s1.ax1x.com/2020/11/03/BsKwKf.png)

​	老样子构造`value`属性闭合，尝试`"> <script>alert()</script> // `构造闭合，查看网页源代码发现语句变成`&quot;&gt; &lt;script&gt;alert()&lt;/script&gt; // `，说明把`"、<、>、`进行了html编码过滤，尝试构造`onmouseover`事件(鼠标移到事件)，构造语句`'onmouseover='alert()'`。完工



## level 4

![](https://s1.ax1x.com/2020/11/03/BsMWYd.png)

​	构造闭合`"> <script>alert()</script> // `，查看源代码发现进行了`>、<`过滤，尝试构造事件`" onmouseover='alert()'`。完工

​	说明一下标签的事件有很多类型，可以自己试试别的事件响应。



## level 5

![](https://s1.ax1x.com/2020/11/03/Bs1Ci4.png)

​	检查一下都有什么过滤。发现有一下过滤

>script --> scr_ipt
>
>onmouseover --> o_nmouseover

无法采用事件，那么尝试构造一个标签` "> <a href='javascript:alert()'>233</a> //`，发现`javascript`没有过滤，说明判断语句匹配值仅仅为`script`，点击构造的`<a>`标签内容。完工

​	