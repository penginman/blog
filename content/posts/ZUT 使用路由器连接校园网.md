---
title: 'ZUT 使用路由器连接校园网'
tags: ['校园网']
categories: ['瞎折腾']
date: 2022-10-11
lastmod: 2022-10-31


---

## 前言
我们学校的校园网不光每个月要宽带费 20/月，而且要绑定一个校园卡，我的一个月月租下来要 49/月，而且校园网还限制设备数量只能连接一个，我一般都是电脑连校园网，手机用流量，但是教职工的校园网就可以两个设备连接，寝室里目前有6+个设备也需要联网，商量以后打算搞个路由器贡献一个账号连校园网开 wifi，之前就见贴吧有老哥已经成功了，所以自己也尝试一下做个记录，给以后的同学看也不错。由于我已经连好路由器了，所以部分图片取自网络图片作为参考，我也尽量说的详细些。

{{< admonition >}}
1. 对于任何硬件、软件的损坏，本人没有赔偿的责任，哪怕这样的后果是因教程中的错误造成的。也请认真对待每一步操作，也许因为你的操作不当硬件因此变成一块砖。另外本教程的操作会使你的路由器失去保修。
2. 本篇文章仅起指导性的作用，在操作的过程中遇到的问题也许我也没有遇到过，请先自行尝试解决，如果我有空也会尽量帮助。
3. 学校明令禁止安装路由器，本人绝对没有怂恿或建议任何同学安装路由器，本人安装路由器的行为完全属于个人意志，仅仅作为个人的学习技术交流，请读者在安装好路由器之后 24 小时内再将之拆除，请按照学校指明的方式连接到互联网。如果学校根据校规等文件追究责任，与本人无关。

{{< /admonition >}}

叠甲过


## 前置知识

校园网插入网线认证是要模拟 Dr.com 软件的发包认证，除了账号密码认证校园网以外，软件还会定时发送心跳包保证校园网连接。

zut 的校园网认证格式为账号：`\r\n你的学号@[unicom|telecom|cmcc]` ，其中 `@` 后面的是你的运营商，对照前面的格式分别为：联通、电信、移动，例如我的是联通校园网，账号就是：`\r\n2022********@unicom`，密码就是你自己的密码。

zut 校园网的客户端版本为：6.0.0（P）

校园网的防检测插件原理还没有写，自己用的 小米4C 是社区固件资源中自带的。想看原理可以看最后挖坑部分

## 需要的材料

本教程使用的路由器型号 `Xiaomi router 4c` pdd二手35，安装的系统 `openwrt`。**请注意**：路由器的选择直接影响到你后面的操作，我专门在网上挑了好久选的这个路由器，但是，**唯独这个路由器的社区资源最特殊**，本人也是经过各种尝试才成功。如果选择其他路由器，本教程中的主要安装思路相同，但是需要自行寻找适合自己路由器的固件资源。

* 路由器。
* 网线两根。一根我买的8m的，因为宿舍AP在门上面，另一根需要连接路由器和电脑，长度自己看。
* 开通校园网的账号。
* 自己思考的能力。
* 需要的软件压缩包：[zut_Xiaomi_router_4c.zip](https://wwn.lanzouy.com/igsaW0dn0k3a)。其中包含
	* dogcom.zip。用来进行校园网模拟拨号，定时发送心跳包的工具。
	* R3GV2 patches.zip。刷机的主要部分包含了：R3GV2 patches（前期连接路由器的工具）、Mi4C.bin（小米 4c 的 openwrt 固件自带防检测插件）、breed.bin（breed web 恢复控制台）
	* MobaXterm_Portable.zip。用来连接路由器和传输文件的软件
	* 小米路由器4C恢复官方固件工具包.zip。刷成砖的[官方补救措施](https://web.vip.miui.com/page/info/mio/mio/detail?postId=19134127&app_version=dev.20051)

## 教程参考的网站或资料
* 贴吧老哥的教程：[https://tieba.baidu.com/p/7760362347](https://tieba.baidu.com/p/7760362347)
* 广东工业大学在 github 上的教程：[https://github.com/shengqiangzhang/Drcom-GDUT-HC5661A-OpenWrt](https://github.com/shengqiangzhang/Drcom-GDUT-HC5661A-OpenWrt)
* 【记录】小米路由器 4C 刷机过程：[https://github.com/shengqiangzhang/Drcom-GDUT-HC5661A-OpenWrt/issues/27](https://github.com/shengqiangzhang/Drcom-GDUT-HC5661A-OpenWrt/issues/27)
* R4CM，说说我的小米路由器4C刷机过程：不用Linux也不用虚拟机…… ：[https://www.right.com.cn/FORUM/thread-4047571-1-1.html](https://www.right.com.cn/FORUM/thread-4047571-1-1.html)
* 路由器认证校园网drcom：[https://www.brothereye.cn/router/669/](https://www.brothereye.cn/router/669/)

## 开工

接通路由器电源，**使用网线连接路由器**，进入初始化管理页面通常是 `192.168.1.1`，能跳过就跳过，下图右选择无需拨号即可。（网络图片仅供参考）

![ab3pm-21hcz.png](https://img.braindance.top/artical/2022/10/11/63450a2f460ac.png)

接下来设置 wifi 名称和密码，可以勾选管理密码和 wifi 密码相同的框，保存以后会提示新的管理后台地址，通常是 `192.168.31.1`，使用管理密码登陆以后，显示路由器界面就算初始化成功。

## 刷入不死 Breed

> Breed 是国内个人 hackpascal 开发的闭源 Bootloader（引导加载器，即为用于加载操作系统的程序），也被称为“不死鸟”。
> 因为有些官方升级固件自带 bootloader，如果从官方固件升级，会导致现有 bootloader 被覆盖。而当 Breed 更新固件时，它会自动删除固件附带的引导加载程序，因此可以防止 Breed 被覆盖。


刷入 Breed 的作用就是为后面刷固件做个保险，失败以后可以进行 reset 复位，防止路由器刷成一块砖。

### 开启路由器 telnet 和 ftp

打开前面下载的压缩包中的 `R3GV2 patches` 文件夹，运行其中的 `0.start_main.bat`，这个批处理的命令主要是运行了文件夹中的 `main.py` ，而 `main.py` 做的事情就是尝试发现你的路由器后台地址，然后需要输入**管理后台的密码**，然后执行一些 exploit 在路由器中写入后门，界面中出现 `Done` 字样就算成功。

这里遇到过扫描不到管理后台的情况，如果遇到了可以参考以下步骤。修改 `main.py` 文件中的第 10 行到 15 行，删掉
```python
line4 = subprocess.check_output(["cmd","/c","chcp","437","&","tracert","-d","-h","1","1.1.1.1"]).decode().split("\r\n")[4].strip().split(" ")

for data in line4:
	if len(data.split(".")) == 4:
		router_ip_address = data
		break
```

添加 `router_ip_address` 变量为你的管理后台地址，通常为 `192.168.31.1`
```python
router_ip_address = '192.168.31.1'
```


接下来使用 MobaXterm 连接路由器，新建一个 session 类型选择 telnet，这时的路由器地址就是管理后台的地址应该是 `192.168.31.1`，用户为 `root`。

![创建session](https://img.braindance.top/artical/2022/10/11/6345188f255ec.png)

ps：如果连接不成功就多运行几次 `0.start_main.bat`。

### 备份原路由器信息
**以下步骤请注意！！！**
最好备份以下路由器原本分区文件，以防不备之需，其中`eeprom.bin` 是最重要的，因为刷入 breed 以后可能会导致 MAC 地址全 0，需要刷入一次这个文件才会恢复。执行以下命令，`of` 后面的就是文件输出的路径

```
dd if=/dev/mtd0 of=/tmp/all.bin
dd if=/dev/mtd2 of=/tmp/bootloader.bin
dd if=/dev/mtd3 of=/tmp/eeprom.bin
```

打开电脑资源管理器，输入 `ftp://192.168.31.1` 按回车，使用 ftp 连接路由器（为什么不用 MobaXterm，因为后门创建的 ftp 是匿名用户，直接使用 windows 资源管理器打开更方便）

![连接 ftp](https://img.braindance.top/artical/2022/10/11/63451aca4e1e0.png)

找到 `tmp` 文件夹其中的三个文件：`all.bin` 、`bootloader.bin` 和 `eeprom.bin`，复制到本地做备份，下载以后一定要看清楚三个文件的大小分别是 `16MB` 、`64k` 和 `64k`，如果大小是 `1k` 那就是错误的，再备份几次，也有可能是后门没写牢再执行 `0.start_main.bat` 试试。

### 上传 Breed  文件并刷入

在资源管理器的 ftp 中把 `breed.bin` 文件上传到 `/tmp` 文件夹下，在 telnet 中执行指令刷入引导加载器

```
mtd write /tmp/breed.bin Bootloader
```

不提示错误信息就是刷入成功了。

### 重启路由器进入 Breed

拔掉路由器电源，用一根牙签类似的东西，插入路由器后面的 reset 孔不要松开，再插上电源，路由器灯会先闪一下，直到连续闪烁几次以后再松开 reset，这时路由器已经进入 breed 了，打开浏览器访问 `192.168.1.1` 就可以看到 Breed 的界面（网络图片仅供参考，系统信息可能不一样）

![Breed界面](https://img.braindance.top/artical/2022/10/11/63451fb563c3a.png)

如果你在之后有不可挽回的错误操作，都可以通过以上操作进行复位重置。

### 检查 MAC 地址

进入 MAC 地址修改，通常你应该看到的前三个 MAC 地址应该是全0，这就需要刷入 一次之前的 `eeprom.bin`。（网络图片仅供参考）

![MAC 地址修改](https://img.braindance.top/artical/2022/10/11/6345201b6c592.png)

进入**固件更新** （界面同下面），勾选 **EEPORM**，选择文件  `eeprom.bin`，其他的都不要动，然后上传，根据提示更新，之后会重新回到 Breed 控制台。

## 刷入 openwrt

同样在 Breed 控制台选择**固件更新** ，勾选**固件**，选择 `Mi4C.bin` openwrt 固件文件，根据提示上传安装。（网络图片仅供参考）

![固件更新](https://img.braindance.top/artical/2022/10/11/634527710db18.png)

之后路由器灯会全灭，然后电源灯进入**黄色闪烁**的状态，就是正在安装，等待安装成功以后等会变成**蓝色**，之后就可以访问 `192.168.1.1` 进入openwrt 管理后台页面，默认密码为 `password` 。

![openwrt 界面](https://img.braindance.top/artical/2022/10/11/634524441d739.png)

说一下 `Mi4C.bin` 这个openw 的固件。这个固件来自前面提到的广东工业大学项目中的 [issue](https://github.com/shengqiangzhang/Drcom-GDUT-HC5661A-OpenWrt/issues/27)，是一位同学自己找人定制的一份固件并且免费分享出来了，其中已经内置了 Dr.com 插件（用来发送心跳包）及防检测插件（ttl，ipid，cookieflash，ua2f 最新版），非常感谢这位同学的分享。

### 配置 PPPoE 拨号

**前置条件**：宿舍上面的AP接口插入路由器的 WAN 口

选择 网络 ---> 接口，点击 `WAN` 接口对应的 编辑（Edit）按钮。

下面图中是我的已经配置好的界面，初次进入应该是有个 `WAN` 和 `WAN6` 接口，它们两个的下面并不是我图中的 `pppoe-wan`，应该是 `eht0.2`。`WAN6` 接口是一个 DHCP IPV6 客户端，它和 `WAN` 接口是同一个物理接口，留着也不影响 。`LAN` 口尽量不要动。

![网络接口配置](https://img.braindance.top/artical/2022/10/11/63452a57015e9.png)

通信协议选择 `PPPoE` 然后点击出现的**切换协议**按钮。然后根据下表填入信息
* PAP/CHAP 用户名。校园网账号，前置知识中的`\r\n****@***`格式。
* PAP/CHAP 密码。校园网密码。

其他的不用动，请仔细检查校园网账号和密码是否正确。之后点击保存退出后，点击界面右下角的**保存并应用**。

![1665478137525.png](https://img.braindance.top/artical/2022/10/11/63452de3e1732.png)

PPPoE 部分配置完成。但是校园网目前还不能用。

### 配置无线网络

选择 网络 ---> 无线，如果提示已禁用就点击启用，只有一个你可以点击编辑的按钮。`ESSID` 就是设备搜索到的无线网名称。网络是 `LAN` 接口就不用动。

![无线网名称配置](https://img.braindance.top/artical/2022/10/11/63452e4608e41.png)

选择 无线安全 配置无线网密码，加密选择 `WPA2-PSK`

![无线网密码配置](https://img.braindance.top/artical/2022/10/11/63452e93b28bc.png)

无线网络部分配置完成。
### 配置管理后台密码

`192.168.1.1` 管理后台默认密码 `password` 容易被进入，进入 系统 ---> 管理权，可以更改访问后台管理员的密码。

## 校园网认证

**前置条件**：配置好 PPPoE 拨号

在前面我们已经成功配置了 PPPoE 拨号，但是要记得校园网还要发送心跳包保证在线状态。这部分主要解决发送心跳包的问题。

还需要说一下，发送心跳包的主要原理来自 [drcom-generic](https://github.com/drcoms/drcom-generic) 项目，广工大项目中使用的 Dr.com 插件是这个项目的 openwrt 插件版本，并且现在刷入的 openwrt 固件自带的也是这个插件，我并没用使用成功。最终是结合了学校贴吧老哥使用的 [dogcom](https://github.com/mchome/dogcom) 才成功，dogcom 则是前面那个项目的 C 语言实现版本。

### 删除 Dr.com 插件

在使用 dogcom 之前，需要把之前的 Dr.com 插件删除，因为会造成重复发包的问题导致无法认证。

进入 系统 ---> 软件包，在界面中筛选器部分搜索 drcom 或者 gdut （记不清了），然后选择 已安装列表，搜索到的软件包点击移除按钮，自动移除未使用的依赖可以取消勾选。

### 通过 openwrt 提供的 ssh 连接路由器

openwrt 安装成功以后其实就可以通过 ssh 连接路由器了，使用 MobaXterm 连接路由器。这时连接的地址是 `192.168.1.1`，用户名是 `root`，密码是你 openwrt 的**管理后台密码**

![ssh连接](https://img.braindance.top/artical/2022/10/11/6345354d1c8b6.png)

MobaXterm 使用 ssh 连接以后左边会自己创建一个 ftp 连接，就是图中的框框部分

![1665480198855.png](https://img.braindance.top/artical/2022/10/11/634535f1bcec4.png)

上面蓝色选中的部分是当前的路径，可以修改当前所在路径，通过拖拽可以直接上传文件

### 安装 dogcom

dogcom 安装方式有两种， 一种是使用 openwrt 版本的通过 opkg 软件包管理器安装，另一种是通过 ftp 上传 dogcom 可执行文件到 `/usr/bin/` 文件夹中。就算使用 opkg 软件包安装，两个最后的结果都是一样的，就是在 `/usr/bin/` 文件夹中有一个 dogcom 的可执行文件。

1. 方法一：使用 opkg 软件包安装

	还是在系统 ---> 软件包界面，有一个上传软件包按钮，点击上传下载的压缩包 `dogcom.zip` 中的 `dogcom_v1.6.2-1_mipsel_24kc.ipk`，之后执行安装即可。

	也可以通过 ftp 把文件上传到 `/tmp` 文件夹中，然后在控制台执行

```
opkg install /tmp/dogcom_v1.6.2-1_mipsel_24kc.ipk
```


2. 方法二：通过 ssh 上传到指定目录
在左侧的 ftp 界面上面路径输入 `/usr/bin` 进入文件夹，然后拖动名为 **dogcom** 的文件上传到该目录。



通过以上方法两个方法上传安装 dogcom 以后，可以在控制台执行

```
/usr/bin/dogcom -h
```

测试软件是否可用，软件输出为
```
root@iapp:~# /usr/bin/dogcom

Drcom-generic implementation in C.
Version: 1.6.2

Usage:
        dogcom -m <dhcp/pppoe> -c <FILEPATH> [options <argument>]...

Options:
        --mode <dhcp/pppoe>, -m <dhcp/pppoe>  set your dogcom mode
        --conf <FILEPATH>, -c <FILEPATH>      import configuration file
        --bindip <IPADDR>, -b <IPADDR>        bind your ip address(default is 0.0.0.0)
        --log <LOGPATH>, -l <LOGPATH>         specify log file
        --daemon, -d                          set daemon flag
        --802.1x, -x                          enable 802.1x
        --eternal, -e                         set eternal flag
        --verbose, -v                         set verbose flag
        --help, -h                            display this help
```

视为成功。

### 上传 dogcom 配置文件

可以参考上一节安装 dogcom 的方法二，使用 ftp 上传 `dogcom.conf` 文件到 `/usr/` 目录下。这个 `dogcom.conf` 内容是发送心跳包的配置文件，来源是通过 [drcom-generic](https://github.com/drcoms/drcom-generic) 项目教程，使用 Wireshark 软件进行抓包并使用 [在线配置器](http://drcoms.github.io/drcom-generic/) 获得的，如果以后校园网配置更改还需要自行抓包测试。`dogcom.conf` 的内容如下

```
server = '1.1.1.1'
pppoe_flag = '\x2f'
keep_alive2_flag = '\xdb'
```

### 配置 PPPoE 拨号文件

在 ssh 中按照顺序执行以下指令：

```
cp /lib/netifd/proto/ppp.sh /lib/netifd/proto/ppp.sh_bak
sed -i '/proto_run_command/i username=`echo -e "$username"`' /lib/netifd/proto/ppp.sh
sed -i '/proto_run_command/i password=`echo -e "$password"`' /lib/netifd/proto/ppp.sh
chmod 777 /usr/bin/dogcom
```

之后就可以执行

```
/usr/bin/dogcom -m pppoe -c /usr/drcom.conf -e -d &
```

dogcom 会自动启用一个守护进程发送心跳包认证，为了保证每次路由器重启以后自动连接校园网并认证，建议在 openwrt 管理页面的 系统 ---> 计划任务 中添加本地启动脚本

```
sleep 10 && /usr/bin/dogcom -m pppoe -c /usr/drcom.conf -e -d &
```

![1665481387645.png](https://img.braindance.top/artical/2022/10/11/63453a962923a.png)

### 查看校园网是否成功认证

配置完成后，重启路由器，并请耐心等待若干分钟（3分钟内），重新回到 openwrt 管理页面的 网络 ---> 接口中，查看 `WAN` 接口 PPPoE 是否拨号成功，如果运行时间、发送、接受均有数据，并且 IPv4 获得的一个地址，则说明路由器已经可以上网了。

![拨号成功](https://img.braindance.top/artical/2022/10/11/63453da03857e.png)

> wan中，学号密码输入错误。
> 
> 路由器的wan没有与校园网端口连接
> 
> 网线断了，或者路由器坏了
> 
> 压根没开通校园网
> 
> dogcom 插件中，校园网心跳配置已经更改
> 
> 端口被学校网络中心拉黑了


## 挖个坑

### 校园网防检测

常见的四种检测：

* 基于 IPv4 数据包包头内的 TTL 字段的检测（固定TTL）
* 基于 HTTP 数据包请求头内的 User-Agent 字段的检测(UA2F)
* DPI (Deep Packet Inspection) 深度包检测技术）（不常用）
* 基于 IPv4 数据包包头内的 Identification 字段的检测（rkp-ipid 设置 IPID）
* 基于网络协议栈时钟偏移的检测技术（防时钟偏移检测）
* Flash Cookie 检测技术（iptables 拒绝 AC 进行 Flash 检测 不常用）

大佬讲解文章：https://catalog.chn.moe/

广东工业大学项目：https://github.com/shengqiangzhang/Drcom-GDUT-HC5661A-OpenWrt#步骤六配置防检测

### 校园网经常掉线

2022.10.31 更新

自己从网上找了一个检测断网并自动重新拨号的脚本，配合定时任务每天凌晨 4 点重启，已经用了 20 多天了感觉还不错，分享一波代码。可以在任意目录下创建一个 ping 文件夹（但是需要自己改下某些配置路径），下面的例子是在 /root/ping 目录里放的脚本。脚本包括产生的日志有三个文件：
* ping.sh。每间隔 `SLEEP_SEC` 时间测试两个外网地址能否访问，超过 `PING_SUM` 次数无法访问判断为拨号掉线，重启 wan 口进行拨号。产生的日志文件存放到 `/root/ping/log.txt`
* daemon.sh。检测 ping.sh 进程是否存活，如果不存在进程则重启进程；判断日志文件超过 50MB 清空日志文件。

`ping.sh`
```shell
#!/bin/sh


PING_SUM=5

#ping interval
SLEEP_SEC=10

#连续重启网卡 REBOOT_CNT 次网络都没有恢复正常，重启软路由
#时间= (SLEEP_SEC * PING_SUM + 20) * REBOOT_CNT
REBOOT_CNT=30

LOG_PATH="/root/ping/log.txt"

cnt=0
reboot_cnt=0
while :
do
    ping -c 1 -W 1 114.114.114.114 > /dev/null
    ret=$?

    ping -c 1 -W 1 223.6.6.6 > /dev/null
    ret2=$?

    if [[ $ret -eq 0 || $ret2 -eq 0 ]]
    then
        echo 'Network OK!'
        cnt=0
        reboot_cnt=0
    else
        cnt=`expr $cnt + 1`
        echo -n `date '+%Y-%m-%d %H:%M:%S'` >> $LOG_PATH
        printf '-> [%d/%d] Network maybe disconnected,checking again after %d seconds!\r\n' $cnt $PING_SUM $SLEEP_SEC >> $LOG_PATH
        printf '-> [%d/%d] Network maybe disconnected,checking again after %d seconds!\r\n' $cnt $PING_SUM $SLEEP_SEC

        if [ $cnt == $PING_SUM ]
        then
            echo 'ifup wan!!!' >> $LOG_PATH
            echo 'ifup wan!!!'

            ifdown wan
            sleep 1
            ifup wan

            cnt=0
            #重连后，等待20秒再进行ping检测
            sleep 20


            #网卡重启超过指定次数还没恢复正常，重启软路由
            reboot_cnt=`expr $reboot_cnt + 1`
            if [ $reboot_cnt == $REBOOT_CNT ]
            then
                echo -n `date '+%Y-%m-%d %H:%M:%S'` >> $LOG_PATH
                echo '-> =============== reboot!' >> $LOG_PATH
                echo '-> =============== reboot!'

                sshpass -p 132465 ssh -p 22 root@192.168.1.1 'reboot'
            fi
        fi
    fi

    sleep $SLEEP_SEC
done
```

`daemon.sh`

```shell
#!/bin/sh

LOG_PATH="/root/ping/log.txt"

# 用ps获取ups进程数量
NUM=`ps | grep ping.sh | grep -v grep | wc -l`
echo ${NUM}

# 少于1，重启进程
if [ "${NUM}" -lt "1" ]
then
    /root/ping/ping.sh > /dev/null &
    echo -n `date '+%Y-%m-%d %H:%M:%S'` >> $LOG_PATH
    echo '-> Ping daemon start' >> $LOG_PATH
fi

s=`du -k /root/ping/log.txt|awk '{print $1}'`
if [ $s -gt 500000 ]
    then
    chengdatetime=`date "+%Y-%m-%d %H:%M:%S"`
    echo $chengdatetime":log size is large than expected and cleaning is started" >> $LOG_PATH
        cat /dev/null > /root/ping/log.txt
fi

exit 0


```

之后在 openwrt 的管理后台 ---> 系统 ---> 计划任务中添加
```shell
0 4 * * * reboot
0 */1 * * * /root/ping/daemon.sh
```
第一行是每天 4 点重启路由器，第二行是启动检测存活脚本（看好文件路径别错），可以自行设置计划运行时间。

---
分割线，以下是旧内容

---

这个我也遇到过了，不知道是什么原因，毕竟我自己用电脑连着认证时不时也会掉，但是也有搜到的下面的办法

https://blog.csdn.net/weixin_35251837/article/details/119553540

在 `/etc/ppp/options` 文件中添加 `persist`






## 完工

读到这里相信你也费了好大的力气了，也恭喜你，至少你是一个善于坚持的人，请享用你的校园网吧。有问题可以在评论区提问。