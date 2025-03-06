---
title: WUSTCTF2020 朴实无华
date: 2021-04-16 15:33:32
categories: ['CTF']
tags: ['RCE']
---

进入页面直接报错

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20210416103737.png)

试试其他的地方，我的习惯是git泄露、请求头、robots.txt、hint.txt都看看。果然在robots下有内容

> User-agent: *
> Disallow: /fAke_f1agggg.php

访问`fAke_f1agggg.php`并且抓包，在响应头里有提示。

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20210416105630.png)

访问`fl4g.php`才正式开始，源码如下：

```php
<img src="/img.jpg">
<?php
header('Content-type:text/html;charset=utf-8');
error_reporting(0);
highlight_file(__file__);


//level 1
if (isset($_GET['num'])){
    $num = $_GET['num'];
    if(intval($num) < 2020 && intval($num + 1) > 2021){
        echo "我不经意间看了看我的劳力士, 不是想看时间, 只是想不经意间, 让你知道我过得比你好.</br>";
    }else{
        die("金钱解决不了穷人的本质问题");
    }
}else{
    die("去非洲吧");
}
//level 2
if (isset($_GET['md5'])){
   $md5=$_GET['md5'];
   if ($md5==md5($md5))
       echo "想到这个CTFer拿到flag后, 感激涕零, 跑去东澜岸, 找一家餐厅, 把厨师轰出去, 自己炒两个拿手小菜, 倒一杯散装白酒, 致富有道, 别学小暴.</br>";
   else
       die("我赶紧喊来我的酒肉朋友, 他打了个电话, 把他一家安排到了非洲");
}else{
    die("去非洲吧");
}

//get flag
if (isset($_GET['get_flag'])){
    $get_flag = $_GET['get_flag'];
    if(!strstr($get_flag," ")){
        $get_flag = str_ireplace("cat", "wctf2020", $get_flag);
        echo "想到这里, 我充实而欣慰, 有钱人的快乐往往就是这么的朴实无华, 且枯燥.</br>";
        system($get_flag);
    }else{
        die("快到非洲了");
    }
}else{
    die("去非洲吧");
}
?>
```

一关一关的**bypass**。

第一关

重点在`intval`函数，intval函数有个特性:

>  直到遇上数字或正负符号才开始做转换，再遇到非数字或字符串结束时(\0)结束转换

这里需要num的值小于2020，加一后值大于2021，可以使用科学计数法方法绕过。构造payload`2e9`，遇到第一个函数转换时，因为是以2开头下一位是字符，会直接被截取为2；遇到第二个函数，因为使用了`$num + 1`会进行类型转换，2e9会先使用科学计数法计算出值并+1。

第二关

需要一个md5值，对这个值再次使用md5加密以后，使用**弱类型**比较，和原来的值相同。md5的一个弱类型比较绕过：

> PHP在处理哈希字符串时，它把每一个以“0E”开头的哈希值都解释为0，所以如果两个不同的密码经过哈希以后，其哈希值都是以“0E”开头的，那么PHP将会认为他们相同，都是0。

使用脚本跑一下得到一个值：`0e215962017`。

最后一层，是一个`system`函数执行，但是在之前有一些过滤

* `strstr`函数匹配空格，可以使用${IFS}绕过（这个里面有更详细的：[命令执行漏洞利用及绕过方式总结](https://www.ghtwf01.cn/index.php/archives/273/)）
* `str_ireplace`会吧cat替换成wctf2020，所以不能使用cat命令，可以用：more、less、od、tail等等绕过，上面的博客里也有写道。

先使用ls查看下当前目录下的文件，发现一个名为`fllllllllllllllllllllllllllllllllllllllllaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaag`读取得到flag。

最终payload：

```url
/fl4g.php?num=2e9&md5=0e215962017&get_flag=more${IFS}fllllllllllllllllllllllllllllllllllllllllaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaag
```

