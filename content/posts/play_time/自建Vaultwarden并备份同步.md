---
title: '自建Vaultwarden并备份同步'
categories: ['瞎折腾']
date: 2025-04-13
---

##  前言

新买服务器，密码很多，保存密码，为了安全，防止被打，备份保底。

## 前置步骤

自行安装1panel面板，并开启https和面板的安全配置，如防火墙、fail2ban等。或者根据配置的流程自己摸索单独配置。

## 安装Vaultwarden 

我用的docker-composes.yaml

```yaml
services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: unless-stopped
    environment:
      ADMIN_TOKEN: "xxxxxxxxxxxxxx"
      DOMAIN: "https://vw.3306.fun"
    volumes:
      - ./vw-data/:/data/
    network_mode: bridge
```

这里的`ADMIN_TOKEN`很重要，相当于后台唯一的管理密码，虽然配置里写明文，用于第一次登录（~~我不会说因为直接在容器里配置没成功~~），后面在管理面板再更改。

之后创建站点反向代理容器的地址，以及开启https。

## 设置waf和限制

在1panel的高级功能-WAF-网站设置，找到创建的站点，然后设置访问频率

![image.png](https://img.braindance.top/file/2024/04/1744507809102_image.png)

然后在Cloudflare WAF中创建一个**速率限制**

![image.png](https://img.braindance.top/file/2024/04/pZBr6nA3.png)

![image.png](https://img.braindance.top/file/2024/04/nIDfbV3g.png)

**添加自定义规则设置**：

规则。阻止其他国家的访问，不想限制所有域名可以添加域名限制，我的表达式为`(http.host wildcard "example.com" and ip.src.country ne "CN")`

**配置DNS记录**，添加记录并开启小黄云（已代理）。

找到 SSL/TLS 概述，选择配置，自定义 SSL/TLS，选择完全（严格）。

## 登录配置

访问 https://example.com/ 创建账户。然后访问后台地址`/admin`输入创建docker时的`ADMIN_TOKEN`登陆，之后**关键步骤**：

1. 将Domain URL设置为自己的网站。
2. Client IP header设置为“cf**-**connecting-ip”，保证只有Cloudflare访问
3. Admin token/Argon2 PHC 选项设置为新的`ADMIN_TOKEN`

## admin后台限制

因为admin页面权限过大，平时也用不到，所以可以在1panel中配置路径重定向到127.0.0.1，需要更改配置的时候关掉重定向，修改关闭再打开。

![image.png](https://img.braindance.top/file/2024/04/ZUOYha2U.png)

## 其他配置

在Vaultwarden开启两步验证等，自行摸索。

## 同步配置

- **Rclone**：用于将备份文件上传到 OneDrive。
- **GPG**：用于加密备份文件。
- **Cron**：用于定时执行备份和更新脚本。

安装rclone

```shell
curl https://rclone.org/install.sh | sudo bash
```

验证安装成功

```
rclone version
```

https://linux.do/t/topic/238502

根据帖子配置网盘：https://linux.do/t/topic/481620。最后代码修改为如下，后台挂载。

```
rclone mount onedrive:/backup/ /mnt/onedrive --allow-non-empty --vfs-cache-mode full --daemon
```



使用crontab @reboot，记得加+x权限

```

apt-get install sqlite3
```

