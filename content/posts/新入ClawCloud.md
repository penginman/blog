---
title: "ClawCloud使用体验"
date: 2025-03-05
draft: false
categories: ['瞎折腾']
tags: ['服务器']
description: "新入了一台ClawCloud服务器，写一下使用感受"
---



https://linux.do/t/topic/160305

https://linux.do/t/topic/207789

https://linux.do/t/topic/115839

## 前言

快毕业了忙的飞起，上个服务器到期了我都没来得及管，现在写这篇博客还是在github的pages上，又接触了一些项目，想买个便宜好使的服务器玩玩。

在论坛里搜到[ClawCloud](https://claw.cloud/)评价不错，刚好最近有活动优惠就入手了一台系统为Debian，之前一直用的ubuntu，这下也换个玩玩。记录一下机器怎么样和自己配置过程，也边以后自己看。

| 2C / 2G / 40G / 1T | $25.20 USD |
| ------------------ | ---------- |

## 测评

部分测试结果：

```
VPS融合怪版本：2025.02.12
Shell项目地址：https://github.com/spiritLHLS/ecs
Go项目地址：https://github.com/oneclickvirt/ecs
---------------------基础信息查询--感谢所有开源项目---------------------
 CPU 型号          : Intel(R) Xeon(R) Platinum
 CPU 核心数        : 2
 CPU 频率          : 2500.002 MHz
 CPU 缓存          : L1: 32.00 KB / L2: 1.00 MB / L3: 33.00 MB
 AES-NI指令集      : ✔ Enabled
 VM-x/AMD-V支持    : ❌ Disabled
 内存              : 130.84 MiB / 1.85 GiB
 Swap              : [ no swap partition or swap file detected ]
 硬盘空间          : 911.28 MiB / 40110.19 MiB
 启动盘路径        : /dev/vda1
 系统在线时间      : 0 days, 0 hour 5 min
 负载              : 1.02, 0.34, 0.12
 系统              : Debian GNU/Linux 12 (bookworm) (x86_64)
 架构              : x86_64 (64 Bit)
 内核              : 6.1.0-31-cloud-amd64
 TCP加速方式       : cubic
 虚拟化架构        : KVM
 NAT类型           : Full Cone
 IPV4 ASN          : AS45102 Alibaba (US) Technology Co., Ltd.
 IPV4 位置         : Tokyo / Tokyo / JP
 IPV6 ASN          : AS45102 Alibaba
 IPV6 位置         : Tokyo / Tokyo / Japan
 IPV6 子网掩码     : 128
----------------------CPU测试--通过sysbench测试-------------------------
 -> CPU 测试中 (Fast Mode, 1-Pass @ 5sec)
 1 线程测试(单核)得分: 		1062 Scores
 2 线程测试(多核)得分: 		1770 Scores
---------------------内存测试--感谢lemonbench开源-----------------------
 -> 内存测试 Test (Fast Mode, 1-Pass @ 5sec)
 单线程读测试:		5350.57 MB/s
 单线程写测试:		5815.83 MB/s
------------------磁盘dd读写测试--感谢lemonbench开源--------------------
 -> 磁盘IO测试中 (4K Block/1M Block, Direct Mode)
 测试操作		写速度					读速度
 100MB-4K Block		35.0 MB/s (8551 IOPS, 2.99s)		52.4 MB/s (12800 IOPS, 2.00s)
 1GB-1M Block		232 MB/s (221 IOPS, 4.52s)		208 MB/s (198 IOPS, 5.04s)
---------------------磁盘fio读写测试--感谢yabs开源----------------------
Block Size | 4k            (IOPS) | 64k           (IOPS)
  ------   | ---            ----  | ----           ---- 
Read       | 20.68 MB/s    (5.1k) | 96.31 MB/s    (1.5k)
Write      | 20.69 MB/s    (5.1k) | 96.82 MB/s    (1.5k)
Total      | 41.37 MB/s   (10.3k) | 193.13 MB/s   (3.0k)
           |                      |                     
Block Size | 512k          (IOPS) | 1m            (IOPS)
  ------   | ---            ----  | ----           ---- 
Read       | 91.97 MB/s     (179) | 91.36 MB/s      (89)
Write      | 96.85 MB/s     (189) | 97.44 MB/s      (95)
Total      | 188.82 MB/s    (368) | 188.81 MB/s    (184)
```



完整测试结果连接如下：

https://paste.spiritlhl.net/#/show/WG2vI.txt 

http://hpaste.spiritlhl.net/#/show/WG2vI.txt        

咱也看不懂，也没有什么特殊需求，就部署点项目，下载包方便。

## 配置

### 更新组件、包管理

```
sudo apt update  #这个命令会更新软件包列表，让系统知道有哪些软件包可以更新。
sudo apt upgrade --only-upgrade #这个命令会安装所有可用的软件包更新。
```

### BBR

BBR 是 Google 提出的一种新型拥塞控制算法（Bottleneck Bandwidth and RTT），全称为瓶颈带宽和往返传播时间。

在 Linux 系统中，BBR 主要有以下特点和作用：

- **提高网络性能**：它可以显著提高吞吐量和降低 TCP 连接的延迟，使数据传输更加高效。
- **适应不同网络环境**：适合高延迟、高带宽的网络链路，以及慢速接入网络的用户，能在一定丢包率的网络链路上充分利用带宽，并降低网络链路上的缓冲区占用率从而降低延迟。
- **优化拥塞控制**：BBR 改变了传统基于丢包反馈的拥塞控制机制，通过精确测量往返传播时间（RTT）和瓶颈带宽等参数来更有效地控制数据发送速率，避免了传统算法中因单纯丢包判断拥塞而导致的带宽利用率不高和端到端延迟大等问题。
- **提升网络稳定性**：有助于减少网络拥塞和数据包丢失，提高网络的稳定性和可靠性。

**运行代码**

```
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh && chmod +x bbr.sh && ./bbr.sh
```

**重启 VPS、使内核更新和BBR设置都生效**

```bash
sudo reboot
```

**确认bbr是否开启**

```bash
lsmod | grep bbr
```

**结果为**

```
tcp_bbr                20480  1
```

### 添加SAWP

注意到测评结果里没有SAWP。在虚拟专用服务器（VPS）中，SWAP 是当物理内存（RAM）已被占满时用于存储数据的磁盘空间。它充当 RAM 的溢出区，允许系统将不活跃的内存页移动到 SWAP 空间，从而为活跃进程释放 RAM。这在处理占用大量内存的应用程序或同时运行多个应用程序时特别有用。然而，访问 SWAP 空间的速度比访问 RAM 慢，因为它涉及磁盘 I/O 操作。因此，虽然 SWAP 可以帮助避免内存不足错误，但应适当配置以确保 VPS 的最佳性能。

**使用脚本添加**

```bash
wget -O swap.sh https://raw.githubusercontent.com/yuju520/Script/main/swap.sh && chmod +x swap.sh && clear && ./swap.sh
```

注意输入SWAP的大小为单位M

**查看当前内存**

```sql
free -m
```

### 添加用户

1. 修改root密码，创建新用户，加入sudo列表；
2. 关闭root账户登陆，普通用户开启免密登录，并关闭密码登录；

```
sudo adduser xiaoming
sudo passwd xiaoming

echo 'xiaoming ALL=(ALL) ALL' | sudo tee /etc/sudoers.d/xiaoming
```

验证

```
su xiaoming
sudo cat /etc/shadow
```

如果能够成功执行命令，说明 sudo 权限已正确添加

```
ssh-keygen -t rsa
```

然后添加公钥内容到服务器`/home/user/.ssh/authorized_keys`中。

**建议登录验证一下！！！再关闭密码登录**

### 关闭root登录、用户密码登录

修改`/etc/ssh/sshd_config `

```
PermitRootLogin no # 禁用root登录
PasswordAuthentication no #禁用用户密码登录
```

如果不生效了注意一下`sshd_config.d`目录下的内容。重启ssh。

```
service ssh restart
```

重启不生效就`reboot`

### 修改ssh端口

修改`/etc/ssh/sshd_config `

```
#Port 22
```

### 安装Docker

```
# 安装Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

### 安装ufw

**1、更新软件包**

```
sudo apt update
```

**2、安装 UFW**

```
sudo apt install ufw
```

**3、如果你在远程位置连接你的服务器，在启用 UFW 防火墙之前，你必须显式允许进来的 SSH 连接。否则，你将永远都无法连接到机器上。**

```
sudo ufw allow 22/tcp
```

> 如果 SSH 运行在非标准端口，你需要将上述命令中的 22 端口替换为对应的 SSH 端口。

**5、启动 UFW**

```
sudo ufw enable
```

### fail2ban

**1、安装 Fail2ban**

```
sudo apt-get install fail2ban
```

**2、Debian 12 及以上的版本需要手动安装 rsyslog**

```
sudo apt-get install rsyslog
```

**3、启动 Fail2ban 服务**

```
sudo systemctl start fail2ban
```

**4、开机自启动**

```
sudo systemctl enable fail2ban
```

**5、查看 Fail2ban 服务状态。**

```
sudo systemctl status fail2ban
```

## 评价

好使的一。官网也有测试的探针，我这是东京的比香港的好使点。终端的体验感觉也比rn的好多了。如果可以的话就传家宝了。