---
title: DMCTF之Misc
categories: ['CTF']
tags: ['DMCTF2020','Misc']
cover: 'https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/cover/20201120232221.png'
date: 2020-12-01 11:20:45
---


# 前言

这次比赛是第一次做Mics的题awa。

# Misc

## Check_in

真·有手就行

![](https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/artical/20201130102716.png)

## fakezip

看到题目**fakezip**翻译：假的压缩包，所以猜应该是伪加密，贴一个原理的博客：[zip伪加密](https://blog.csdn.net/u011377996/article/details/79286958)，使用010 Editor打开压缩包，

![](https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/artical/20201130103932.png)

找到01修改为00，再解压压缩包，虽然提示了压缩包错误但是直接无视，获得一个flag.txt

打开以后内容是：`♬♩¶♯♬♭♬♫♫♪♬∮♬♭‖♭♬♭♬∮♬♭‖♭♬♭♫♫♯=`，音符加密虽然是第一次听说，但是百度还是可以简单找到：[文本加密为音乐符号](https://www.qqxiuzi.cn/bianma/wenbenjiami.php?s=yinyue)

![](https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/artical/20201130104321.png)

## Base family

题目给出了是base家族，所以base所有种类都试一遍。base常见的种类有：`base16`、`base32`、`base58`、`base64`、`base91`。还有几种没听过的base种类可以在这个网站找到：[CTF在线工具](http://ctf.ssleye.com/)。

**原层**：

```
XUZbB{fp}U)=ql[n%GCbk9RZ7!XD$D)f1G{011LN(TSlXCJT:4nxQ[8Y#I:=k.Qi4t3/S!,N/%[I}^8jjP|0&whvi88gpQce(2lKt9ZHiT^g1.nZH,k=kjTT16pHJ_DrW,Td"^w$Q8+8T])e.llK?*z`gS:+C]llUG:z1=ekEN}8DmJf&GP<Rk:o_Jk<J.zp8%H0g7sYSTJ9p."duRBGj`g0!I+xjm(fh)]IF:>omN8=m+Xp(X0:U*8Sj5|8p._o[i0:%.qu}%_=<D
```

**base91解码**：

```
3G6MzYGwFwTsqcb3MWzTdQBTHZWBZ2LUBprZ3P62T2nsbt1R7o6a7PEsXsBvSFvoexeZJEkhW9Wv1VusvpWK1nfWsVHDypW2j3MMEygzSYLmwxKV5kNwWomvXc5ohX2Jgj6bMRnu6JXkasXdbbw3Aw8Pvh6vWwPfTZ4mpkpNU9fDhyNi1bciCZMXeLiCWL67BVupHPobQcFWkpftgLPggB8wgwW
```

**base58解码**：

```
JZVFSMSZPJMXQTTKMMZVS2TDGVGXUQJTJZKFM3KONJCTEWL2MN4U26SNGJGVIWJQJZ5GWMK2NJNGSTTNKV5E2RDDGNHFOWJTJZCFSNCNPJGTCWTKKF4U42SFGNGXUWJRJZVFSMSNKRNGWTL2IUZFS6TDGVHDEUJ5
```

**base32解码**：

```
NjY2YzYxNjc3Yjc5MzA3NTVmNjE2YzcyMzM2MTY0Nzk1ZjZiNmUzMDc3NWY3NDY4MzM1ZjQyNjE3MzY1NjY2MTZkMzE2Yzc5N2Q=
```

**base64解码**：

```
666c61677b7930755f616c72336164795f6b6e30775f7468335f4261736566616d316c797d
```

**hex解码**：

```
flag{y0u_alr3ady_kn0w_th3_Basefam1ly}
```

## SlientEye

根据题目直接百度**SlientEye**，下载以后打开图片-->decode：

![](https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/artical/20201130105145.png)

参数啥的都没动，密码为默认密码，直接Decode，得到一个flag.txt：

```html
&#102;&#108;&#97;&#103;&#123;&#110;&#49;&#95;&#98;&#117;&#95;&#106;&#49;&#97;&#110;&#103;&#95;&#119;&#117;&#95;&#100;&#51;&#125;
```

再去百度搜到一篇博客：[&#x开头的是什么编码?](https://www.cnblogs.com/philipding/p/10153094.html)，~~我点开一看，哦，原来是~~entity code实体编码，~~我啪的一下就~~把flag.txt改为flag.html，~~很快啊，然后是一个左正蹬、一个右鞭腿、一个左刺拳~~打开flag.html获得flag：**flag{n1_bu_j1ang_wu_d3}**

## 编码之王

下载文件打开后一堆社会主义核心价值观，前面提到的：[CTF在线工具](http://ctf.ssleye.com/)就有核心价值观编码，下面放密文，上面是解码内容。

解出来以后看到第一句：`如是我闻:`，-->[与佛论禅](http://www.keyfc.net/bbs/tools/tudoucode.aspx)

再解之后看第一句：`新佛曰：`。-->[新与佛论禅](http://hi.pcmoe.net/buddha.html)

解完以后直接出了一堆由：`[、]、!、+、(、)`组成的符号，还是百度找到了这种编码叫JSfuck，可以直接浏览器控制台console输出获得flag

![](https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/artical/20201130110943.png)

## jpgsteg

题目即是用到的工具，百度下载软件，使用jphide.exe打开图片`Tap code.jpg`，选择seek功能解密，密码为123456（我蒙的）：

![](https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/artical/20201130111311.png)

成功解密后获得一个文档，内容如下：

```
... ....
.. ...
.. ...
. .
... ..
. .
..... .....
.. ....
... ...
.. ..

```

这里我思索了好久，刚开始以为是摩斯密码，但是又对不上号，最后找到了一个名为敲击码的，正好一行中的`.`分成两部分代表坐标

![](https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/artical/20201130111538.png)

解码得到：`ohhamazing` ，加上括号就是：**flag{ohhamazing}**

事后多看了一眼图片命名：`Tap code.jpg`。这用啥解密不就在脸上写着的~~wosabi \0/~~。。。

## Collision

打开压缩包发现都是加密，但是原始大小都为4，只有CRC不一样：

![](https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/artical/20201201102424.png)

所以很有可能是四位数据的CRC32碰撞，就去网上找了一个python脚本：[crc32碰撞 ctf python](https://blog.csdn.net/weixin_45396639/article/details/103393759)作者原创的脚本：

```python
import string
import threading
import binascii
import sys


def crc(_crc):
    l = 1
    dic = string.printable
    _input = _crc
    _input = "0X" + _input
    for i in dic:
        for n in dic:
            for h in dic:
                for m in dic:
                    s = i + n + h + m
                    s = s.encode()
                    # print(str(binascii.crc32(s)), _input)
                    if hex(binascii.crc32(s)).upper() == _input.upper():
                        print(_crc, ':', s.decode())
                        print(l)
                        sys.exit()  #直接退出，不进行接下来的碰撞了，一般在做题的时候，碰撞不会重复
                    l = l + 1


def crc32():
    print("四字节碰撞!!!")
    num = int(input("你可能需要多个线程同时进行碰撞，请输入线程数："))
    _thread = []
    _args = []
    print("输入参数")
    for i in range(num):
        print(i+1, end=':')
        _args.append(input())
    # print(_args)
    for i in range(num):
        t = threading.Thread(target=crc, args=(_args[i],))
        _thread.append(t) #如果在这里开始线程会出一点小bug，虽然不要紧，但是不好看，不信的话你们自己试试
    # print(_thread)
    for i in range(num):
        _thread[i].start()
    for i in range(num):
        _thread[i].join()
    input()


if __name__ == '__main__':
    crc32()
```

使用python的多线程，因为是5个文件，开了5个线程，然后分别输入CRC码，压缩文件原来的CRC码为：

```
ff92876d
6c4a558b
77e8fd00
1e59a66e
d1f4eb9a
```

碰撞以后获得的明文：

```
1on}
32co
llis
flag
{crc
```

根据flag的结构拼接一下：**flag{crc32collis1on}**

## kaomoji

题目的压缩包解压以后得到一个**flag.zip**压缩包和**secret.txt**，flag.zip中也含有secret.txt，将外面没有加密的secret.txt使用**winrar**压缩（需要和原来的压缩软件一致哒）以后对比flag.zip里的文件发现CRC码相同的：

![](https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/artical/20201201103227.png)

配合ARCHPR使用明文攻击，获得加密密钥： **[b00df998 5bdbbde6 485fa1f8\]**

我在进行明文攻击时虽然没有跑出来压缩包的密码，但是获得了上面的密钥，也是可以解压加密的压缩包的（具体原理不清楚）。解压明文攻击解开获得的压缩包**flag_decrypted.zip**，打开flag.txt是颜文字表情加密，直接复制到浏览器console控制台运行获得flag：flag{kaomoj1_1s_cut3}

## ARCHPR

题目根据提示：**This file was encrypted by me with a four-digit password, try to crack it!**

密码只是用了4位数字加密，可以使用ARCHPR 进行爆破，获得一个flag.png和hint.txt，hint.txt如下：

```hint.txt
The flag is hidden by a kind of magic called LSB. Try to find it if you can find it. The key is given to you. After decryption, change it to lowercase.

key: .--. .- ... ... .-- --- .-. -..
```

提示中提到了使用LSB隐写，密码是一段摩斯密码，摩斯密码解密后得到密文：`password`，使用LSB（[项目地址](https://github.com/livz/cloacked-pixel)）脚本执行：

```
python lsb.py extract flag.png 1.txt password
```

**1.txt**中就包含了解密以后的flag：**flag{th1s_15_f1agggggg}**

## outguess

​	看题目找工具，使用outguess参考[隐写工具outguess 的下载安装及使用](https://blog.csdn.net/weixin_43877387/article/details/103123858)，在kali中安装以后执行

```
outguess -r flag.jpg hide.txt
```

获得hide.txt文件的内容：

```
Qb lbh xabj NRF? Gur xrl vf f3phe1gl, tb naq penpx vg!


Encrypted data: U2FsdGVkX1/nmu9u2Ho1dD9kQWv7L5a6bsUrWxBkVp68txdFL4v/givGGYy7dBU+
```

上面一段使用**凯撒密码**移动13位获得：**Do you know AES? The key is s3cur1ty, go and crack it!**

其实这里和别人讨论以后才知道他们使用的是叫ROT13，相应的还搜到了ROT5、ROT13、ROT18、ROT47，百度百科看了以后就是凯撒密码的变种。所以下面一段的密文使用AES进行解密，密码是`s3cur1ty`，获得flag：**flag{y0u_ar3_awes0m3}**

解密网站：https://tool.oschina.net/encrypt

## Whitespace

题目即提示，Whitespace进行一波搜索以后了解到是一种用空白符编程的语言，在压缩包里面摸了好久，在注释里发现空白编码：

![](https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/artical/20201201104847.png)

这也让我想到自己在网上下工具的压缩包时，网站经常在注释里附上密码，通常都是网站的网址。

在[whitespace](https://vii5ard.github.io/whitespace)网站中粘贴密文点上面的`run`：

![](https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/artical/20201201105235.png)

解密获得：**password is BlindWaterMark!**

输入解压以后获得两张看着相同的罗翔老师.png图片和一个hint.txt：

```hint.txt
Do you see any difference between the two pictures?
Get to know its secrets and tell you quietly that you need to know a knowledge called Manchester coding.


上传文档
你看这两幅画有什么不同吗？
了解它的秘密，悄悄地告诉你，你需要知道一门叫做曼彻斯特编码的知识。
```

根据提示找出图片的不同和曼切斯特编码。图片看着相同但是经过加密，在网上搜索了一波了解到了**盲水印**技术，使用盲水印解密图片，项目地址：https://github.com/chishaxie/BlindWaterMark，执行：

```
python bwn.py decode 2.png 1.png 3.png 
```

获得解密图片：

![](https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/artical/20201201105728.png)

图片中的内容为：

```
296969a5695
6696a6a9a69
5669595a566
965696666aa
69596a9666a
a6a6569955a
5a66aa69a56
9566a6a6aa6
```

就百度曼切斯特编码解码，找到一篇博客：[一些CTF编码脚本](https://blog.csdn.net/weixin_30416871/article/details/98566881)，在里面找到了这两段：

![](https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/artical/20201201105901.png)

心里一顿狂喜，因为都是`2965a`这个几个字符组成的，应该就是找对了。根据博客代码自行修改脚本

```
import sys
s = '296969a56956696a6a9a695669595a566965696666aa69596a9666aa6a6569955a5a66aa69a569566a6a6aa6'  #这是前面图片里的内容

s=bin(int(s,16))
r=""
for i in range(len(s)/2):
    if s[i*2:i*2+2] == '10':
        r += '1'
    else:
        r += '0'
print hex(int(r,2))[2:-1].decode('hex')
```

执行脚本后获得flag：**flag{ab1de_by_th3_law}**

## Steghide

题目即工具。参考博客[隐写工具Steghide](https://www.jianshu.com/p/c3679f805a0c)，在kali中安装Steghide后执行：

```
steghide.exe extract -sf trump.jpg
```

会提示`Enter passphrase:`直接回车表示空密码，获得flag.txt，打开以后里面都是由`¿ ¡ .`组成的密文，和**Ook**的另一种编码：**short Ook**类似也是只用`? ! .`组成，把叹号和问号全部替换反过来`¡--->!  ¿--->?`，替换之后在线解码：https://www.splitbrain.org/services/ook，获得flag：**flag{y0u_ar3_clev3r}**

## SSTV

题目既是工具。搜索SSTV百度百科：

> 慢扫描电视（Slow-scan television）是业余无线电爱好者的一种主要图片传输方法，慢扫描电视通过无线电传输和接收单色或彩色静态图片。

了解到是一种无线电传递图片的方法，搜到的博客[慢扫描电视 SSTV](https://blog.csdn.net/zkf0100007/article/details/83387790)和B站的视频[慢扫描电视SSTV](https://www.bilibili.com/video/BV1ea4y1J787)，下载MMSSTV软件，把output.wav音频调制麦克风输出，我使用的方法是在声音设置里把立体声混音打开并且设为默认设备，然后电脑里播放的声音就会被录制到。等待图片绘制成功，获得flag：**DMCTF{SSTV,yyds?}**

![](https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/artical/20201201111538.png)

## SimpleQrcode

​	下载题目是一个gif图片都是二维码，使用使用stegSovle中的Frame Browser功能，一帧一帧播放，一帧一扫，有几帧图片是少了二维码的上边，有一张是少了右边，可以参考第一张完整的图片，把上面截取拼接上去，扫码后17张图片对应的内容（根据代码行号）：

```
DM
CT
F{
Qr
Co
de
_1
s_
so
_i
nt
er
es
ti
ng
!!
!}
```

参考下题目标题和flag格式，拼接后获得flag：**DMCTF{QrCode_1s_so_interesting!!!}**
