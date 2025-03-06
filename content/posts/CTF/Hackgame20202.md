---
title: Hackgame2020(二)
categories: ['CTF']
tags: ['Hackergame 2020','Java安全']
date: 2020-11-05 09:29:22
---


### 一闪而过的Flag

​	根据题目信息**程序每次运行时隐约可见黑色控制台上有 flag 一闪而过**，我想到了应该能看程序的代码啥的找到，但是~~天下武功，唯快不破~~，下载 运行 截图一气呵成。

​	![](https://s1.ax1x.com/2020/11/09/B7TOUA.png)

​	接下来让我康康哪一位是~~瞎子~~，包括答案里也是满满的嘲讽。

**参考答案：flag{Are_you_eyes1ght_g00D?_can_you_dIst1nguish_1iI?}**



### 从零开始的记账工具人

​	~~npy~~给了我一张账单，让我计算账单上面的金额，flag就是由金额组成的。本以为是一个简单的Excel的表格函数解决，打开我傻了。

![](https://s1.ax1x.com/2020/11/05/B2Flxf.png)

​	原来是搞这个大小写金额的转换，没见过Excel有这种操作就去百度，转了一大圈，网上只有数字转大写金额的，没有反过来的。还去了github上找代码，发现运行的结果和自己想要的还是有些出入。卡了有足足半天，之后自己写了一个Java代码跑了一遍，代码如下：

```java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner sn = new Scanner(System.in);
        String str;
        int x;
        int cnt=0;
        double result=0;
        while(cnt++!=1000){
            str = sn.next();
            x = sn.nextInt();
            result += tonum(str)*x;
        }
        System.out.println(result);
    }
    public static double tonum(String s){
        int len;
        double res=0,temp = 1;
        double result=0;
        len = s.length();
        for(int i=0;i<len;i++){
            switch (s.charAt(i)){
                case '壹': temp=1;break;
                case '贰': temp=2;break;
                case '叁': temp=3;break;
                case '肆': temp=4;break;
                case '伍': temp=5;break;
                case '陆': temp=6;break;
                case '柒': temp=7;break;
                case '捌': temp=8;break;
                case '玖': temp=9;break;

                case '零': break;
                case '拾': res+=temp*10;temp=0;break;
                case '佰': res+=temp*100;temp=0;break;
                case '元': res+=temp;temp=0;break;
                case '角': res+=temp*0.1;temp=0;break;
                case '分': res+=temp*0.01;temp=0;break;
                case '整': break;
                default:
                    System.out.println("这个认不出来" + s.charAt(i));
            }
        }
        return res;
    }
}

```

​	程序用的控制台输入，直接表格两列从头拉倒尾粘贴，出答案。程序的思路就是把金额大写当做字符串处理，每一位用`switch`判断数字或是个、十、百、千那一位上的数字。

![](https://s1.ax1x.com/2020/11/05/B2F7ee.png)

​	Java渣轻喷。

**参考答案：flag{19115.33}**



### 超简单的世界模拟器

​	这道题下面有两道小题

### 蝴蝶效应

​	先看有意思的一个漫画

![](https://s1.ax1x.com/2020/11/05/B2k9eg.png)

​	一个人用石头模拟了一整个宇宙，斯巴拉西。

​	打开题目以后是一个终端，然提示等待你输入一个**15*15矩阵**（只能有0和1组成），看到这挺懵的，但是题目里有一个重要信息**生命游戏**，百度百科看了一下这个[生命游戏](https://baike.baidu.com/item/%E7%94%9F%E5%91%BD%E6%B8%B8%E6%88%8F/2926434?fr=aladdin)，游戏的规则大致就是：**一个细胞会根据周围的细胞数量判断存活和死亡**，然后细胞会不断演算，这个和前面的漫画有异曲同工之妙。

​	返回终端里面一片白中间有几个框框![](https://s1.ax1x.com/2020/11/05/B2ZFBT.png)，题目中

> 如果被特殊标注的正方形内的细胞被“清除”，你将会得到对应的 flag：
>
> “清除”任意一个正方形，你将会得到第一个 flag。同时“清除”两个正方形，你将会得到第二个 flag。

​	用细胞去碰框框就是了，正好也看到了一个知乎的提问：[*生命游戏*(Game of Life)有哪些图形? - 知乎](https://baike.baidu.com/item/%E7%94%9F%E5%91%BD%E6%B8%B8%E6%88%8F/2926434?fr=aladdin)，看了看内容更有趣了。

![](https://s1.ax1x.com/2020/11/05/B2elzn.png)

​	大师我悟了，就是提供一个初始的**15*15矩阵**，根据**生命游戏**的规则进行演算，然后去消除黑框框。我臭屁完了直接贴图形

* 第一关

```轻量级飞船
000000000000000
000000000000000
000000000000000
000000000000000
000000000000000
000000000000000
000000000011000
000000000111100
000000000110110
000000000001100
000000000000000
000000000000000
000000000000000
000000000000000
000000000000000
```

​	第一关打上面的黑框，刚好在上面15行的范围内，用一个**轻量级飞船**直线打过去即可。

* 第二关

```三飞船
000000000000000
000000000000000
000000000000000
000000000000000
000000000011000
000000000111100
000000000110110
000000000001100
000000000000000
001100000000000
011110000000000
011011000001100
000110000011110
000000000011011
000000000000110
```

​	第二关我期初试了试**滑翔者**放在右上角以便能打到最远距离，但是和第二个都是擦肩而过。于是我就乱试乱拼凑，最后拼出来一个**三飞船**，正好把两个黑框都给消了。

**参考答案：**

**1. flag{D0_Y0U_l1k3_g4me_0f_l1fe?_d5e1c80641}**

**2. flag{1s_th3_e55ence_0f_0ur_un1ver5e_ju5t_c0mputat1on?_ea3e769cb8}**



完工。