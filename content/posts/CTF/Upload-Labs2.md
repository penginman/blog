---
title: Upload-Labs(二)
date: 2020-11-09 20:12:27
tags:  ['文件上传漏洞']
categories: ['CTF']
cover: https://cdn.jsdelivr.net/gh/penginman/PicBed@master/cover/20201111104025.jpg
---

## 前言

​	这次彻底的从头到尾分析了一下源码的执行过程，大致的写一下，以防以后再看的时候不知道题目是什么情况。

```
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
    //这里下面是过滤
        $deny_ext = array(".php",".php5",".php4",".html", ......);
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //首尾去空

	//这里下面是移动文件。
    	if (!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.$file_name;
            if (move_uploaded_file($temp_file, $img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '此文件类型不允许上传！';
        }
   
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}
```



过滤部分：

* `$deny_ext`是一个过滤的后缀数组，只要是在里面的后缀都是被禁止上传的。

 * `$file_name = trim($FILES['upload_file']['name'])`这段代码有两个点需要说：
   1. `$FILES['upload_file']['name']`是获取上传文件的名称，PHP中$FILES是一个预定义的数组，用来获取通过 POST 方法上传文件的相关信息。如果为单个文件上传，那么 $FILES 为二维数组；如果为多个文件上传，那么 $FILES 为三维数组。贴一个参考的博客：[PHP $_FILES函数详解](https://www.cnblogs.com/laijinquan/p/8682282.html)。
   2. `trim()`函数的作用就是去除文件名称前后的空格换行符等。
* `$file_name = deldot($file_name)`这个注释中很清楚，是删除文章末尾的点。
* `$file_ext = strrchr($file_name, '.')`中`strrchr(string s1,char c1)`函数查找字符或字符串c1在另一个字符串s1中最后一次出现的位置，并返回从该位置到字符串结尾的所有字符。说白了就是获取文件的后缀名。
* `$file_ext = strtolower($file_ext)`注释上转换小写。
* `$file_ext = str_ireplace('::$DATA', '', $file_ext)`去除字符串::$DATA。第八题讲了原理

上传部分：

* `in_array($file_ext, $deny_ext)`判断文件的后缀(第一个参数)是不是在黑名单数组(第二个参数)中。
* `$_FILES['upload_file']['tmp_name']`文件被上传后在服务端储存的临时文件名，一般是系统默认。可以在php.ini的upload_tmp_dir 指定。
*  `$img_path = UPLOAD_PATH.'/'.$file_name`这个变量是设置需要保存到的路径
* `move_uploaded_file($temp_file, $img_path)`本函数检查并确保指定的文件(第一个参数)是合法的上传文件(即通过 PHP 的 HTTP POST 上传机制所上传的)。如果文件合法，则将其移动为由指定的文件路径(第二个参数)。



## Pass-06

源码：

```
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
        $file_name = $_FILES['upload_file']['name'];
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        
        if (!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.date("YmdHis").rand(1000,9999).$file_ext;
            if (move_uploaded_file($temp_file,$img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '此文件不允许上传';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}
```

​	看源码发现少了`trim()`函数**对文件名前后的空格处理**，所以我们可以在上传文件时在后缀名后面添加空格使其成为.php  (有空格)绕过黑名单数组。

![](https://s1.ax1x.com/2020/11/09/Bb9jQe.png)

​	上传以后访问文件执行成功。完工

![](https://s1.ax1x.com/2020/11/09/Bb9xLd.png)

​	这里说明一下，我前面是在[BUUCTF](https://buuoj.cn/)在线靶场上做的linux环境下的题目，但是这道题用了上面的方法怎么都访问不到，所以我在本地windows环境上搭建了一个靶机进行上传(而且后面有道题必须是在windows环境下才可以通过)。在github上下载的源码题目比在线靶场上的题目多了一道，对应的题目为 在线靶机pass-06-->github下载的pass-07。默认使用的都是linux环境下的题目，有改变会提前说明。

​	

## Pass-07

源码：

```
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //首尾去空
        
        if (!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.$file_name;
            if (move_uploaded_file($temp_file, $img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '此文件类型不允许上传！';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}
```

​	这道题少了**删除文件名末尾的点**，我们可以通过构造**2333.php. .**(末尾加 点 空格 点)，被解析后文件后缀就会成为". "(一个点一个空格)，可以绕过黑名单，访问的文件名是`2333.php. .`

![](https://s1.ax1x.com/2020/11/09/BbCOkq.png)

​	我又参考了其他博客，讲到windows环境下可以利用系统会自动删除后缀中最后的一个"."，尝试在windows靶机上测试**只添加一个点**，访问的文件名为`2333.php、2333.php.`都可以，因为windows会删除最后一个点。

![](https://s1.ax1x.com/2020/11/09/BbMywV.png)

![](https://s1.ax1x.com/2020/11/09/BbKXq0.png)

​	两种方式第一个在linux环境下的php服务器上，第二个在windows环境下的php服务器上，上传后都可以成功访问文件。完工



## Pass-08

源码：

```
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess",".ini");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = trim($file_ext); //首尾去空
        
        if (!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.date("YmdHis").rand(1000,9999).$file_ext;
            if (move_uploaded_file($temp_file, $img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '此文件类型不允许上传！';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}
```

​	审查代码发现少了对`::$DATA`字符串的处理，这里就要用到前面提到的windows环境了，贴一下原理：

> php在window的时候如果文件名+"::$DATA"会把::$DATA之后的数据当成文件流处理,不会检测后缀名.且保持"::$DATA"之前的文件名

​	直接上传的时候抓包在文件名后面添加`::$DATA`。

![](https://s1.ax1x.com/2020/11/09/BbP84P.png)

​	上传访问。完工



## Pass-09

```
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //首尾去空
        
        if (!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.$file_name;
            if (move_uploaded_file($temp_file, $img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '此文件类型不允许上传！';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}
```

​	这次题目和第七题差不多，代码会先剔除文件名前后的空格，然后删除末尾的点，再通过`strrchr()`函数截取后缀名转换小写。

​	所以和第七题一样构造**2333.php. .**(末尾加 点 空格 点)，被处理后的文件名后缀就成了一个点"."，铁定不在黑名单后缀里，实现绕过后缀检查。

​	但是这道题**只能使用windows环境**，因为执行了`deldot()`函数删除了最后一个点之后，文件名成了`2333.php.`，而linux环境下因为不会自动删除最后一个点而不能访问成功。

​	图前面有了就不贴了。



## Pass-10

源码：

```
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array("php","php5","php4","php3","php2","html","htm","phtml","pht","jsp","jspa","jspx","jsw","jsv","jspf","jtml","asp","aspx","asa","asax","ascx","ashx","asmx","cer","swf","htaccess");

        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = str_ireplace($deny_ext,"", $file_name);
        $temp_file = $_FILES['upload_file']['tmp_name'];
        $img_path =  UPLOAD_PATH.'/'.$file_name;        
        if (move_uploaded_file($temp_file, $img_path)) {
            $is_upload = true;
        } else {
            $msg = '上传出错！';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}
```

​	这道题和前面不同的是`$file_name = str_ireplace($deny_ext,"", $file_name)`，对于这个函数：`str_ireplace(find,replace,string,count)`，find是要查找的值，replace是要替换成的值，string是被搜索的字符串，count 可选。一个变量，对替换数进行计数。所以这行代码的作用就是把文件名中所有包含在黑名单后缀里的字符串替换成空串，但是这个函数只会执行一次，所以我们可以构造一个双写绕过，即构造文件名`2333.pphphp`，只有一个"php"字符串被匹配到并被替换成空串，剩下的文件名就成了`2333.php`。

![](https://s1.ax1x.com/2020/11/09/Bb8yPf.png)

上传并访问文件。完工