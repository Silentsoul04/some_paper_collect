> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [xz.aliyun.com](https://xz.aliyun.com/t/8933)

1. 前言
-----

在 cnvd 上看到最新的披露中有 waychar 报名系统的 sql 注入，于是想复现看看。

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201230111545-4ea75e36-4a4d-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201230111545-4ea75e36-4a4d-1.png)

> 漏洞复现环境  
> 源码下载地址：[http://down.chinaz.com/soft/39094.htm](http://down.chinaz.com/soft/39094.htm)  
> 使用 phpstudy 搭载环境

2. 前台登陆处存在 SQL 注入
-----------------

### 复现

登陆处，用数据库已有的账号密码登录  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20201230111616-60f78048-4a4d-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201230111616-60f78048-4a4d-1.png)  
抓包，保存为 txt 文件  
[![](https://xzfile.aliyuncs.com/media/upload/picture/20201230111635-6c79e780-4a4d-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201230111635-6c79e780-4a4d-1.png)

sqlmap：

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201230111658-7a1c7880-4a4d-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201230111658-7a1c7880-4a4d-1.png)

### 分析

`controller/ajax.php` 24-58 行

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201230112538-aff26b58-4a4e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201230112538-aff26b58-4a4e-1.png)

前端传输的`username`参数没有任何的过滤操作就拼接到了 sql 语句中，导致了 sql 注入

3. 前台找回密码处存在 SQL 注入
-------------------

### 复现

输入数据库中存在的号码

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201230111730-8d2e1d8e-4a4d-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201230111730-8d2e1d8e-4a4d-1.png)

截获数据包，保存为 txt 文件

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201230111743-94d3c480-4a4d-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201230111743-94d3c480-4a4d-1.png)

使用 sqlmap

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201230111800-9f0e9696-4a4d-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201230111800-9f0e9696-4a4d-1.png)

### 分析

`controller/ajax.php` 12-23 行

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201230111813-a6dc8590-4a4d-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201230111813-a6dc8590-4a4d-1.png)

`mobi`参数没有做任何过滤就传递给`$mobi`变量，然后拼接在 sql 语句中，导致了 sql 注入

4. 前台重置密码处延时注入
--------------

### 复现

payload：`12345678'and(select*from(select+sleep(5))a/**/union/**/select+1)='`

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201230111830-b10502ae-4a4d-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201230111830-b10502ae-4a4d-1.png)

### 分析

`controller/ajax.php` 100-113 行

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201230111859-c1f2086e-4a4d-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201230111859-c1f2086e-4a4d-1.png)

`password_new`参数没有做任何过滤操作，传递给了`$password_o`变量，并且最后拼接在了 SQL 语句中  
sql 语句：`update w_user set password_o = '$password_o', password = '$password' where id = " . $_COOKIE['id'];`

5. 活动信息处 sql 注入
---------------

### 复现

payload：`/index.php?c=race&action=race_msg&id=57 order by 1,2,3,4,5,6,7`

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201230111932-d6087c20-4a4d-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201230111932-d6087c20-4a4d-1.png)

可以看到是正常显示页面的  
payload：`/index.php?c=race&action=race_msg&id=57 order by 1,2,3,4,5,6,7,8`

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201230112025-f526a596-4a4d-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201230112025-f526a596-4a4d-1.png)

红框部分的内容消失不见  
说明有 7 个字段  
payload：`/index.php?c=race&action=race_msg&id=57 union select 1,2,3,4,5,user(),7`

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201230112051-051f128a-4a4e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201230112051-051f128a-4a4e-1.png)

获得了当前的用户

### 分析

`controller/ajax.php` 136-142 行

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201230112108-0efc01f0-4a4e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201230112108-0efc01f0-4a4e-1.png)

`id`参数没有过滤，导致了 sql 注入

6. 后台会员管理搜索处存在 SQL 注入
---------------------

密码默认为 admin123  
会员管理

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201230112127-1a9356ee-4a4e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201230112127-1a9356ee-4a4e-1.png)

随便输入一个号码，并抓包

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201230112139-2144b74e-4a4e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201230112139-2144b74e-4a4e-1.png)

将数据包保存为一个 txt

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201230112149-27b0e88c-4a4e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201230112149-27b0e88c-4a4e-1.png)

结果

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201230112202-2f3e9f9a-4a4e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201230112202-2f3e9f9a-4a4e-1.png)

分析：  
代码位置：controller/user.php 12-24 行

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201230112217-38099b48-4a4e-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201230112217-38099b48-4a4e-1.png)

mobi 参数没有经过过滤，就拼接到 sql 语句中，导致了 SQL 注入

7. 总结
-----

经过复现了 6 处 sql 注入，发现源码中基本上对参数都没有做任何过滤操作，导致了整个站中都有 SQL 注入，感兴趣的可以在官网下载源码来看看。