> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI2NDQyNzg1OA==&mid=2247485520&idx=1&sn=afe7ab1dd663bf8dcc9811c840f33614&chksm=eaad886dddda017b5ca9f42ac926ec8abdb42e958561924144f79aad5969c7310bf059e9ba03&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_svg/iaRlzG8zy7BvRMzJGOiceBHhbWU2ah6heQIpA6vRFSf5MqB7kpysUPMEKgXTZPLFWQTEUFj85jDic0ZaHicKGMKIPFNglRYkfrjA/640?wx_fmt=svg)

**目录**

![](https://mmbiz.qpic.cn/mmbiz_svg/iaRlzG8zy7BvRMzJGOiceBHhbWU2ah6heQPRxy4CqRJhPx9DlAnUn1Bp3z4m11DzOV3LbvPkiad3fbPDkvXzQYDnD7MicRVjZWic1/640?wx_fmt=svg)

1.kiwi 模块

2.kiwi 模块的使用

   2.1 creds_all

   2.1 kiwi_cmd

  

**1.kiwi 模块**

使用 kiwi 模块需要 system 权限，所以我们在使用该模块之前需要将当前 MSF 中的 shell 提升为 system。提到 system 有两个方法，一是当前的权限是 administrator 用户，二是利用其它手段先提权到 administrator 用户。然后 administrator 用户可以直接 getsystem 到 system 权限。

提权到 system 权限

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dwtuWC0WqZzuqI4W5byp3v1pW46gVhzv3Ol4vmOVicxtuhkomvzlhLiaYBaKxiaNZRpgVpF6fjeblCQ/640?wx_fmt=png)

进程迁移

kiwi 模块同时支持 32 位和 64 位的系统，但是该模块默认是加载 32 位的系统，所以如果目标主机是 64 位系统的话，直接默认加载该模块会导致很多功能无法使用。所以如果目标系统是 64 位的，则必须先查看系统进程列表，然后将 meterpreter 进程迁移到一个 64 位程序的进程中，才能加载 kiwi 并且查看系统明文。如果目标系统是 32 位的，则没有这个限制。  

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dwtuWC0WqZzuqI4W5byp3vaRCmDS0fDpBZ2JgDGl0hSQWuvxwmwicCPrvfHmZueleOQicvSZvxGCZA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dwtuWC0WqZzuqI4W5byp3vFYAa8BLJjOD7ktgdKsmFpZVmHWiaze4Z03HXUVQyDGY5zIFDZPRYwyQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dwtuWC0WqZzuqI4W5byp3vqU75r4tsWV0MODZsIjcIJBVfz6PKibP6SUzyhNicWZyzaicYgxicrLy5EA/640?wx_fmt=png)

**2.kiwi 模块的使用**

加载 kiwi 模块  

```
load kiwi
```

查看 kiwi 模块的使用  

```
help kiwi
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dwtuWC0WqZzuqI4W5byp3vibCSBicxsqibbIxic3S3AXjMC9vuogj9ibS86oMDZoiawoz2ra4Ovia5hQZ0g/640?wx_fmt=png)

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

creds_all

该命令可以列举系统中的明文密码

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dwtuWC0WqZzuqI4W5byp3vSyLw71AhyaQRDrwf2ACknTBibuSbRwYxjxrUDAvYGlJ6gdyUNxOlxhQ/640?wx_fmt=png)

kiwi_cmd

kiwi_cmd 模块可以让我们使用 mimikatz 的全部功能，该命令后面接 mimikatz.exe 的命令

```
kiwi_cmd sekurlsa::logonpasswords
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2dwtuWC0WqZzuqI4W5byp3vQ1Yr4ymuCykCGR9ibibDHDtYrWLNwfHVlO1jURTZ5HKxPBibhaZ58IHmw/640?wx_fmt=png)

其他模块的用法后续会在原文慢慢更新。

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2ckkbwTsBvnDJpb89o8WMxvAKOaVnz60hOe7y3wAHiclddyK53lpEKIQlx4DKOq6EojHibVicgibDB2aQ/640?wx_fmt=gif)

来源：谢公子的博客

责编：Shawn

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2edCjiaG0xjojnN3pdR8wTrKhibQ3xVUhjlJEVqibQStgROJqic7fBuw2cJ2CQ3Muw9DTQqkgthIjZf7Q/640?wx_fmt=png)

如果文中有错误的地方，欢迎指出。有想转载的，可以留言我加白名单。

最后，欢迎加入谢公子的小黑屋（安全交流群）(QQ 群：783820465)

![](https://mmbiz.qpic.cn/mmbiz_gif/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SjCxEtGic0gSRL5ibeQyZWEGNKLmnd6Um2Vua5GK4DaxsSq08ZuH4Avew/640?wx_fmt=gif)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2et9NHxRhN8exP4Ly6FKH9SFQtevncFtKIlfLdaxSwwqFxgkrUz1x12kPp3ueaJctagDUcyJDGJyA/640?wx_fmt=png)