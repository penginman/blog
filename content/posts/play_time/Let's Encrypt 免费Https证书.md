---
title: Let_s_Encrypt 免费Https证书
date: 2020-09-03 15:30:37
categories: ["Play"]
tag: ["Https证书"]
---

参考文章:[Let's Encrypt，免费好用的 HTTPS 证书][Let's Encrypt，免费好用的 HTTPS 证书]

### 先放官网

[Let's Encrypt](https://letsencrypt.org/)

> [Let's Encrypt](https://letsencrypt.org/) 是免费、自动化、开放的证书签发服务, 它得到了 Mozilla、Cisco、Akamai、Electronic Frontier Foundation 和 Chrome 等众多公司和机构的支持，发展十分迅猛

---

#### 所需环境

- 一个 HTTP 服务，以 Nginx 为例

- python

- 两个目录:

  > /site 网站目录
  >
  > /site_site_cert 保存证书的目录

**证书的颁发有两种方式：**

#### 通过在线生成

通过网址在线生成，需要验证文件

#### 通过程序生成

通过本地 IIS，生成后会自动绑定本地 IIS 站点上的 HTTPS 域名。

我使用的是第一种方法：在线生成，原理是：先在你的服务器上传一个密钥，然后 Let's Encrypt 会对网站进行访问，下载密钥进行验证。

---

### 开工

### 创建账号

首先创建一个目录，我是在根目录下创建文件夹 site_cert

```shell
mkdir /site_cert
```

这个文件夹用来存放各种临时文件和最后的证书文件。进入这个目录，创建一个 RSA 私钥，用于 Let's Encrypt 识别你的身份

```shell
openssl genrsa 4096 > account.key
```

### 创建 CSR 文件

在这之前，还需要创建域名私钥（一定不要使用上面的账户私钥）

```shell
openssl genrsa 4096 > domain.key
```

我参考的文章提到了两种私钥 RSA 和 ECC，我现在也还不懂什么原理，把两种区别放出来吧

> RSA 私钥：兼容性好
>
> ECC 私钥：部分老旧操作系统、浏览器不支持。优点是证书体积小

两个用于身份身份验证的私钥文件创建好，就可以生成 CSR（Certificate Signing Request，证书签名请求）文件了，申请时可以把域名带 `www` 和不带 `www` 的两种情况都加进去，一张证书最多可以包含 100 个域名。

```shell
openssl req -new -sha256 -key domain.key -subj "/" -reqexts SAN -config <(cat /etc/ssl/openssl.cnf <(printf "[SAN]\nsubjectAltName=DNS:yoursite.com,DNS:www.yoursite.com")) > domain.csr
```

- 其中 DNS 的`yoursite.com`和`www.yoursite.com`记得要换成自己的域名

我在这里遇到了问题提示找不到`/etc/ssl/openssl.cnf`文件，在网上找的的[解决办法][linux 使用openssl报找不到/usr/lib/ssl/openssl.cnf的解决办法]是

执行 :

```shell
openssl version -a
```

会输出 openssl 的信息，其中`OPENSSLDIR`对应的路径就是`openssl.cnf`文件的地址，自行将上面的`cat /etc/ssl/openssl.cnf`,改为自己的路径运行。

### 配置验证服务

前面介绍过了 Let's Encrypt 验证的原理是在你的服务器上生成一个随机文件，在通过创建 CSR 时的域名进行访问下载，如果成功表明你对这个域名的拥有权。

创建用于存放网站的目录 site 以及用于验证文件存放的子目录

```shell
mkdir -p /site/.well-known/acme-challenge/
```

然后再 Nginx 中配置:

```nginx
server {
    server_name www.yoursite.com yoursite.com;

    location ^~ /.well-known/acme-challenge/ {
        alias /home/xxx/www/challenges/;
        try_files $uri =404;
    }

    location / {
        rewrite ^/(.*)$ https://yoursite.com/$1 permanent;
    }
}
```

- 别忘了改`yoursite`

这个配置会优先查找`/site`目录下的网站，建议保留以后证书认证还可以用到，因为颁发的证书一次可以使用 90 天。

### 获取网站证书

先下载`acme-tiny`脚本到之前的 site_cert 目录：

```shell
wget https://raw.githubusercontent.com/diafygi/acme-tiny/master/acme_tiny.py
```

指定账户私钥、CSR 以及网站上验证文件的目录，执行脚本:

```shell
python acme_tiny.py --account-key ./account.key --csr ./domain.csr --acme-dir /fakesite/.well-known/acme-challenge/ > ./signed.crt
```

执行成功的话会在当前目录生成一个`signed.crt`文件，这个文件就是申请好的证书文件。

我在这里出现了错误提示

```
ValueError: Wrote file to /site/.well-known/acme-challenge/blablabla, but couldn't download http://www.yoursite.com/.well-known/acme-challenge/blablabla
```

大概的意思就是，在网站目录里写入了一个验证文件，但是 Let's Encrypt 的服务器访问不到你的网站，建议先去看一看 Nginx 配置是否出错，再有可能是自己的域名无法在国外解析，建议暂时使用国外的 DNS 解析商。推荐的有：

> [Hurricane Electric Free DNS](https://dns.he.net/)
>
> [ZoneEdit](https://www.zoneedit.com/)
>
> [CloudFlare](https://www.cloudflare.com/)

这些都是免费的，但是因为我自己的域名后缀为.tk，上面第一个 DNS 解析商警告因为.tk 域名滥用，不给解析。

网站证书到手以后，还要下载 Let's Encrypt 的中间证书。证书链中大部分都是「站点证书 – 中间证书 – 根证书」这样三级。服务端只需要发送前两个证书即可。我们需要把中间证书和网站证书合在一起：

```shell
wget -O - https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem > intermediate.pem


cat signed.crt intermediate.pem > chained.pem
```

**最终**，在 Nginx 中添加证书配置，并 reload，我的部分配置如下

```nginx
server {
   listen 443 ssl;
   ssl_certificate       /site_cert/chained.pem;
   ssl_certificate_key   /site_cert/domain.key;
   ssl_protocols         TLSv1 TLSv1.1 TLSv1.2;
   ssl_ciphers           HIGH:!aNULL:!MD5;
   server_name           braindance.tk;
   index index.html index.htm;
   root
        …………………………
}
```

执行

```shell
nginx -s reload
```

### 证书自动更新 计划任务

​ 至此我们已经成功的获取到了 Https 证书，但是获取到的 Https 证书只有 90 天的时效，到期的话还需要使用相同的方法进行更新，为了避免某次忘记更新导致网站出现问题，我们可以使用 linux 中的 crond 服务为我们自动更新证书。

用 `vi` 在 `/site_cert` 文件夹 创建 计划任务脚本 `renew_cert.sh`

```shell
vi /site_cert/renew_cert.sh
```

通过`vi`输入以下内容

```shell
#!/bin/bash

cd /fakesite_cert/
python acme_tiny.py --account-key ./account.key --csr ./domain.csr --acme-dir /fakesite/.well-known/acme-challenge/ > ./signed.crt || exit
wget -O - https://letsencrypt.org/certs/lets-encrypt-x3-cross-signed.pem > intermediate.pem
cat signed.crt intermediate.pem > chained.pem
nginx -s reload
```

更新的大致过程是，运行 python 脚本再次更新`signed.crt`申请证书文件，再进行证书合并写入`chained.pem`文件。

然后给这个文件赋予 可执行 属性

```shell
chmod +x /fakesite_cert/renew_cert.sh
```

使用`crontab -e`指令打开定时任务配置文件，并加入以下内容。

```
0 0 1 * * /home/xxx/shell/renew_cert.sh >/dev/null 2>&1
```

对于上面指令的具体意思请自行搜索`crontab`命令

### 完工

[Let's Encrypt，免费好用的 HTTPS 证书]: https://imququ.com/post/letsencrypt-certificate.html
[linux 使用openssl报找不到/usr/lib/ssl/openssl.cnf的解决办法]: https://blog.csdn.net/hjxdreamer/article/details/103296944
