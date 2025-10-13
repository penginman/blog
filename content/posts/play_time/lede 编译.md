---
title: lede 编译
date: 2023-09-15
categories: ["Play"]
tags: ["软路由"]
draft: true
---

主题

```
cd package
mkdir openwrt-packages
cd openwrt-packages
git clone https://github.com/jerrykuku/luci-theme-argon.git
git clone https://github.com/thinktip/luci-theme-neobird.git
git clone https://github.com/rosywrt/luci-theme-rosy.git
git clone https://github.com/gngpp/luci-theme-design.git
git clone https://github.com/xiaoqingfengATGH/luci-theme-infinityfreedom.git
```

软件

```
git clone https://github.com/Zxilly/UA2F.git package/UA2F
git clone https://github.com/mchome/openwrt-dogcom.git package/dogcom
git clone https://github.com/UnblockNeteaseMusic/luci-app-unblockneteasemusic.git package/UnblockNeteaseMusic
```

修改源

```
sed -i '$a src-git kenzo https://github.com/kenzok8/openwrt-packages' feeds.conf.default
sed -i '$a src-git small https://github.com/kenzok8/small' feeds.conf.default
git pull
./scripts/feeds update -a
./scripts/feeds install -a
make menuconfig
```

theme

LuCI --> Themes

UnblockNeteaseMusic

LuCI --> Applications --> ua2f

UA2F

Network --> Routing and Redirection --> luci-app-unblockneteasemusic

dogcom

Network > dogcom

配置编译 iptables 模块（没配也照样用了）

Network --> Firewall -->

- iptables-mod-filter
- iptables-mod-ipopt
- iptables-mod-u32
