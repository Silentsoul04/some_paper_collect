> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/wN5YTsstuags_YQAe1eqUQ)

![](https://mmbiz.qpic.cn/mmbiz_png/kTIZMBcJhwial0TE41OIwlFPMaQg7G6BiaVTWK5cFmBm6kl597AoJibRQDvBtCYTQLYbjGr7LmZiabex8m2WicPoZbw/640?wx_fmt=png)

工具介绍

**致远 OA 漏洞检查与利用工具，收录漏洞如下：**

```
信息泄露：
致远OA A8 状态监控页面信息泄露
致远OA A6 initDataAssess.jsp 用户敏感信息泄露
致远OA A6 createMysql.jsp 数据库敏感信息泄露
致远OA A6 DownExcelBeanServlet 用户敏感信息泄露
致远OA getSessionList.jsp Session泄漏漏洞
SQL注入：
致远OA A6 setextno.jsp SQL注入漏洞
致远OA A6 test.jsp SQL注入漏洞

文件上传：
致远OA ajax.do 登录绕过&任意文件上传
致远OA Session泄露&任意文件上传漏洞

任意文件下载：
致远OA webmail.do任意文件下载
```

**支持批量扫描，使用方法如下：**

```
Usage:
python3 seeyon_exp.py -u url              #漏洞检测
python3 seeyon_exp.py -u url --att        #漏洞检测+getshell
python3 seeyon_exp.py -f url.txt          #批量漏洞检查
python3 seeyon_exp.py -f url.txt --att    #批量漏洞检测+getshell

Options:
  -h, --help            show this help message and exit
  -u URL, --url=URL     target url
  -f FILE, --file=FILE  url file
  --att                 getshell
```

```
python3 seeyon_exp.py -u url
```

![](https://mmbiz.qpic.cn/mmbiz_png/kTIZMBcJhwial0TE41OIwlFPMaQg7G6BiafsMIMSwiaIRXdiao11wEEjodunxqmbU0PrbwMpB52Oibl8BibLNrBp4RLg/640?wx_fmt=png)

```
 python3 seeyon_exp.py -u url  --att
```

![](https://mmbiz.qpic.cn/mmbiz_png/kTIZMBcJhwial0TE41OIwlFPMaQg7G6BiaAoJrNKeicjvqLkB5Pv6rADOGjw1icg5M8s2emgybfdibqkreibn10uXQHw/640?wx_fmt=png)

**默认使用冰蝎 3 的 webshell，密码为 rebeyond  
扫码结果保存为 result.txt**

**使用批量扫描时，建议先筛选出存活 url**

**仅用于授权测试，违者后果自负**

参考链接：  

```
https://github.com/PeiQi0/PeiQi-WIKI-POC/tree/PeiQi/PeiQi_Wiki/OA%E4%BA%A7%E5%93%81%E6%BC%8F%E6%B4%9E/%E8%87%B4%E8%BF%9COA
```

GitHub 地址：

```
https://github.com/Summer177/seeyon_exp.git
```