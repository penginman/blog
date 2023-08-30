---
title: JetBrains 全家桶破解
date: 2023-08-30
categories: ['瞎折腾']
tags: ['JetBrains 激活']
draft: false
---

突然想开发了我的 IDEA 还是 2021 的，IDEA 和 Pycharm 还在机械硬盘，装个最新的到固态里面。教育邮箱懒得申请了，失效了又要麻烦续杯。写出来记录一下以后自己参考。

## 准备

破解使用的是 [ja-netfilter](https://gitee.com/ja-netfilter/ja-netfilter)：https://gitee.com/ja-netfilter/ja-netfilter。release 下载以后解压。

![1693410710464.png](https://img.braindance.top/artical/2023/08/30/64ef657904e57.png)

`config` 配置文件默认为空，需要添加一些配置。plugins 是可以加载一些其他联动插件，本文使用的方法不用其他插件。

找到 IDEA 或者 Pycharm 的 `Help` ---> `Edit Custom VM Options` 添加启动参数，路径是`ja-netfilter`的 jar 包路径。

```
-javaagent:C:\\path\\to\\ja-netfilter.jar
```

**Jetbrain全家桶在2022.2版本以上默认启用Java17**，所以用的时候需要在 VM Options 里额外增加下面两行参数。

```
--add-opens=java.base/jdk.internal.org.objectweb.asm=ALL-UNNAMED
--add-opens=java.base/jdk.internal.org.objectweb.asm.tree=ALL-UNNAMED
```

在这里修改的 VM Options 配置文件路径是 C 盘下的本版本全局配置，也可以修改添加到安装目录下的 bin 目录中的 `idea64.exe.vmoptions` 配置文件。

## 修改其他配置

修改 `config` 修改文件夹下的

`congig/url.conf`

```
[URL]
PREFIX,https://account.jetbrains.com/lservice/rpc/validateKey.action
```

`congig/dns.conf`

```
[DNS]
EQUAL,jetbrains.com
```

## 激活

### 使用 power 插件

power 插件被作者成为非对称加密的屠龙刀，这种方式激活可以自定义信息，并且可以设置全家桶激活。

参考大佬[博客文章](https://xuzhengtong.com/2022/07/25/ja-netfilter/ja-netfilter-plugins-power/) 本地运行两个 python 文件，一个用于生成本地证书签名文件

```python
import datetime

from cryptography import x509
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import hashes, serialization
from cryptography.hazmat.primitives.asymmetric import rsa
from cryptography.x509.oid import NameOID

one_day = datetime.timedelta(days=1)
ten_day = datetime.timedelta(days=3650)
today = datetime.datetime.today()
yesterday = today - one_day
tomorrow = today + ten_day

private_key = rsa.generate_private_key(
    public_exponent=65537,
    key_size=4096,
    backend=default_backend()
)
public_key = private_key.public_key()
builder = x509.CertificateBuilder()

builder = builder.subject_name(x509.Name([
    x509.NameAttribute(NameOID.COMMON_NAME, 'MoYuno-from-2022-07-25'),
]))
builder = builder.issuer_name(x509.Name([
    x509.NameAttribute(NameOID.COMMON_NAME, 'JetProfile CA'),
]))
builder = builder.not_valid_before(yesterday)
builder = builder.not_valid_after(tomorrow)
builder = builder.serial_number(x509.random_serial_number())
builder = builder.public_key(public_key)

certificate = builder.sign(
    private_key=private_key, algorithm=hashes.SHA256(),
    backend=default_backend()
)
private_bytes = private_key.private_bytes(
    encoding=serialization.Encoding.PEM,
    format=serialization.PrivateFormat.TraditionalOpenSSL,
    encryption_algorithm=serialization.NoEncryption())
public_bytes = certificate.public_bytes(
    encoding=serialization.Encoding.PEM)
with open("ca.key", "wb") as fout:
    fout.write(private_bytes)
with open("ca.crt", "wb") as fout:
    fout.write(public_bytes)
```

还有一个生成 key 和伪造验签

```py
import base64

from Crypto.Hash import SHA1, SHA256
from Crypto.PublicKey import RSA
from Crypto.Signature import pkcs1_15
from Crypto.Util.asn1 import DerSequence, DerObjectId, DerNull, DerOctetString
from Crypto.Util.number import ceil_div
from cryptography import x509
from cryptography.hazmat.primitives import hashes
from cryptography.hazmat.primitives.asymmetric import padding


# noinspection PyTypeChecker
def pkcs15_encode(msg_hash, emLen, with_hash_parameters=True):
    """
    Implement the ``EMSA-PKCS1-V1_5-ENCODE`` function, as defined
    :param msg_hash: hash object
    :param emLen: int
    :param with_hash_parameters: bool
    :return: An ``emLen`` byte long string that encodes the hash.
    """
    digestAlgo = DerSequence([DerObjectId(msg_hash.oid).encode()])

    if with_hash_parameters:
        digestAlgo.append(DerNull().encode())

    digest = DerOctetString(msg_hash.digest())
    digestInfo = DerSequence([
        digestAlgo.encode(),
        digest.encode()
    ]).encode()

    # We need at least 11 bytes for the remaining data: 3 fixed bytes and
    # at least 8 bytes of padding).
    if emLen < len(digestInfo) + 11:
        raise TypeError("Selected hash algorithm has a too long digest (%d bytes)." % len(digest))
    PS = b'\xFF' * (emLen - len(digestInfo) - 3)
    return b'\x00\x01' + PS + b'\x00' + digestInfo


certBase64 = "MIIFTDCCAzSgAwIBAgIBDTANBgkqhkiG9w0BAQsFADAYMRYwFAYDVQQDDA1KZXRQcm9maWxlIENBMB4XDTIyMDcyNTIzMTcwOVoXDTMyMDcyMzIzMTcwOVowHzEdMBsGA1UEAwwUcHJvZDJ5LWZyb20tMjAyMDEwMTkwggIiMA0GCSqGSIb3DQEBAQUAA4ICDwAwggIKAoICAQDDx3gz77KvezmZJhwkF/10Q3vESk96tK6wJ00CSKkLybRDeQVOlHX3QAnPL7BjwCTzHqErsuyPuiZ6YTAVE/n7hLhIbh3lC+EBbxpa2hpIdIvUimr70iSrH9ZBWmnn5Fxy4r/r0tbxr34zpQzu4uWLiEqmOiDfRN+Zzj9FBaJ/gKsuhF7zNAbFHsClYntim5furDRITBra28nu0hfQIEBSHGPS2EKWTbKk2ifBLzMEDp99zIGEe/hrLpgBhdwGVD7VJsoeTXnvcgpt+91kiM918GWThO1L3eKU6W2mGZQv1bRyps7Fo61NElNWtJqqZ3KKyxJGyR1QpdOHd9flAesvYwb/lvc4uqYiKqwvvn+4iHPQlLtZDbzj0ICbKtVKSWgSprh0T5ZQGGNWXN4OMHtg9EuXvbagLshTEDkLKLzEBqSNpNmMmyzwyNO9/voQmHLjiWLdjVIYndjl15G+A9Dw5mVYqzKPMLEpHzg6ldkKJkGAxNBhCMUsmbEypT6r7wsdTvgEwFnP8ToOsAb12lSLxoR2bOT3xJ3WIfbyjvlBnauXfdu6SFF/82QFrLtQyddPvCHEiJTI0NmSYhjQObFohXMVVoXjGbXvuqgJNbc5UK06pCGQ2jKw4j6k1kw2g4fEYBd1fvEzb1/t+izpP8dEI0365xh0C1dpQjUj3uyRywIDAQABo4GZMIGWMEgGA1UdIwRBMD+AFKOetkhnQhI2Qb1t4Lm0oFKLl/GzoRykGjAYMRYwFAYDVQQDDA1KZXRQcm9maWxlIENBggkA0myxg7KDeeEwCQYDVR0TBAIwADATBgNVHSUEDDAKBggrBgEFBQcDATALBgNVHQ8EBAMCBaAwHQYDVR0OBBYEFCTaESKW9YVBwJNH6DEjTPTAhAL/MA0GCSqGSIb3DQEBCwUAA4ICAQA29wUDKatiQe1S0qfId+1dRWnYznrHE0Cx41HUaeI5hvdZrFbDIP6syb/S9oAXST6w4pfgh80jk1xVL+B7NT5kFC+AI7mpd8dK8Z+K67tagYg41TdLGfSHqK+lljln5ElqUEN21fba5CVZplE286jy973XFOFbWZUpJC/5onCCAh8pK8AqpN7k3ovR6bfAga41UWdTnGeiyw9+XOj30ryebseTKaDfjQxsxEmyuA8YYCu9lgb58cvVrvc99So8KdOBaxHnxeEfiUqvPA8Y0QG7lc5elZYQ6cbiIqqsb/k9XSgB2Gk4CjuacBSxCAfd06NlJvZSDFSR1HTKhQfPLIQY1OpBC+NrKRWnQT4/IORL6F36gI9lTK+ioX8mzQ2bvXn4sXA3jrpRnGM2WemQvMPvstfSDKfcUdKjwX3rZ2jMwREkx/thtF3Huvsc8suOyzto1faD8mV0m4guq85fb4c9ki6cinz3QM2k6otVvh67gK116RZ7I8P/urTWvK7IOdwOE7UVqtpEe6TKvNhr1rzeaxUMdPcD0kY7fhBpuPwEQA+Xk0uiVR+XbpaPD4HWuapJm+31jC7zBp/BamRI25v26P5qMUQF/+P7eE4Ah/X0Rtf2Qvr2+p9kbfqalT8EiqOsvRiTvlMG1hdo33JdcwsxC05BWvZ++7Af0FgJ3TtFlw=="

cert = x509.load_der_x509_certificate(base64.b64decode(certBase64))
public_key = cert.public_key()
sign = int.from_bytes(cert.signature, byteorder="big", )
print(f"sign:{sign}")

modBits = public_key.key_size
digest_cert = SHA256.new(cert.tbs_certificate_bytes)
r = int.from_bytes(pkcs15_encode(digest_cert, ceil_div(modBits, 8)), byteorder='big', signed=False)
print(f"result:{r}")

licenseId = 'R7FP0YWA38'
licensePart = '{"licenseId":"R7FP0YWA38","licenseeName":"Braindance","assigneeName":"","assigneeEmail":"","licenseRestriction":"","checkConcurrentUse":false,"products":[{"code":"DPN","paidUpTo":"2077-01-01","extended":false},{"code":"DB","paidUpTo":"2077-01-01","extended":false},{"code":"PS","paidUpTo":"2077-01-01","extended":false},{"code":"II","paidUpTo":"2077-01-01","extended":false},{"code":"RSC","paidUpTo":"2077-01-01","extended":true},{"code":"GO","paidUpTo":"2077-01-01","extended":false},{"code":"DM","paidUpTo":"2077-01-01","extended":false},{"code":"RSF","paidUpTo":"2077-01-01","extended":true},{"code":"DS","paidUpTo":"2077-01-01","extended":false},{"code":"PC","paidUpTo":"2077-01-01","extended":false},{"code":"RC","paidUpTo":"2077-01-01","extended":false},{"code":"CL","paidUpTo":"2077-01-01","extended":false},{"code":"WS","paidUpTo":"2077-01-01","extended":false},{"code":"RD","paidUpTo":"2077-01-01","extended":false},{"code":"RS0","paidUpTo":"2077-01-01","extended":false},{"code":"RM","paidUpTo":"2077-01-01","extended":false},{"code":"AC","paidUpTo":"2077-01-01","extended":false},{"code":"RSV","paidUpTo":"2077-01-01","extended":true},{"code":"DC","paidUpTo":"2077-01-01","extended":false},{"code":"RSU","paidUpTo":"2077-01-01","extended":false},{"code":"DP","paidUpTo":"2077-01-01","extended":true},{"code":"PDB","paidUpTo":"2077-01-01","extended":true},{"code":"PWS","paidUpTo":"2077-01-01","extended":true},{"code":"PSI","paidUpTo":"2077-01-01","extended":true},{"code":"PPS","paidUpTo":"2077-01-01","extended":true},{"code":"PCWMP","paidUpTo":"2077-01-01","extended":true},{"code":"PGO","paidUpTo":"2077-01-01","extended":true},{"code":"PPC","paidUpTo":"2077-01-01","extended":true},{"code":"PRB","paidUpTo":"2077-01-01","extended":true},{"code":"PSW","paidUpTo":"2077-01-01","extended":true},{"code":"RS","paidUpTo":"2077-01-01","extended":true}],"metadata":"0120211210PPAM000005","hash":"28822622/0:1202205338","gracePeriodDays":7,"autoProlongated":false,"isAutoProlongated":false}'

digest = SHA1.new(licensePart.encode('utf-8'))

with open('ca.key') as prifile:
    private_key = RSA.import_key(prifile.read())
    # 使用私钥对HASH值进行签名
    signature = pkcs1_15.new(private_key).sign(digest)

    sig_results = base64.b64encode(signature)
    licensePartBase64 = base64.b64encode(bytes(licensePart.encode('utf-8')))
    public_key.verify(
        base64.b64decode(sig_results),
        base64.b64decode(licensePartBase64),
        padding=padding.PKCS1v15(),
        algorithm=hashes.SHA1(),
    )
    result = licenseId + "-" + licensePartBase64.decode('utf-8') + "-" + sig_results.decode('utf-8') + "-" + certBase64
    print(result)
```

安装 Crypto 依赖使用 `pip install pycrytodome`。**需要修改变量**  `certBase64 `  **为第一个文件生成的 cert 证书内容（自行删除换行）**。其中变量 `licensePart` 中的信息和变量 `licenseId` 对应，`licenseeName` 可以自行修改，`code` 应该就是全家桶各个软件的缩写，`paidUpTo` 过期时间。

配置文件`congig/power.conf` 的格式

```
[Result]
EQUAL,sign,y,z->result
```

`sign` 和 `result` 分别对应第二个文件的两行输出，分别是签名密文、证书签名。`y`,`z` 分别是 RSA 中的指数、jetbrains内置root证书的公钥（**不用修改**）。最终配置文件。

```
[Result]
EQUAL,688827393930711928512275549698293070665686146516966052655941231404870441973710402205123355604429394140415171936010826276494797379204430556181912308828215621834023869095264591943036963130120724420798769592253168800129581361086511367675773802899334078239434964778238252446370058840916752890687348786357248109407287360623267817632253915161513374442235501450990684679388233149022805420822037547053343732736438118829252210572948271275369220730128852181626787073863828515056541044882335869396696141056207575568411139674823212977124270019967190277434142980781559286683916236429191621661925978231096547985871015033045347745040517507548703203423373963474065307957598679613534182723075187728429764246533540129748262856350129370323776549193166705032852633517719905394268849453593835332705268187404502153581134679736820933961668519544538659820375073084965933956885058156852851457008982063229683626311524790625910341414580691932545385821852904086377007435193707757250435137675275183055336401236456974574121655434382553698002922301524374402422775517514284490136029700408044713357398902530280387081498510385206614656124276242043287045844898682620475564484729941780647067683830306648941012819834344380065067184504095694554818053932782057955011188082,65537,860106576952879101192782278876319243486072481962999610484027161162448933268423045647258145695082284265933019120714643752088997312766689988016808929265129401027490891810902278465065056686129972085119605237470899952751915070244375173428976413406363879128531449407795115913715863867259163957682164040613505040314747660800424242248055421184038777878268502955477482203711835548014501087778959157112423823275878824729132393281517778742463067583320091009916141454657614089600126948087954465055321987012989937065785013284988096504657892738536613208311013047138019418152103262155848541574327484510025594166239784429845180875774012229784878903603491426732347994359380330103328705981064044872334790365894924494923595382470094461546336020961505275530597716457288511366082299255537762891238136381924520749228412559219346777184174219999640906007205260040707839706131662149325151230558316068068139406816080119906833578907759960298749494098180107991752250725928647349597506532778539709852254478061194098069801549845163358315116260915270480057699929968468068015735162890213859113563672040630687357054902747438421559817252127187138838514773245413540030800888215961904267348727206110582505606182944023582459006406137831940959195566364811905585377246353->31872219281407242025505148642475109331663948030010491344733687844358944945421064967310388547820970408352359213697487269225694990179009814674781374751323403257628081559561462351695605167675284372388551941279783515209238245831229026662363729380633136520288327292047232179909791526492877475417113579821717193807584807644097527647305469671333646868883650312280989663788656507661713409911267085806708237966730821529702498972114194166091819277582149433578383639532136271637219758962252614390071122773223025154710411681628917523557526099053858210363406122853294409830276270946292893988830514538950951686480580886602618927728470029090747400687617046511462665469446846624685614084264191213318074804549715573780408305977947238915527798680393538207482620648181504876534152430149355791756374642327623133843473947861771150672096834149014464956451480803326284417202116346454345929350148770746553056995922154382822307758515805142704373984019252210715650875853634697920708113806880196144197384637328982263167395073688501517286678083973976140696077590122053014085412828620051470085033364773099146103525313018873319293728800442101520384088109603555959893639842091339193872012790186223768662175794579863414151470043973965016829149915484844451483261
```

第二个文件还有一行输出就是 key，重启软件激活即可

![1693412928011.png](https://img.braindance.top/artical/2023/08/31/64ef6e21d9a6a.png)