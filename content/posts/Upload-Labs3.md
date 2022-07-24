---
title: Upload-Labs(三)
categories: ['DROPS']
tags: ['文件上传漏洞']
cover: 'https://cdn.jsdelivr.net/gh/penginman/PicBed@master/cover/20201116160237.png'
date: 2020-11-16 16:04:30
---

## 前言

​	继续接着上一次的`Upload-labs`往下写。这记下第11-15题，目前进度是20题都已经完成正在抽时间写博客，然后16题是我卡的最久的关，不过也学到了感觉很牛的姿势，所以到时候专门开一篇只讲16。

​	在线靶机地址：

* [linux环境](https://buuoj.cn/challenges#Upload-Labs-Linux)
* [windows环境](https://buuoj.cn/challenges#[Windows]Upload-Labs-Windows)

## Pass-11

​	(这题使用了windows环境)

​	源码：

```php
$is_upload = false;
$msg = null;
if(isset($_POST['submit'])){
    $ext_arr = array('jpg','png','gif');
    $file_ext = substr($_FILES['upload_file']['name'],strrpos($_FILES['upload_file']['name'],".")+1);
    if(in_array($file_ext,$ext_arr)){
        $temp_file = $_FILES['upload_file']['tmp_name'];
        $img_path = $_GET['save_path']."/".rand(10, 99).date("YmdHis").".".$file_ext;

        if(move_uploaded_file($temp_file,$img_path)){
            $is_upload = true;
        } else {
            $msg = '上传出错！';
        }
    } else{
        $msg = "只允许上传.jpg|.png|.gif类型文件！";
    }
}
```

​	分析代码发现是一个白名单验证，但是和之前不同点在于路径中使用了`$_GET['save_path']`，本题提示也写道

`本pass上传路径可控！`，就是通过这个GET变量控制上传路径。

​	这一关的突破方法需要有一些条件：php版本需要低于`5.3.29`（我使用的是php版本5.3.17的本地靶机），另一个条件是`magic_quotes_gpc`需要为关闭状态。`magic_quotes_gpc`的作用官方文档写道：

>**Warning** 本特性已自 PHP 5.3.0 起*废弃*并将自 PHP 5.4.0 起*移除*。
>
>​      为 GPC (Get/Post/Cookie) 操作设置 magic_quotes 状态。      当 magic_quotes 为 on，所有的 ' (单引号)、" (双引号)、\（反斜杠）和 NUL's 被一个反斜杠自动转义。     

​	使用bp抓包并添加`0x00截断`，在GET请求中可以使用url编码的截断：`%00`。贴个自己参考的博客，[截断上传原理剖析](https://www.cnblogs.com/milantgh/p/3612978.html)。个人对于这道题的分析就是，上面文件的代码执行到第8行的时候，获取到了`$_GET['save_path']`变量的值，但是我们在这个变量后面添加了`0x00截断`，所以后面的代码便不会执行，文件也就不会被重命名。

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201115204109.png)

​	文件成功上传，然后访问的时候记得改一下路径，因为文件名已经截断，所以访问路径由`..../upload/233.php�/5120201115205501.jpg`变为`..../upload/233.php`。完工



## Pass-12

​	（windows环境）

​	这题和上一题差不多一样，就是把`$_GET['save_path']`变成了`$_POST['save_path']`。由GET请求改成了POST请求，但是抓包修改的地方就不一样了，需要通过**16进制修改**

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201115210935.png)

​	这里我命名为`233.phpa`是因为方便我在Hex表中找到这句话的位置和修改数据。`a`的16进制是`61`，需要改成截断的值：`00`

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201115211024.png)

​	上传成功以后打开图片，和上一题一样，需要把路径中已经截断的后面删除掉。完工



## Pass-13

​	（Linux环境）

​	源码中有关的函数解释：

> [PHP中pack、unpack的详细用法](https://segmentfault.com/a/1190000008305573)
>
> [fread()](https://www.php.net/manual/zh/function.fread.php)
>
> [fclose()](https://www.php.net/fclose)
>
> [intval()](https://www.php.net/intval)

​	这道题和前面题目都不一样了：

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201115211509.png)

​	题目说道需要**上传图片马**，然后使用**文件包含漏洞**进行测试，那么我们就先制作图片马。

​	查看本题的提示`本pass检查图标内容开头2个字节！`，意思就是只检测文件前面两个字节的标识，贴一个[各种格式图片文件头标识分析](https://blog.csdn.net/qq_37414405/article/details/84660148)，所以这道题只用在上传的文件头部的两个字节处粘贴对应**文件的头标识**即可绕过检测。

​	以GIF的文件头GIF89a 为例，创建文件notepad++编辑打开输入`GIF89a<?php phpinfo(); ?>`，后缀名无所谓了，因为题目只检测前两个字节即可上传。

​	**还有一种方法制作图片马**：使用windows的`copy /b`指令，把两个文件进行~~无缝~~拼接，可以使用一张正常的图片加一个php文件进行拼接，即可正常上传。参考博客：[windows窗口命令——(copy/b)文件无缝拼接隐藏](https://blog.csdn.net/gaoshi66/article/details/83653143)。

​	别忘了题目中说道了需要**三种后缀都上传成功才算过关！**

​	**上传以后需要使用文件包含进行判定是否执行**，先在新标签页面中打开图片，url中记下图片在服务器中的目录和名称（我的图片名称为8220201116071327.gif），点击**2**的链接进入`include.php`进行文件包含：网址输入`https://...../include.php?file=./upload/8220201116071327.gif`，找到php成功执行的页面。这里还有一个小知识点我学到的就是：[路径中的'.'和'..'还有'./'和'../'都是什么意思](https://www.cnblogs.com/xc90/articles/10257402.html)。完工

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201116152417.png)



## Pass-14

​	（Linux环境）

​	这题提示写道`本pass使用getimagesize()检查是否为图片文件！`，和上一题不一样的地方就是使用了`getimagesize()`函数，这个函数的官方文档[getimagesize()](https://www.php.net/manual/zh/function.getimagesize.php)，所以这道题就不能用13题的加**文件的头标识**方法绕过，这次要使用上一题中的`copy /b`指令用一张正常的图片进行拼接还是可以通过的。

​	PS：听同学说`getimagesize()`不过是检测了前八个字节，不过我没试。

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201116155407.png)

​	因为是拼接的，所以要直接摸到图片最底部查看成功。完工



## Pass-15

​	（Linux环境）

​	13、14、15题都是对**文件的头标识**进行了检测，只不过第十四关使用的是getimagesize函数，第十五关使用的是exif_imagetype函数,函数返回值内容不一样而已。使用`copy /b`制作的图片马可以直接通过。

​	网上搜集过来的资料

> png 文件头  89504E470D0A1A0A
>
> jpg 文件头 89504E470D0A1A0A
>
> gif 文件头 474946383961

​	这几个字节应该都是够长的可以绕过这三个函数，所以验证了同学说的不同的函数检测的文件头长度是不一样的。完工



