---
title: Upload-Labs(一)
date: 2020-11-08 19:23:16
tags:  ['文件上传漏洞']
categories: ['CTF']
cover: https://cdn.jsdelivr.net/gh/penginman/PicBed@master/cover/20201111104319.png
---

## 介绍

> 大部分的网站和应用系统都有上传功能，而程序员在开发任意文件上传功能时，并未考虑文件格式后缀的合法性校验或者是否只在前端通过js进行后缀检验。这时攻击者可以上传一个与网站脚本语言相对应的恶意代码动态脚本，例如(jsp、asp、php、aspx文件后缀)到服务器上，从而访问这些恶意脚本中包含的恶意代码，进行动态解析最终达到执行恶意代码的效果，进一步影响服务器安全。

在线靶机地址：

* [linux环境](https://buuoj.cn/challenges#Upload-Labs-Linux)
* [windows环境](https://buuoj.cn/challenges#[Windows]Upload-Labs-Windows)



## Pass-01

​	尝试上传php木马，发现提示上传错误

![](https://s1.ax1x.com/2020/11/08/BTKbfs.png)

​	提示只能上传`jpg、png、gif`类型的图片。查看源码发现是一个前端的后缀过滤，那么我们尝试绕过前端的JS代码。

​	源码：

```javascript
function checkFile() {
    var file = document.getElementsByName('upload_file')[0].value;
    if (file == null || file == "") {
        alert("请选择要上传的文件!");
        return false;
    }
    //定义允许上传的文件类型
    var allow_ext = ".jpg|.png|.gif";
    //提取上传文件的类型
    var ext_name = file.substring(file.lastIndexOf("."));
    //判断上传文件类型是否允许上传
    if (allow_ext.indexOf(ext_name + "|") == -1) {
        var errMsg = "该文件不允许上传，请上传" + allow_ext + "类型的文件,当前文件类型为：" + ext_name;
        alert(errMsg);
        return false;
    }
}
```



​	把文件后缀名改成jpg格式上传，使用**burp suit**抓包。把`.jpg`后缀重新改为`.php`即可实现绕过前端JS代码。

![](https://s1.ax1x.com/2020/11/08/BTM5g1.png)

​	然后右键打开图片，代码成功执行。完工

![](https://s1.ax1x.com/2020/11/08/BTQV8s.png)

​	注：后面题目的php代码都使用`2333.php`：

```php
<?php eval(phpinfo());
```

​	执行结果是打印出php版本信息。



## Pass-02

​	源码：

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        if (($_FILES['upload_file']['type'] == 'image/jpeg') || ($_FILES['upload_file']['type'] == 'image/png') || ($_FILES['upload_file']['type'] == 'image/gif')) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH . '/' . $_FILES['upload_file']['name']            
            if (move_uploaded_file($temp_file, $img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '文件类型不正确，请重新上传！';
        }
    } else {
        $msg = UPLOAD_PATH.'文件夹不存在,请手工创建！';
    }
}
```

​	发现文件判断是根据`image/jpeg、image/png...`进行过滤判定，这些值都是Http请求中的**Content-Type**常见的值，通常浏览网页中各种各样的文件类型的就是通过它判断。那么这道题的目标就是绕过它。贴一个我参考值种类的博客:[Http请求中Content-Type](https://www.cnblogs.com/klb561/p/10090540.html)。

​	同样是burp抓包修改**Content-Type**的值。

![](https://s1.ax1x.com/2020/11/08/BT1EXq.png)

​	打开图片，php代码成功执行。完工



## Pass-03

源码：

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array('.asp','.aspx','.php','.jsp');
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //收尾去空

        if(!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.date("YmdHis").rand(1000,9999).$file_ext;            
            if (move_uploaded_file($temp_file,$img_path)) {
                 $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '不允许上传.asp,.aspx,.php,.jsp后缀文件！';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}
```

​	发现只过滤了`.asp、.aspx、.php、.jsp`文件，那么可以使用`php3、phtml、phps、php5`文件绕过过滤，并执行语句。通常，在嵌入了php脚本的html中，使用 `phtml`作为后缀名；而php3，我的理解是php之前版本的文件后缀，如`php5`。

​	直接把`2333.php`改为`2333.php3`上传打开，执行成功。完工

![](https://s1.ax1x.com/2020/11/09/BTq2cV.png)



## Pass-04

源码：

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2","php1",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2","pHp1",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = strtolower($file_ext); //转换为小写
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
        $file_ext = trim($file_ext); //收尾去空

        if (!in_array($file_ext, $deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH.'/'.$file_name;
            if (move_uploaded_file($temp_file, $img_path)) {
                $is_upload = true;
            } else {
                $msg = '上传出错！';
            }
        } else {
            $msg = '此文件不允许上传!';
        }
    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}
```

​	好家伙，这次直接一大堆文件都被过滤了，几乎有问题的都在数组里。发现没有.`htaccess`文件过滤，所以上传一个.`htaccess`文件内容如下：

```
SetHandler application/x-httpd-php
```

​	原理的话我讲一下个人的见解：.`htaccess`文件是一个`apache`服务器的配置文件，它的作用就是对于该目录下的所有文件都需要符合这个配置文件。然后上传的文件内容作用是：所有文件访问时都会解析为`php`。参考的博客：[htaccess使用方法介绍](https://www.cnblogs.com/gyrgyr/p/10773118.html)。

​	接下来上传`2333.jpg`图片木马，再打开就会被成功解析为`php`文件并执行：

![](https://s1.ax1x.com/2020/11/09/BTqoN9.png)

![](https://s1.ax1x.com/2020/11/09/BTqqc6.png)

​	完工



## Pass-05

 源码：

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");
        $file_name = trim($_FILES['upload_file']['name']);
        $file_name = deldot($file_name);//删除文件名末尾的点
        $file_ext = strrchr($file_name, '.');
        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA
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

​	细心的话可以发现，这道题的源码中在末尾添加了`.htaccess`文件过滤，但是也少了一条语句

```php
$file_ext = strtolower($file_ext); //转换为小写
```

目标就很明确了，直接使用大小写绕过过滤。

![](https://s1.ax1x.com/2020/11/09/BTLmNj.png)

​	打开图片成功执行。完工

