---
title: "为VSCode加上Cursor的代码补全"
date: 2026-01-07T10:50:42+08:00
slug: code-completion-for-vscode
categories: 'Play'
tag: ['VS Code','效率工具']
draft: true
---

之前用的 Windsurf 的免费代码补全最近用的很闹心，于是乎去找新的替代品，发现效果不错，记录一下（~~才不是为了把年度总结刷下去~~）。

## 下载插件扩展

相关项目：https://github.com/Haleclipse/Cometix-Tab

仓库中有说有用到实验性 API 需要 Insider 版本才可以使用其实后面能偷跑。

下载拖到 VSCode 里安装。

## 配置扩展

打开 VSCode 设置搜索`cometix Tab` 

1.  `Server URL` 修改为 `https://api2.cursor.sh.lilingfeng.dev` 。
2. `Auth Token` 随便填

## Stable 版本偷跑

找到 VSCode 安装目录下的 `resources\app\product.json` ，找到 `extensionEnabledApiProposals` 配置项，在列表中**添加**：

```
"Haleclipse.cometix-tab": [
	"inlineCompletionsAdditions"
],
```

然后重启 VSCode

## 效果

代码补全效果如下。

![](https://img.braindance.top/2026/01/completionexample-f5e1e6973953a8e1a5d0f24fdad26e6e.png)

## 最后

看到这的切用且珍惜，因为是公益佬在维护的。完结撒花。
