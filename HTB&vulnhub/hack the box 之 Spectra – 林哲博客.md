> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.lz80.com](https://www.lz80.com/21423.html)

> Spectra 介绍 探测 访问 10.10.10.229:80 端口 找到 spectra.htb and www.spectra.htb 反向代理 http://spectra.htb/main/ http…......

![](https://image.3001.net/images/20210415/1618467554_6077dae257c04e794e2a8.png!small?1618467550396)

探测
--

![](https://image.3001.net/images/20210415/1618467895_6077dc37463ccdf841be5.png!small?1618467891159)

访问 10.10.10.229:80 端口

![](https://image.3001.net/images/20210415/1618467969_6077dc81665b168204d31.png!small?1618467965245)

找到 spectra.htb and www.spectra.htb 反向代理

http://spectra.htb/main/  
http://www.spectra.htb/testing/index.php

在 / etc/hosts 中添加 spectra.htb

![](https://image.3001.net/images/20210415/1618468247_6077dd97b448ed890b382.png!small?1618468243634)

再用目录猜解工具 gobuster 扫域名，这是一个扫描目录文件、DNS 和 VHost 爆破工具。

![](https://image.3001.net/images/20210415/1618468463_6077de6fea6be4bb793e5.png!small?1618468459805)

进一步的

![](https://image.3001.net/images/20210415/1618468484_6077de84af0fbf95face8.png!small?1618468480604)

![](https://image.3001.net/images/20210415/1618468527_6077deaf144f51a099ff0.png!small?1618468522993)

testing 路径下有一些 php 文件，我们主要看看有没有一些配置信息。

![](https://image.3001.net/images/20210415/1618468613_6077df05261b0179e2812.png!small?1618468609101)

发现 wp-config.php.save 文件里有配置信息，发现了数据库用户名密码

![](https://image.3001.net/images/20210415/1618468713_6077df694eb8ceee56247.png!small?1618468709297)

![](https://image.3001.net/images/20210415/1618468770_6077dfa24608f2820d685.png!small?1618468766173)

发现不允许这个 ip 链接数据库。再用 administrator / devteam01 尝试登录

![](https://image.3001.net/images/20210415/1618468911_6077e02f6f5f85b6a7e96.png!small?1618468907321)![](https://image.3001.net/images/20210415/1618468873_6077e0096b417f7910bb2.png!small?1618468869399)

发现是 wordpress 5.4.2 版本。

渗透测试反弹 shell
------------

访问访问管理员面板后，可以创建反弹 shell。

下载 php 反弹代码：http://pentestmonkey.net/tools/php-reverse-shell/php-reverse-shell-1.0.tar.gz

在如下页面填写代码

![](https://image.3001.net/images/20210415/1618469122_6077e1028f20f50e58075.png!small?1618469119311)

![](https://image.3001.net/images/20210415/1618469138_6077e1121dac76613bfd1.png!small?1618469134675)

还可以参考下面链接

https://www.hackingarticles.in/wordpress-reverse-shell/

https://rioasmara.com/2019/02/25/penetration-test-wordpress-reverse-shell/

kali 监听

![](https://image.3001.net/images/20210415/1618469276_6077e19c13e9d3acb7052.png!small?1618469272032)

访问已编辑 php 页面 http://spectra.htb/main/wp-content/themes/twentyseventeen/404.php

反弹 shell

![](https://image.3001.net/images/20210415/1618469394_6077e212f213c3fc0e699.png!small?1618469390860)

![](https://image.3001.net/images/20210415/1618469438_6077e23e17e29fe2d981b.png!small?1618469433966)

![](https://image.3001.net/images/20210415/1618469574_6077e2c6494f86b7a0e3d.png!small?1618469570282)

![](https://image.3001.net/images/20210415/1618469879_6077e3f796c35584031ee.png!small?1618469875502)

输出

![](https://image.3001.net/images/20210415/1618469916_6077e41c87375f5abbbc1.png!small?1618469912408)

继续访问数据库

mysql -D dev –user dev -p development01 超时了

mysql -D dev –user devtest -p devteam01 连接被拒绝

更多输出信息

![](https://image.3001.net/images/20210415/1618469985_6077e4615f475f1b92a77.png!small?1618469981426)

https://pentestlab.blog/2017/09/25/suid-executables/

https://unix.stackexchange.com/questions/116792/privileged-mode-in-bash

bash 信息

![](https://image.3001.net/images/20210415/1618470047_6077e49fe51d85610f673.png!small?1618470043799)

![](https://image.3001.net/images/20210415/1618470061_6077e4ade1693e947d746.png!small?1618470058609)

结束
--

OVER