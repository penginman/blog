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

**配置开机自动挂载**。

* 方法1：编辑 `~/.config/autostart/rclone-onedrive.desktop` 添加以下内容

```
[Desktop Entry]
Type=Application
Name=Mount OneDrive with rclone
Exec=rclone mount onedrive:/backup/ /mnt/onedrive --allow-non-empty --vfs-cache-mode full --daemon writes &
Terminal=false
Hidden=false
```

* 方法2：直接把挂载命令写成一个 sh 脚本文件，在 crontab 中添加配置每次开机后自动执行。

```
@reboot /path/to/rclone-onedrive.sh
```

### 自动备份

采用 sqlite3 先热备数据库，然后打包。备份计划每天一个日备份，保存在daily文件夹下，保留30份，循环覆写。每个月一个月备份，保存在monthly文件夹下，保留12份，循环覆写。注意安装依赖环境

```
apt-get install sqlite3
```

脚本文件如下：

```
#!/bin/bash

# 定义路径
# 数据地址
DATA_DIR="/vw-data"
# 网盘挂载的地址
BACKUP_DIR="/mnt/onedrive"
DAILY_BACKUP_DIR="${BACKUP_DIR}/daily"
MONTHLY_BACKUP_DIR="${BACKUP_DIR}/monthly"
WORK_DIR="${BACKUP_DIR}/work"
DATE=$(date +%Y%m%d)
DAILY_BACKUP_RETENTION=30
MONTHLY_BACKUP_RETENTION=12

# 定义加密密码
GPG_PASSPHRASE="your_secret_passphrase_here"  # 你的加密密码

# 确保目标目录存在
mkdir -p "$DAILY_BACKUP_DIR" "$MONTHLY_BACKUP_DIR" "$WORK_DIR"

# Step 1: 执行SQLite3热备份
sqlite3 "${DATA_DIR}/db.sqlite3" ".backup '${WORK_DIR}/db_${DATE}.sqlite3'"

# Step 2: 打包备份文件
tar -czf "${WORK_DIR}/backup_${DATE}.tar.gz" -C "${WORK_DIR}" "db_${DATE}.sqlite3"

# Step 3: 加密备份文件
echo "$GPG_PASSPHRASE" | gpg --batch --yes --passphrase-fd 0 --symmetric --cipher-algo AES256 --output "${DAILY_BACKUP_DIR}/backup_${DATE}.tar.gz.gpg" "${WORK_DIR}/backup_${DATE}.tar.gz"

# Step 4: 删除临时文件
rm "${WORK_DIR}/db_${DATE}.sqlite3" "${WORK_DIR}/backup_${DATE}.tar.gz"

# Step 5: 每月备份逻辑（每月第一天执行）
if [ "$(date +%d)" -eq 1 ]; then
  cp "${DAILY_BACKUP_DIR}/backup_${DATE}.tar.gz.gpg" "${MONTHLY_BACKUP_DIR}/"
fi

# Step 6: 保留最近的30份每日备份
cd "${DAILY_BACKUP_DIR}"
ls -tp | grep -v '/$' | tail -n +$((${DAILY_BACKUP_RETENTION}+1)) | xargs -I {} rm -- {}

# Step 7: 保留最近12个月的备份
cd "${MONTHLY_BACKUP_DIR}"
ls -tp | grep -v '/$' | tail -n +$((${MONTHLY_BACKUP_RETENTION}+1)) | xargs -I {} rm -- {}
```

修改脚本权限

```
chmod 700 /path/to/backup.sh
```

创建定时任务每天凌晨 2 点自动备份。使用crontab @reboot，记得加+x权限

```
0 2 * * * /path/to/backup.sh
```

可以手动运行一次脚本，检查云盘中是否有备份文件，同时重启一下服务器查看网盘是否自动挂载。

### 恢复备份

1. 可以同样用管道符传入密码。（**不推荐，安全问题**）

```
echo "$GPG_PASSPHRASE" | gpg --batch --yes --passphrase-fd 0 --decrypt --output "${WORK_DIR}/backup_${DATE}.tar.gz" "${DAILY_BACKUP_DIR}/backup_${DATE}.tar.gz.gpg"
```

2. 交互式输入密码，执行命令后会弹窗输出密码。（**推荐**）

```
gpg --decrypt --output "${WORK_DIR}/backup_${DATE}.tar.gz" "${DAILY_BACKUP_DIR}/backup_${DATE}.tar.gz.gpg"
```



