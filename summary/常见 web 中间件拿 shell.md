\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/\_axO\_TIeNCplR1wnVcmIAw)

![](https://mmbiz.qpic.cn/mmbiz_gif/3xxicXNlTXLicwgPqvK8QgwnCr09iaSllrsXJLMkThiaHibEntZKkJiaicEd4ibWQxyn3gtAWbyGqtHVb0qqsHFC9jW3oQ/640?wx_fmt=gif)

1.weblogic

![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLibUl0qjn09HNXx81sXV7TXsJNPC0dpsZsibAnq1Zic3zfCI4M0y3Mf50fbE3UpaP7icg8AjBb8d5QHAw/640?wx_fmt=jpeg)

后台页面：（http 为 7001，https 为 7002）

Google 关键字：WebLogic Server AdministrationConsole inurl:console

默认的用户名密码

1、用户名密码均为：weblogic  
2、用户名密码均为：system  
3、用户名密码均为：portaladmin  
4、用户名密码均为：guest

上传地方：

workshop> Deployments> Web 应用程序 > 部署新的 Web 应用程序模块…

上传 war 的 webshell。

  
 ![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLibUl0qjn09HNXx81sXV7TXsF5IgHELWILjIgicp7jFd6aMvzLBL95z7mC8JG4Jex4iceZ8BQxhzpyjA/640?wx_fmt=jpeg)   

上传后目标模块 -> 部署。

2.Tomcat

后台：http://172.16.102.35:8080/manager/html  
默认用户名密码  
tomcat tomcat

上传地方：

  
 ![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLibUl0qjn09HNXx81sXV7TXsBw4LV3Ij07cibEpkImV4nAaqSz5icy0g6X6g2zhtcZthA7FsJMmTtuQA/640?wx_fmt=jpeg)   

Deploy 之后即发布成功

shell 地址：http://172.16.102.35:8080/magerx/test.jsp(其中 magerx 为 war 包的名字）

  
![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLibUl0qjn09HNXx81sXV7TXs1M4z5LK001SYLLHIoKMSrqHVPENt19yUrXQMDrSKgfDBODCdgPX0ZQ/640?wx_fmt=jpeg)

3.jboss

后台：http://172.16.102.35:9990

上传地方：Deployments>Manage Deployments>Add Content

  
 ![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLibUl0qjn09HNXx81sXV7TXs30ibX7LFibeeicuQzGFvBexCkJkjiblsDrMKupQ6TXfItztw2RhibibHjmVA/640?wx_fmt=jpeg)   

Enable 后即可发布

shell 地址：

http://172.16.102.35:8080/magerx/test.jsp

![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLibUl0qjn09HNXx81sXV7TXsAJDHCUToIOnde2YbkGtVsR0XbwYFltaQHsgvYicCyl4VPge9ZTEHTIg/640?wx_fmt=jpeg)

4.JOnAS

后台：http://172.16.102.35:9000/jonasAdmin/

默认用户名密码：  
jadmin jonas  
tomcat tomcat  
jonas jonas

上传地方：Deployment>Web Modules (WAR)>Upload

![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLibUl0qjn09HNXx81sXV7TXs7xkZibcoSqN9G10T5aJib7ibLTQ8zN2BHg9whWZRsTKqD1icjkDOygHSQw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLibUl0qjn09HNXx81sXV7TXsl4nvsTeLHYicDF6hpibSTqW5hyfCzWFOtIuf9jSLcAwiaKu7ibYjgNFATQ/640?wx_fmt=jpeg)

Apply 之后即可发布

shell 地址: http://172.16.102.35:9000/magerx/test.jsp

5.WebSphere

后台地址: https://172.16.102.35:9043/ibm/console/logon.jsp

上传地方: 应用程序 > 新建应用程序 > 新建企业应用程序

  
 ![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLibUl0qjn09HNXx81sXV7TXsaXE0aFVFh1WFRIUB1tn62mic3KsH39EhUJbW6gic6pQYa55kAzTu1zicg/640?wx_fmt=jpeg)   
 ![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLibUl0qjn09HNXx81sXV7TXs91icfKOHicGNAU4f5SjK182NQqPUcpFxsMfibfLq7zxsnmt0iaibFCSPuvw/640?wx_fmt=jpeg) 

接下来各种下一步，步骤 4 注意填好 “上下文根”

![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLibUl0qjn09HNXx81sXV7TXsiaDBTRL076RmUTuyGVviclpaAHIIrXGu8a3Dj4iaSdbFqE3JSjxzJ3hoQ/640?wx_fmt=jpeg)

完成后单击保存

 ![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLibUl0qjn09HNXx81sXV7TXsst4Jk2NxMpjevgBjcyPDY723iavUsLKiatXciaaAjgc5D3QDwLd7UcBeQ/640?wx_fmt=jpeg)   

回到应用程序 > 应用程序类型 > WebSphere 企业应用程序

  
 ![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLibUl0qjn09HNXx81sXV7TXspzdBib6LiaDuPYjOjlEFyrdYicZGbguEleIr1utNNwgoMU1C7dv6qUMrw/640?wx_fmt=jpeg)   

选中你上传的 war 包 这里是 paxmac  点击启动 即可发布  
shell 地址: http://172.16.102.35:9080/paxmac/test.jsp

<!–  
jsp 文件打包，可以使用 jdk/JRE 自带的 jar 命令：  
切换到要打包的文件目录  
jar -cvf magerx.war  test.jsp –>

![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLicjiasf4mjVyxw4RbQt9odm9nxs9434icI9TG8AXHjS3Btc6nTWgSPGkvvXMb7jzFUTbWP7TKu6EJ6g/640?wx_fmt=jpeg)

推荐文章 ++++

![](https://mmbiz.qpic.cn/mmbiz_jpg/US10Gcd0tQFGib3mCxJr4oMx1yp1ExzTETemWvK6Zkd7tVl23CVBppz63sRECqYNkQsonScb65VaG9yU2YJibxNA/640?wx_fmt=jpeg)

\*[Webshell 高级样本收集](http://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650482270&idx=4&sn=e0db73d1f68d2d14fd76aafb6bc1e74d&chksm=83ba43bab4cdcaac1045755d9674d8dfeaeb7993a056ecff8e04f1696a46371542f4ae8780ee&scene=21#wechat_redirect)  

\* [在服务器上利用 WebDAV 并获取 Shell](http://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650480818&idx=4&sn=03e42d67681bb1f970007040e581074a&chksm=83ba4556b4cdcc408aca654c77b8b2386360876bb655ca0d814ad4620f478f0f7d3d895d029e&scene=21#wechat_redirect)

\* [反弹 shell 的各种姿势](http://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650475868&idx=4&sn=8335155778a02870f518819e898253a6&chksm=83ba68b8b4cde1ae894d3feb9b6644d04cfedacca79201b25bcc47cb222f901187ae7bb408bb&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib0FWIDRa9Kwh52ibXkf9AAkntMYBpLvaibEiaVibzNO1jiaVV7eSibPuMU3mZfCK8fWz6LicAAzHOM8bZUw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_gif/NZycfjXibQzlug4f7dWSUNbmSAia9VeEY0umcbm5fPmqdHj2d12xlsic4wefHeHYJsxjlaMSJKHAJxHnr1S24t5DQ/640?wx_fmt=gif)