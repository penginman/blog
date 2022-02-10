---
title: PicGo复制自定义链接
categories: ['瞎折腾']
tags: ['PicGo上传','图床']
cover: 'https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/cover/20201116160027.jpg'
date: 2020-11-30 09:54:11
---

2022年2月3日22:41:32

**！！！！！**

**建议直接看文章末尾，我发现我就是个笨比。**

# 前言

现在博客里面的图片使用的是github+jsdelivr+PicGo图床。PicGo是一个开源的上传图片的软件，支持大部分图床的上传，只需要设置对应的图床参数即可一键上传。github上PicGo的概述：

> **PicGo: 一个用于快速上传图片并获取图片 URL 链接的工具**
>
> PicGo 本体支持如下图床：
>
> - `七牛图床` v1.0
> - `腾讯云 COS v4\v5 版本` v1.1 & v1.5.0
> - `又拍云` v1.2.0
> - `GitHub` v1.5.0
> - `SM.MS V2` v2.3.0-beta.0
> - `阿里云 OSS` v1.6.0
> - `Imgur` v1.6.0
>
> **本体不再增加默认的图床支持。你可以自行开发第三方图床插件。**

项目地址：[PicGo](https://github.com/Molunerfinn/PicGo)

软件界面：

![](https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/artical/20201130095609.png)

# 起因

PicGo上传成功以后可以自动复制url，但是默认复制的图片链接是github提供的，github在国内又经常抽风，图片通常都是无法访问的，所以我使用了jsdelivr提供的链接访问图片，PicGo也提供了自定义链接，但是规定必须包含`$url`参数，也就是默认的url地址：

![](https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/artical/20201130093835.png)

起初是把jsdelivr的链接直接拼在后面，每次使用的时候都再删一次。直到昨晚受不了了就尝试去修改软件试图绕过判断。

# 解决方法

在软件设置的配置文件里找到了`customLink`，和之前自己设置的链接对照了一下，认为这个就是实际的自定义链接，直接在此处修改，再上传自动复制的链接就正确了，而且绕过了必须包含$url。![](https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/artical/20201130094455.png)

![](https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/artical/20201130094651.png)

# 结语

软件提供了自动使用时间戳重命名，所以我自定义链接中还是使用了`$filename`变量，然后博客中的图片大部分都是使用QQ的截屏功能，出来的截图后缀都是`png`格式，这个可以根据需要自己设定拼接，然后使用的markdown编辑器是typora，想要添加图片直接<kbd>Crtl</kbd>+<kbd>Shift</kbd>+<kbd>i</kbd>，把图片链接贴里面就彳亍了。<span class='heimu'>~~白嫖真爽~~</span>

---

2022-1-31 11:43:48

这时候发现自定义链接直接用markdown格式就更好了

```other
![$fileName](https://cdn.jsdelivr.net/gh/username/repo@master/artical/$fileName.png)
```

---

# 最简单修改

前面的都是我在改自定义链接，至少明白了可以绕过自定义链接必须包含变量名称这个限制。

![image-20220203224558136](https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master/artical/202202032246055.png)

直接在自定义域名那里修改成jsDeliver对应的仓库路径，比如我的

```other
https://cdn.jsdelivr.net/gh/guobang-yoo/PicBed@master
```

然后返回的链接就会把原来的`raw.github.com/xxxxxx`给替代了。
