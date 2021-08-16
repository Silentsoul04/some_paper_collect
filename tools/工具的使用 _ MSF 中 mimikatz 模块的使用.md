> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI2NDQyNzg1OA==&mid=2247485867&idx=1&sn=0214e97984d0f85ae77f51e5b077c63e&chksm=eaad8996ddda0080eb00131f89a16fd429f10046fdb7409f6119497af317c13dc7e1d3fe33e7&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_png/b96CibCt70iaZEGn5b7odL7EkjBrdMksVEibGibTd6asp8mHqyWTXzF8lyicCjDicIiaXg5ou90A90vdQK32hfyQaCbpg/640?wx_fmt=png)

**目录**  

mimikatz 模块的加载

mimikatz 模块的使用

mimikatz_command 模块的用法

![](https://mmbiz.qpic.cn/mmbiz_png/Oj9XzC4Q6Br3CXFEvicPTLic76Io2UIksbJ2NFhdntXoQJiabHLIwgC1h2icSMKiazBNwABqtx69svvPaHTWXMbaSdA/640?wx_fmt=png)

mimikatz 模块的加载

![](https://mmbiz.qpic.cn/mmbiz_png/cVtibqxAt4HwxvgJBVrtyGIJFiaUbtJLo9w1bpAbicJc6ibrT2qSicJOwjYbFhUgU8dx5VNgvWSDs8cUIKDh7oIicOcA/640?wx_fmt=png)

MSF 中的 mimikatz 模块，可以列举出系统中的各种凭据，以及执行一些 mimikatz 相关的命令。目前，该模块已经更新为功能更全的 kiwi 模块，传送门：[工具的使用 | MSF 中 kiwi 模块的使用](http://mp.weixin.qq.com/s?__biz=MzI2NDQyNzg1OA==&mid=2247485520&idx=1&sn=afe7ab1dd663bf8dcc9811c840f33614&chksm=eaad886dddda017b5ca9f42ac926ec8abdb42e958561924144f79aad5969c7310bf059e9ba03&scene=21#wechat_redirect)  

使用 mimikatz 模块需要 System 权限，所以我们在使用该模块之前需要将当前 MSF 中的 shell 提升为 system。提到 system 有两个方法，一是当前的权限是 administrator 用户，二是利用其它手段先提权到 administrator 用户。然后 administrator 用户可以直接 getsystem 到 system 权限。

**提权到 system 权限**

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2el8IVJb5iaQMuhOH5eb7lm8wTJkns41rPibHloHCOOCcDrKiak9kRcQaNPJvoY8QnB8ne16zKFw6x5A/640?wx_fmt=png)

**进程迁移**

kiwi 模块同时支持 32 位和 64 位的系统，但是该模块默认是加载 32 位的系统，所以如果目标主机是 64 位系统的话，直接默认加载该模块会导致很多功能无法使用。所以如果目标系统是 64 位的，则必须先查看系统进程列表，然后将 meterpreter 进程迁移到一个 64 位程序的进程中，才能加载 kiwi 并且查看系统明文。如果目标系统是 32 位的，则没有这个限制。

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2el8IVJb5iaQMuhOH5eb7lm8hXRNXibhibRszVah7XQQEhYQpSmuENbjjxITjuJXD3QhyMBJ9SVyn2Cw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2el8IVJb5iaQMuhOH5eb7lm8I4xOibHLHYJ2WaB9sWQrSiax9UxUrCEeHzxia5AUvv4W4r1zrWobCXUIg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2el8IVJb5iaQMuhOH5eb7lm8VtI01uibsvnKW3NDnYxFWkjCT8Tgib5BXzNdMQDeyia0C0TmTkrULYLYQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Oj9XzC4Q6Br3CXFEvicPTLic76Io2UIksbJ2NFhdntXoQJiabHLIwgC1h2icSMKiazBNwABqtx69svvPaHTWXMbaSdA/640?wx_fmt=png)

mimikatz 模块的使用

![](https://mmbiz.qpic.cn/mmbiz_png/cVtibqxAt4HwxvgJBVrtyGIJFiaUbtJLo9w1bpAbicJc6ibrT2qSicJOwjYbFhUgU8dx5VNgvWSDs8cUIKDh7oIicOcA/640?wx_fmt=png)

```
加载kiwi模块
load mimikatz
查看kiwi模块的使用
help mimikatz
```

```
mimikatz_command -f mimikatz的命令
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2el8IVJb5iaQMuhOH5eb7lm8ia8rs16w9jdNU1RIdn7K9DBaI698yeKTNMGjv5l8fP6gVeNCzHCpdhw/640?wx_fmt=png)

可以看到 mimikatz 下有七个命令：

*   kerberos：kerberos 相关的模块
    
*   livessp：尝试检索 livessp 凭据
    
*   mimikatz_command：运行一个定制的命令
    
*   msv：msv 凭证相关的模块，列出目标主机的用户密码哈希
    
*   ssp：ssp 凭证相关的模块
    
*   tspkg：tspkg 凭证相关的模块
    
*   wdigest：wdigest 凭证相关的模块
    

### mimikatz_command 模块的用法

mimikatz_command 模块可以让我们使用 mimikatz 的全部功能。

```
mimikatz_command -f sekurlsa::searchPasswords
```

例如，使用以下命令查看系统中的明文密码

```
mimikatz_command -f sekurlsa::searchPasswords
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2el8IVJb5iaQMuhOH5eb7lm84oTsCs9sdK5BhxhpgVKLOUhPzCsZW1iayklfDliae2DZwSeaLYpN0uuw/640?wx_fmt=png)

未完待续，请关注原文！