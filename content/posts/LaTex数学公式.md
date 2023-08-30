---
title: LaTex数学公式
date: 2023-05-21
categories: ['学习笔记']
tags: ['LaTex']
draft: true
math: true
---

https://blog.csdn.net/m0_55746113/article/details/122728673

## 标注符号

### 上下标

上标符号为“^”、下标符号为“_” ，多于一个字符用`{}`包含，例如`2^r`、`a_5`

$2^r,a_5,A_{n+1}$

### 平均值、箭头、向量等

加^号：`\hat{x}`

加横线：`\overline{x}`

加^：`\widehat{x}`

加波浪线：`\widetilde{x}`

加一个点：`\dot{x}`

加两个点：`\ddot{x}`

$\hat x,\overline x,\widehat x,\widetilde x,\dot x,\ddot x$

### 加粗

矩阵字母一般会用加粗的罗马体来表示。`\bf`

## 希腊字母表

![1684640635418.png](https://img.braindance.top/artical/2023/05/21/6469937a48ead.png)



## 运算符号

### 加减乘除

`+` 和 `-`。`\times` 和 `\div`。点乘用 `\cdot`

$ +,-,\times,\div,\cdot$

### 大于、小于、约等于

大于小于直接  `>` 、 `<`

大于等于 `\ge`，小于等于 `\le`

远大于 `\gg`，远小于 `\ll`

不等于 `\ne`

约等于 `\approx`

$>,<,\ge,\le,\gg,\ll,\ne,\approx$

### 分式和根式

`\frac{分子}{分母}`或`\frac 分子 分母`。如

$ \frac{1}{2}$ 

### 交集并集

交集用 `\cap`

并集用 `\cup`

$\cap,\cup$

### 属于不属于

属于用 `\in`

不属于用 `\notin`

$\in,\notin$

### 省略号

横向省略号 `\cdots`

垂直省略号 `\vdots`

斜向省略号 `\ddots`

$\cdots, \vdots , \ddots $

### 求和与求积

求和用 `\sum`

求积（product）用 `\prod`

可以用`\limits`来强制显示在下方。

$\sum,\prod,\sum_i,\sum \limits_{i=1}^N,\prod_{i=1}^N x_i$

## 标注符号

### 括号

小括号直接 `()`，中括号直接 `[]`，大括号 `\{\}`。

左向上取整 `\lceil`，右向上取整 `\rceil`

 $\lceil ,\rceil $

左向下取整 `\lfloor`，右向下取整 `\rfloor`

$\lfloor ,\rfloor $

绝对值用`||`

#### 下方加符号

在任意符号下面加符号 `\underset{}{}`。例如 `\underset{B}{A}`。在下方换行使用 `\substack{}`。

$\underset{B}{A}，\underset{\substack{B>0 \\ C<0}}{A}$

### 花体符号

傅里叶变换中的f用 `\mathcal`

$\mathcal f,\mathcal L,\mathcal G$

## 多行公式

使用 `\\` 来换行，使用 `\begin{align}`、`\end{align}` 环绕表示多行环境。多行环境`\\`默认右对齐，如果想要使**某个符号对齐**，需要在符号前加 `&`，例如`=`号前面

$ 	\begin{align} a&=b+c+d \\ &=e+f \end{align}$
$$
\begin{align}
	a&=b+c+d \\
	&=e+f
	\end{align}
$$



