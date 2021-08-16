> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/lQVX7_Tsw7crERmzTiH0yg)

**0x01** Introduction  

* * *

虚拟机下载页面：https://www.vulnhub.com/entry/coffee-addicts-1,699/  

Description

```
Our coffee shop has been hacked!! can you fix the damage and find who did it?

This works better with VirtualBox rather than VMware
```

‍

**0x02** **Writeup**  

* * *

2.1 getshell

#####  2.1.1 端口信息

```
nmap -sS -sV -p 1-65535 -T4 -O 10.0.0.131
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSxuGQc3gghw9wMLdPCJ65rOaEJNI2jHrMmWcAic3qqiaeuRsf3RsLmtcJLNAcvwibfmwibtKvD7wKMQg/640?wx_fmt=png)

只开放了 22、80 端口，先从 web 应用入手

##### 2.1.2 脆弱服务

访问 80 端口应用版本信息：

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSxuGQc3gghw9wMLdPCJ65rJ9WIMicu0xrOgz5kic8dbTIib6ibEVn0j07NXCAoaKpYHJcz6bZDBw7kjA/640?wx_fmt=png)

“ADD coffeeaddicts.thm to your /etc/hosts”，修改本地 hosts

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSxuGQc3gghw9wMLdPCJ65r1CxkTcFwW1ia7MGDHFdkibq01ztSY3icxEhKa3SdGyRpqArRqX5qvaYkQ/640?wx_fmt=png)

再次访问，发现页面被黑

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSxuGQc3gghw9wMLdPCJ65rQKPvtjde9Mw7Wiaqp2ZS8PJKz4icRq8lMhGBb0lL6f4JdYF8t6m68r0w/640?wx_fmt=png)

使用 dirb 爬取目录

目录扫描：  

```
dirb http://coffeeaddicts.thm/
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSxuGQc3gghw9wMLdPCJ65rgtwatic4TEiaedSbia6cnZ1hQxKmBge95icrp7yyCRo3PRpoVjpzL8LmOg/640?wx_fmt=png)

#### 发现存在 wordpress 网站，使用 wpscan 扫描 cms 漏洞  

```
wpscan --url http://coffeeaddicts.thm/wordpress
wpscan --url http://coffeeaddicts.thm/wordpress -e u
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSxuGQc3gghw9wMLdPCJ65rdGh8CoJtAUD4aLtBeEJKEPI6tnPmy5SzKibfMbMmOsiaNib7m9VGGGeNA/640?wx_fmt=png)

发现用户 gus；在查看 wordpress 页面，发现与密码相关信息

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSxuGQc3gghw9wMLdPCJ65rxf7XxxMo5XLmLV1h1icbIzahnt4UwJcW4YHMBLdSHfh7PISX96ZB95g/640?wx_fmt=png)

使用 Crunch 生成密码字典，生成 gus 、i、need、you、back 这个几个词组的组合

```
crunch 15 15 -p gus i need you back
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSxuGQc3gghw9wMLdPCJ65ribegP3RibbjZhcric4C7ibbYIibaz8iclPkduM7jxrRkzib11nIAyZvvRXjmg/640?wx_fmt=png)

使用 burp-suite 爆破口令

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSxuGQc3gghw9wMLdPCJ65rLwSMicapnCuJNF0CpbvNIxeex1BpUXmClqzmSUszlBWxnia61EDO4ckQ/640?wx_fmt=png)

登陆测试爆破是否成功

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSxuGQc3gghw9wMLdPCJ65rCZDMom1gKnhlCib8X0n3jnQxibdYD8obx67oguBIIqpkQSs9ia6p340BQ/640?wx_fmt=png)

登陆成功，尝试 webshell 后台拿取 webshell。wp 常见后台 getshell 方法如下：

```
1、直接修改替换默认404页面
2、上传包含木马文件的zip主题部署
3、上产包含木马文件的zip插件部署
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSxuGQc3gghw9wMLdPCJ65r67KSdpw3LwYvdMNEBKtJyDfuemloM8ib2JR8YzBOxJv7EWgTJ2y4Oyg/640?wx_fmt=png)

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSxuGQc3gghw9wMLdPCJ65rKzJckde2bwFbficQxibtHHVb14Qwb1DOVTOlWEZrzVJrEbNGkaWrL9AQ/640?wx_fmt=png)

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSxuGQc3gghw9wMLdPCJ65rHj6kMUmlRfn8Rn901piboSzAVAgjWn7RibkN5Jibdvz8LsdfYjDhR4X4w/640?wx_fmt=png)

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSxuGQc3gghw9wMLdPCJ65rWR1UGPcnnAwm1iadkyfgpzYWHWZia69ofAXibfxvDCyPYRCVpXrKQ8chw/640?wx_fmt=png)

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSxuGQc3gghw9wMLdPCJ65rDADZwaPI0m33ghZBTbwvAAffINFcRKerAlKM2TJX56fUAIJ00QIdXg/640?wx_fmt=png)

在 gus 家目录发现 flag1。

##### 2.2 权限提权  

收集系统信息

```
uname -a
 cat /etc/redhat-release
 cat /etc/crontab
 crontab -l
 /var/spool/cron/
 ……
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSxuGQc3gghw9wMLdPCJ65r4YUAQLM1iavUybpk7SDuEKMImyl73bTmibU64cuJ0BfL7N3Qr4MMhiadA/640?wx_fmt=png)

这有一个思路暴力猜解可登陆的 3 个用户

```
hydra -L user.txt -P /usr/share/wordlists/rockyou.txt ssh://10.0.0.131
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSxuGQc3gghw9wMLdPCJ65rqsw22j83W1iak1ksDqdtOSlaQEkQyzDyx4HCF3g3tpnxIp7o4EqEMPw/640?wx_fmt=png)

爆破出 1 个账户口令  

第二个思路在 badbyte 家目录发现私钥

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSxuGQc3gghw9wMLdPCJ65r1qSkuax9Jh88tSRlO5YXJeTZNtej3QXvZIeMicXlXENyUj8YcuqAdpw/640?wx_fmt=png)

下载私钥转换为 john 可破解 hash

```
wget https://raw.githubusercontent.com/openwall/john/bleeding-jumbo/run/ssh2john.py
python ssh2john.py id_rsa > id_rsa.hash
john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSxuGQc3gghw9wMLdPCJ65r37GXoOZFuI56LmlyIGXUicXoEKeIZXVvb9TxBFXdVDo3hkibibp4Nvogg/640?wx_fmt=png)

使用 openssl 由私钥产生公钥  

```
openssl rsa -in id_rsa -out pub_id_rsa
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSxuGQc3gghw9wMLdPCJ65retHnxBcjVO3eqStzniaZcnP1JFRwphF3GSb4kMN1XEYl5hmbhsjONJw/640?wx_fmt=png)

登陆  

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSxuGQc3gghw9wMLdPCJ65rFicf9pLN588Dxc050iaCK8KZbdHeBTiaDRqpiaiaml1icBNAGEhB8ePBia0Bw/640?wx_fmt=png)

再次收集信息查看可利用点权限提升，发现 sudo 可提升权限

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSxuGQc3gghw9wMLdPCJ65rQVkgj1k0jxhr58AQFxYwGNVGrJlcicIatEiaQ6NClaIaaJIAIbicIB2qg/640?wx_fmt=png)

执行 sudo 提权  

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSxuGQc3gghw9wMLdPCJ65r37GXoOZFuI56LmlyIGXUicXoEKeIZXVvb9TxBFXdVDo3hkibibp4Nvogg/640?wx_fmt=png)

拿到 root 权限

![图片](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSxuGQc3gghw9wMLdPCJ65rB2JZWuicaLnXcQlfo4TPeLia12K3J79aQWXDx9AbrdMJRJDve8v2wAuw/640?wx_fmt=png)

获得 flag2！！！

**PS：**

君子平等待众人，上而不媚下不嗔。

盛气凌人人切恨，傲慢无礼伤人尊。

临财见色不失足，逢冤遇怨敛芒针。

言而有信行必果，德贯天地照古今。

更多精彩

*   [Web 渗透技术初级课程介绍](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247486030&idx=2&sn=185f303a2f1b5267c0865f117931959d&chksm=fcfc3718cb8bbe0e6f3ca97859e78342852537da2bef3cd76a83cb90ee64a8ca8953b35aa67e&scene=21#wechat_redirect)
    
*   [Web 漏扫软件 Awvs 之 14.3.210615184 原包下载](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247487193&idx=1&sn=04b00f06e4ccb16362f9a34b0e80dab7&chksm=fcfc338fcb8bba990958286bfbcdf5e98234bb6024baeb979db2f0df94d4b955dd3eab775ce9&scene=21#wechat_redirect)  
    
*   [系统漏扫软件 Nessus 之 pro 插件库](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247487142&idx=1&sn=592bc806cefdd8352e198ecf708b0061&chksm=fcfc33f0cb8bbae635657879e6c31dfb42b07ad21647040e83f70b899d1463e6bfab365cec50&scene=21#wechat_redirect)  
    
*   [商务合作](http://mp.weixin.qq.com/s?__biz=MzU2OTUxOTE2MQ==&mid=2247486808&idx=1&sn=f50f15f9a3ab7312a08b1f932292faca&chksm=fcfc300ecb8bb918213c6070d864ffcd70ad27ab6525521c31e9ccaa57bdfa2968360ed7e8fe&scene=21#wechat_redirect)
    

![](https://mmbiz.qpic.cn/mmbiz_png/zSNEpUdpZQSd9wDlUiar0tUpHCYAzrZfTzOvS2SEw9cia9j7d1HKP2bWArPLCegs1XoejVUPu0GkSuZh7Wia7aExA/640?wx_fmt=png)

**如果感觉文章不错，分享让更多的人知道吧！**