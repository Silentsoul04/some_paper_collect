> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/UTY1eUClbO7TTCj0I43O_A)

*   准备好上传的 shell.php 文件（一句话木马）
    

```
<?php @eval('phpinfo();'); ?>
```

第一关：前端 JS 验证
============

*   第一关是一个针对于前端的验证，也就是使用 JavaScript 进行验证。
    
*   先写一个 php 文件，你可以写一个一句话木马，也可以写一个 phpinfo，因为 phpinfo 看起来更加直接一点。我这里文件内容就写一个 phpinfo() 吧。
    
*   进入正题，选择我们要上传的 php 文件，我这里就以关卡来命名，点击上传，出现提示，这明显的是一个 js 的 alert 提示框。我们就来尝试进行绕过。
    
*   我们先上传一个文件上去看看效果，上传失败，说不允许上传 PHP 文件
    

![](https://mmbiz.qpic.cn/mmbiz_png/uOhQJgKzib22AGK1Lb8h8DFQhAO4sFxKVtDyiaIb6fZ2vquicO38fibglzloS91LfyGiay6EGVhAibPQcL6BYicib9e4ibw/640?wx_fmt=png) image.png

*   根据提示：本 pass 在客户端使用 js 对不合法图片进行检查！
    
*   可以得出是前端 JS 验证。
    
*   绕过方法：禁用 JS 脚本。
    
*   上传成功。
    

![](https://mmbiz.qpic.cn/mmbiz_png/uOhQJgKzib22AGK1Lb8h8DFQhAO4sFxKV0X58j3wiass9rSBFeVMJXtQS7rCias9kW0Leqj7a0k5xldXAEBsFQOoA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/uOhQJgKzib22AGK1Lb8h8DFQhAO4sFxKVFTG2ZOt5RgotQTjRjpUtlHxa8kF9M6DInzAsoVibEsLQYqzhJbASsHA/640?wx_fmt=png)

绕过原理：
-----

*   就是对上传文件进行一个 JavaScript 脚本的验证。屏蔽掉就行了。
    

```
function checkFile() {    var file = document.getElementsByName('upload_file')[0].value;  // 获取文件名    if (file == null || file == "") {        alert("请选择要上传的文件!");        return false;    }    //定义允许上传的文件类型    var allow_ext = ".jpg|.png|.gif";    //提取上传文件的类型    var ext_name = file.substring(file.lastIndexOf("."));    //判断上传文件类型是否允许上传    //通过lastIndexOf取到“.”的索引，再使用substring函数截取 .后缀名    if (allow_ext.indexOf(ext_name + "|") == -1) {    //如果 allow_ext 中没有 ext_name字符串，则返回-1        var errMsg = "该文件不允许上传，请上传" + allow_ext + "类型的文件,当前文件类型为：" + ext_name;        alert(errMsg);        return false;    }    //判断上传文件类型是否允许上传}
```

第二关：服务端 MIME 验证
===============

*   根据提示：本 pass 在服务端对数据包的 MIME 进行检查！
    
*   绕过方法：我们来抓个包来看看，修改一下`Content-Type`字段。
    
*   Content-Type: image/png
    

![](https://mmbiz.qpic.cn/mmbiz_png/uOhQJgKzib22AGK1Lb8h8DFQhAO4sFxKVsicjvCiaKluSoRvSrKz3QW4utku7prkS6ctzCUZWR5nwVLsaoqVbnruQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/uOhQJgKzib22AGK1Lb8h8DFQhAO4sFxKVe27AADMwFMvT03JOuGZUph6hdCosvzkrPmrru20icd85SRDTI8hYP8g/640?wx_fmt=png)

绕过原理：
-----

```
<?phpinclude '../config.php';include '../head.php';include '../menu.php';$is_upload = false;$msg = null;// 当点击鼠标上传的时候，进行验证if (isset($_POST['submit'])) {  // UPLOAD_PATH 声明在配置文件上的文件上传路径，来判断路径是否存在。    if (file_exists(UPLOAD_PATH)) {      // 进行上传文件类型判断， || 或的意思        if (($_FILES['upload_file']['type'] == 'image/jpeg') || ($_FILES['upload_file']['type'] == 'image/png') || ($_FILES['upload_file']['type'] == 'image/gif')) {            $temp_file = $_FILES['upload_file']['tmp_name'];            $img_path = UPLOAD_PATH . '/' . $_FILES['upload_file']['name'];           // move_uploaded_file 移动文件函数，将前者移动到后者哪里            if (move_uploaded_file($temp_file, $img_path)) {                $is_upload = true;            } else {                $msg = '上传出错！';            }        } else {            $msg = '文件类型不正确，请重新上传！';        }    } else {        $msg = UPLOAD_PATH.'文件夹不存在,请手工创建！';    }}?><?phpheader("Content-type: text/html;charset=utf-8");error_reporting(0);define("WWW_ROOT",$_SERVER['DOCUMENT_ROOT']);define("APP_ROOT",str_replace('\\','/',dirname(__FILE__)));define("APP_URL_ROOT",str_replace(WWW_ROOT,"",APP_ROOT));//文件包含漏洞页面define("INC_VUL_PATH",APP_URL_ROOT . "/include.php");//设置上传目录define("UPLOAD_PATH", "../upload");?>
```

第三关：黑名单绕过
=========

*   根据提示：本 pass 禁止上传. asp|.aspx|.php|.jsp 后缀文件！
    
*   比较容易想的绕过方法：
    

*   大小写绕过
    
*   用其他可被解析的后缀名来代替 : .phtml .phps .php5 .pht
    

*   大小写绕过失败！
    

![](https://mmbiz.qpic.cn/mmbiz_png/uOhQJgKzib22AGK1Lb8h8DFQhAO4sFxKVqH90dygqmOkxibdZZDh4NsfyP9zEAiawTXcd6zd3fhOe0B7fiaeCBVUDg/640?wx_fmt=png)image.png

*   尝试用其他后缀名试试：.php5
    

![](https://mmbiz.qpic.cn/mmbiz_png/uOhQJgKzib22AGK1Lb8h8DFQhAO4sFxKV9gicMia1KI7hnpaCmt5x3auujrH9wFfyqx8fG0jWtjtXXHmJa2CLkqQA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/uOhQJgKzib22AGK1Lb8h8DFQhAO4sFxKVrBsuv8EYl32mBITfBsyx3nrkgnr3xibccnfvCgfOfRHGO2NY6iah2SdQ/640?wx_fmt=png)

绕过原理：
-----

*   黑名单验证，只要避免啊上传这种后缀就可以正常的进行文件上传。
    
*   trim 函数详解
    
*   strrchr 函数详解
    

```
<?phpinclude '../config.php';include '../common.php';include '../head.php';include '../menu.php';$is_upload = false;$msg = null;if (isset($_POST['submit'])) {    if (file_exists(UPLOAD_PATH)) {      // 声明了变量 deny_ext（拒绝_后缀名）        $deny_ext = array('.asp','.aspx','.php','.jsp');      // trim 过滤为空的函数，替换空格。自动去除空格        $file_name = trim($_FILES['upload_file']['name']);      //trim去除字符创两侧的的特殊字符        $file_name = deldot($file_name);//删除文件名末尾的点，xiasohuang.jpg.php 防止上传验证为jpg格式实际上是php代码。      $file_ext = strrchr($file_name, '.');      // strrchr 分隔字符，就是删除掉.前面的文件名称，这样就得到了文件的真实后缀了。      //取出后缀名 如：.txt        $file_ext = strtolower($file_ext); //转换为小写        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA，将目标中的::$DATA替换为空。        $file_ext = trim($file_ext); //收尾去空      // 判断接收到的文件中，有没有黑名单里面的后缀，就继续进行文件上传，如果存在就不允许上传这个格式。        if(!in_array($file_ext, $deny_ext)) {            $temp_file = $_FILES['upload_file']['tmp_name'];            $img_path = UPLOAD_PATH.'/'.date("YmdHis").rand(1000,9999).$file_ext;       //获取当前时间 再连接上一个随机数和后缀名，生成一个新的文件名与上传路径拼接                if (move_uploaded_file($temp_file,$img_path)) {                 $is_upload = true;            } else {                $msg = '上传出错！';            }        } else {            $msg = '不允许上传.asp,.aspx,.php,.jsp后缀文件！';        }    } else {        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';    }}?>
```

*   网上搜了一下，原来需要对 apache 配置文件做修改，在 phpstudy 中点击 “其他选项菜单” 打开 Apache 配置文件`httpd-conf`。
    
*   ![](https://mmbiz.qpic.cn/mmbiz_png/uOhQJgKzib22AGK1Lb8h8DFQhAO4sFxKVeGiaB4hUicGPBtK3bzItWtVEYhnCLdOIicmNcR0Jl6icZnM2UzOlcbXrSw/640?wx_fmt=png)
    

![](https://mmbiz.qpic.cn/mmbiz_png/uOhQJgKzib22AGK1Lb8h8DFQhAO4sFxKVeYFaeJbe9KVBGWoKkCdcxde0vkpSdX4YY6OcJ1S0wMgBf9VtWwBrSw/640?wx_fmt=png)image.png

第四关：.htaccess 绕过
================

*   根据提示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/uOhQJgKzib22AGK1Lb8h8DFQhAO4sFxKVic5p6Uqkb5qHmt4rWSmMnCTmsckq8BrSlR0CqqMPSAppOJEWU4RriacQ/640?wx_fmt=png)image.png

*   绕过方法：
    

*   首先通过抓包来上传一个. htaccess，只上传. htaccess，不要文件名（因为. htaccess 是一个配置文件）
    
*   然后上传一个 shana.jpg 格式文件，为什么是（shana 因为这个文件是自己定义的）
    

![](https://mmbiz.qpic.cn/mmbiz_png/uOhQJgKzib22AGK1Lb8h8DFQhAO4sFxKVCvBV5F6PqDZv2Hh5pzOJll1icyYdO984BzcchBI61DnWCOiatloIke7g/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/uOhQJgKzib22AGK1Lb8h8DFQhAO4sFxKVgsNTE6bcbGMhXfAm2RqEFYxl2UIru8bKF57Q0wlmdrvPaS7nbzP9ibA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/uOhQJgKzib22AGK1Lb8h8DFQhAO4sFxKVyW604kOfz3YR9mCOKbuQ789rkuWPpNn33l4dFz62YG8tNJ2umKzouQ/640?wx_fmt=png)

绕过原理：
-----

*   这是解析漏洞 只有 apache 才有。
    
*   .htaccess 文件 (或者 "分布式配置文件"）, 全称是 Hypertext Access(超文本入口)。
    
*   提供了针对目录改变配置的方法， 即，在一个特定的文档目录中放置一个包含一个或多个指令的文件， 以作用于此目录及其所有子目录。作为用户，所能使用的命令受到限制。管理员可以通过 Apache 的 AllowOverride 指令来设置。
    
*   这个漏洞的原理就是服务器没有过滤 htaccess 文件的上传，而 htaccess 文件上传后，当前目录就会按照这个配置文件里面的内容执行。
    

```
<FilesMatch "自定义">Sethandler application/x-httpd-php</Eilesmatch >
```

*   然后上传 “自定义. 可以上传的后缀” 都会按照“自定义. php” 来执行
    
*   前提：只试用于 Apache 平台的伪静态转换，是 Apache 文件一个配置文件。
    
*   .htaccess 代码 ，保存到文件，文件类型 .htaccess
    
*   参考链接
    

```
<FilesMatch "shana">SetHandler application/x-httpd-php</FilesMatch>// FilesMatch 文件匹配 ，如果匹配到文件中存在 "shana" 就会将文件以解析 application/x-httpd-php 类型进行解析。
```

*   首先将这个文件（ .htaccess），进行上传，上传之后，再次解析文件的话就会以这个（刚才上传的配置文件为主）
    
*   然后将文件名中含有 "shana" 字段的图片进行上传，就会将文件以 php 代码进行解析了。（也可以在最后添加 phpinfo 进行回显）
    
*   就可以进行访问了。
    

第五关：.user.ini 绕过
================

*   user.ini 文件构成的 PHP 后门
    
*   根据提示：上传目录存在 php 文件（readme.php）
    

![](https://mmbiz.qpic.cn/mmbiz_png/uOhQJgKzib22AGK1Lb8h8DFQhAO4sFxKVhWliahAqoNQBiamWlMvrkicWs5a8RAYvnMsLXDla5zuL9StYjbLkeY1TQ/640?wx_fmt=png)image.png

*   绕过方法：
    

*   复写后缀名绕过
    
*   先上传`.user.ini`文件，然后再上传一个 5.jpg 文件，实现绕过。
    

```
.user.ini文件内容：auto_prepend_file=5.jpg
```

### 绕过方式一：

![](https://mmbiz.qpic.cn/mmbiz_png/uOhQJgKzib22AGK1Lb8h8DFQhAO4sFxKV3GX68ySGzgibicDicicoZL1AocrFMet9V3FXPca72hPqDJdlZnqgnYT1pg/640?wx_fmt=png)image.png

### 绕过方式二：

![](https://mmbiz.qpic.cn/mmbiz_png/uOhQJgKzib22AGK1Lb8h8DFQhAO4sFxKVkgkibxZPWVrXzmRoCdib5kwwQibSZ5GyrobRkdpYoJ9dGSINIoQicib7gHQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/uOhQJgKzib22AGK1Lb8h8DFQhAO4sFxKV84ctoRBT1gNzmjibIFGMUUNJ9iakFKbgETSica1xymNh8g1Rk1ib27hKgg/640?wx_fmt=png)

绕过原理：
-----

*   自 PHP 5.3.0 起，PHP 支持基于每个目录的 .htaccess 风格的 INI 文件。
    
*   此类文件仅被 CGI／FastCGI SAPI 处理。
    
*   此功能使得 PECL 的 htscanner 扩展作废。
    
*   如果使用 Apache，则用 .htaccess 文件有同样效果。
    
*   除了主 php.ini 之外，PHP 还会在每个目录下扫描 INI 文件，从被执行的 PHP 文件所在目录开始一直上升到 web 根目录（$_SERVER['DOCUMENT_ROOT'] 所指定的）。
    
*   如果被执行的 PHP 文件在 web 根目录之外，则只扫描该目录。
    
*   在 .user.ini 风格的 INI 文件中只有具有 PHP_INI_PERDIR 和 PHP_INI_USER 模式的 INI 设置可被识别。
    
*   两个新的 INI 指令，user_ini.filename 和 user_ini.cache_ttl 控制着用户 INI 文件的使用。
    
*   user_ini.filename 设定了 PHP 会在每个目录下搜寻的文件名；如果设定为空字符串则 PHP 不会搜寻。默认值是 .user.ini。
    
*   user_ini.cache_ttl 控制着重新读取用户 INI 文件的间隔时间。默认是 300 秒（5 分钟）。
    
*   **但是想要引发 .user.ini 解析漏洞需要三个前提条件：**
    

*   服务器脚本语言为 PHP
    
*   服务器使用 CGI／FastCGI 模式
    
*   上传目录下要有可执行的 php 文件
    

*   先来创建一个 .user.ini 文件并写入一下内容：auto_prepend_file=x.png
    
*   上传 .user.ini 后，再上传一个 x.png 文件，此时 x.png 文件只要有符合 php 语言的代码就会执行。
    

第六关：大小写绕过
=========

*   根据提示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/uOhQJgKzib22AGK1Lb8h8DFQhAO4sFxKVu5xDlFBwIibcxnk75YtBKIcI8o3c6aJufMfn7p6EZaAMdyZw6qexCYw/640?wx_fmt=png)image.png

*   绕过方法：
    

*   通过抓取上传数据包，修改上传的文件后缀，实现上传。
    

```
<?php$is_upload = false;$msg = null;if (isset($_POST['submit'])) {    if (file_exists(UPLOAD_PATH)) {        $deny_ext = array(".php",".php5",".php4",".php3",".php2",".html",".htm",".phtml",".pht",".pHp",".pHp5",".pHp4",".pHp3",".pHp2",".Html",".Htm",".pHtml",".jsp",".jspa",".jspx",".jsw",".jsv",".jspf",".jtml",".jSp",".jSpx",".jSpa",".jSw",".jSv",".jSpf",".jHtml",".asp",".aspx",".asa",".asax",".ascx",".ashx",".asmx",".cer",".aSp",".aSpx",".aSa",".aSax",".aScx",".aShx",".aSmx",".cEr",".sWf",".swf",".htaccess");        $file_name = trim($_FILES['upload_file']['name']);        $file_name = deldot($file_name);//删除文件名末尾的点        $file_ext = strrchr($file_name, '.');        $file_ext = strtolower($file_ext); //转换为小写      // 通过源代码我们可以发现，黑名单里虽然过滤的很全面，但是在下面的后缀名处理之中却出现了纰漏，没有将后缀名转换为小写。只是将文件名转化为小写        $file_ext = str_ireplace('::$DATA', '', $file_ext);//去除字符串::$DATA        $file_ext = trim($file_ext); //首尾去空                if (!in_array($file_ext, $deny_ext)) {            $temp_file = $_FILES['upload_file']['tmp_name'];            $img_path = UPLOAD_PATH.'/'.$file_name;            if (move_uploaded_file($temp_file, $img_path)) {                $is_upload = true;            } else {                $msg = '上传出错！';            }        } else {            $msg = '此文件类型不允许上传！';        }    } else {        $msg = UPLOAD_PATH . '文件夹不存在,请手工创建！';    }}?>
```

![](https://mmbiz.qpic.cn/mmbiz_png/uOhQJgKzib22AGK1Lb8h8DFQhAO4sFxKVprdvrcSfibzK7CRrdkuhP9OwicEneOdI8uUic4j48NDNFWI2LZMJ0kEjw/640?wx_fmt=png)image.png

绕过原理：
-----

```
$file_ext = strtolower($file_ext); //转换为小写// 通过源代码我们可以发现，黑名单里虽然过滤的很全面，但是在下面的后缀名处理之中却出现了纰漏，没有将后缀名转换为小写。只是将文件名转化为小写
```

第七关：空格绕过
========

*   根据提示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/uOhQJgKzib22AGK1Lb8h8DFQhAO4sFxKV5iaf3q5WdUpsBPR46v93aZRPCzPSlwardBLdPhSVqXuCC1e4juSd8SA/640?wx_fmt=png)image.png

*   绕过方法：
    

*   通过抓取上传数据包，修改上传的文件后缀，实现上传。
    

![](https://mmbiz.qpic.cn/mmbiz_png/uOhQJgKzib22AGK1Lb8h8DFQhAO4sFxKVg3XR94929Vc6vDlqw1b2oFEuJl3Arb2o0uiamujYNVUdvkI8tY7M0LQ/640?wx_fmt=png)image.png

绕过原理：
-----

*   在 windows 系统下，如果文件名以 “.” 或者空格作为结尾，系统会自动删除 “.” 与空格，利用此特性也可以绕过黑名单验证。apache 中可以利用点结尾和空格绕过，asp 和 aspx 中可以用空格绕过。
    

第八关：点绕过
=======

*   根据提示：本 pass 禁止上传所有可以解析的后缀！
    

![](https://mmbiz.qpic.cn/mmbiz_png/uOhQJgKzib22AGK1Lb8h8DFQhAO4sFxKVGLjJicQR3nFqCQpmqIDDzqslt6F62Ivroh9HcTwArFngenDA7uM1Ztg/640?wx_fmt=png)image.png

*   绕过方法：
    

*   通过抓取上传数据包，修改上传的文件后缀，实现上传。
    

![](https://mmbiz.qpic.cn/mmbiz_png/uOhQJgKzib22AGK1Lb8h8DFQhAO4sFxKVsHg9ibictYTA1H0ht9iauw96U4m62mPV4PBIBQ8SkKboztIy6cooasZAg/640?wx_fmt=png)image.png

绕过原理：
-----

*   在 windows 系统下，如果文件名以 “.” 或者空格作为结尾，系统会自动删除 “.” 与空格，利用此特性也可以绕过黑名单验证。apache 中可以利用点结尾和空格绕过，asp 和 aspx 中可以用空格绕过。
    

第九关：::$DATA 绕过
==============

*   根据提示：
    

![](https://mmbiz.qpic.cn/mmbiz_png/uOhQJgKzib22AGK1Lb8h8DFQhAO4sFxKVnazG1kyWkrFZbBF9ibcPf4ngfib6nrrXVzjGfNwkxQe73fTkT9ukZ7rQ/640?wx_fmt=png)image.png

*   绕过方法：
    

*   通过抓取上传数据包，修改上传的文件后缀，实现上传。
    

![](https://mmbiz.qpic.cn/mmbiz_png/uOhQJgKzib22AGK1Lb8h8DFQhAO4sFxKVVoC5PPQH7ia0jvbbLjTM92h9TvNibhuj0ibeeyz6j8xNqze9x5aFwMMEQ/640?wx_fmt=png)image.png

绕过原理：
-----

*   与上面题相比呢这个道题去掉了：DATA','', 去除字符串 DATA 这行代码。
    
*   ::DATA" 会把:: 之后的数据当成文件流处理不会检测后缀名且持 DATA" 之前的文件名
    
*   他的目的就是不检查后缀名....
    

第十关：点 + 空格 + 点绕过
================

*   根据提示：本 pass 只允许上传. jpg|.png|.gif 后缀的文件！
    

![](https://mmbiz.qpic.cn/mmbiz_png/uOhQJgKzib22AGK1Lb8h8DFQhAO4sFxKVkyNTnYR531Vq5CvudlXGln6xZZ4GL5I7wibMAGfTxz0Mg5XibQiaKeYBw/640?wx_fmt=png)image.png

*   绕过方法：
    

*   通过抓取上传数据包，修改上传的文件后缀，实现上传。
    

![](https://mmbiz.qpic.cn/mmbiz_png/uOhQJgKzib22AGK1Lb8h8DFQhAO4sFxKVYibh4xkgeibHRPJeHx2zfTYQ1ooO4akaHRzRLIz952fWBjlpvK4LOibOQ/640?wx_fmt=png)image.png

绕过原理：
-----

*   在 windows 系统下，如果文件名以 “.” 或者空格作为结尾，系统会自动删除 “.” 与空格，利用此特性也可以绕过黑名单验证。apache 中可以利用点结尾和空格绕过，asp 和 aspx 中可以用空格绕过。
    
*   经过脚本一系列的处理之后原本. php. . 的后缀名变成了. php. ，而由于 Windows 的特性，又将文件末尾的点给去除了，最终就存的时候. php 的文件。同理也可以上传. htaccess. . 等文件。。。(就算没有经过脚本的处理,.php. . 在 windows 中也是会被存储为. php)