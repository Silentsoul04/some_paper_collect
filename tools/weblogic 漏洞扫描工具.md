> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/oUN5lWgf_FpxDnE5BOWLHg)

师傅们周末快乐，周末也要学习呀![](https://mmbiz.qpic.cn/mmbiz_png/m41BLSafyI6TtQ2VyDeW9C9UAyKJ890Je4TntibulAZlLGsTEicz8FiaZZoHJFOiajAeib5lhnGWsq5ePChp1jIh5AQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/m41BLSafyI6TtQ2VyDeW9C9UAyKJ890Je4TntibulAZlLGsTEicz8FiaZZoHJFOiajAeib5lhnGWsq5ePChp1jIh5AQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/m41BLSafyI6TtQ2VyDeW9C9UAyKJ890Je4TntibulAZlLGsTEicz8FiaZZoHJFOiajAeib5lhnGWsq5ePChp1jIh5AQ/640?wx_fmt=png)

目前可检测漏洞编号有（部分非原理检测，需手动验证）：

*   weblogic administrator console
    
*   CVE-2014-4210
    
*   CVE-2016-0638
    
*   CVE-2016-3510
    
*   CVE-2017-3248
    
*   CVE-2017-3506
    
*   CVE-2017-10271
    
*   CVE-2018-2628
    
*   CVE-2018-2893
    
*   CVE-2018-2894
    
*   CVE-2018-3191
    
*   CVE-2018-3245
    
*   CVE-2018-3252
    
*   CVE-2019-2618
    
*   CVE-2019-2725
    
*   CVE-2019-2729
    
*   CVE-2019-2890
    
*   CVE-2020-2551
    
*   CVE-2020-14882
    
*   CVE-2020-14883
    

大佬可自己加最新 poc。  
使用环境：

*   python >= 3.6
    

进入项目目录，使用以下命令安装依赖库

```
$ pip3 install requests
```

![](https://mmbiz.qpic.cn/mmbiz_png/m41BLSafyI5frs1A1GqkHL5VSKEr0zdR3X6DqWRv6TZgCjL1NGkTUPNwd9FIt4qfK8HCSZHxBibNdS7nUS4r8Tg/640?wx_fmt=png)

使用说明：  

```
usage: ws.py [-h] -t TARGETS [TARGETS ...] -v VULNERABILITY
             [VULNERABILITY ...] [-o OUTPUT]

optional arguments:
  -h, --help            帮助信息
  -t TARGETS [TARGETS ...], --targets TARGETS [TARGETS ...]
                        直接填入目标或文件列表（默认使用端口7001）. 例子：
                        127.0.0.1:7001
  -v VULNERABILITY [VULNERABILITY ...], --vulnerability VULNERABILITY [VULNERABILITY ...]
                        漏洞名称或CVE编号，例子："weblogic administrator console"
  -o OUTPUT, --output OUTPUT
                        输出 json 结果的路径。默认不输出结果
  -s, --ssl             强制使用 https 协议请求
```

示例：  

```
(venv) ~/weblogicScanner$ python ws.py -t 192.168.124.129
[23:03:04][INFO] [*][Weblogic Console][192.168.56.129:7001] Start...
[23:03:04][INFO] [+][Weblogic Console][192.168.56.129:7001] Found module!
[23:03:04][INFO] [*][Weblogic Console][192.168.56.129:7001] Please verify manually!
[23:03:04][INFO] [*][CVE-2014-4210][192.168.56.129:7001] Start...
[23:03:04][INFO] [-][CVE-2014-4210][192.168.56.129:7001] Not found.
[23:03:04][INFO] [*][CVE-2016-0638][192.168.56.129:7001] Start...
[23:03:06][INFO] [-][CVE-2016-0638][192.168.56.129:7001] Not vulnerability.
[23:03:06][INFO] [*][CVE-2016-3510][192.168.56.129:7001] Start...
[23:03:08][INFO] [-][CVE-2016-3510][192.168.56.129:7001] Not vulnerability.
[23:03:08][INFO] [*][CVE-2017-3248][192.168.56.129:7001] Start...
[23:03:10][INFO] [-][CVE-2017-3248][192.168.56.129:7001] Not vulnerability.
[23:03:10][INFO] [*][CVE-2017-3506][192.168.56.129:7001] Start...
[23:03:10][INFO] [-][CVE-2017-3506][192.168.56.129:7001] Not vulnerability.
[23:03:10][INFO] [*][CVE-2017-10271][192.168.56.129:7001] Start...
[23:03:10][INFO] [-][CVE-2017-10271][192.168.56.129:7001] Not vulnerability.
[23:03:10][INFO] [*][CVE-2018-2628][192.168.56.129:7001] Start...
[23:03:14][INFO] [+][CVE-2018-2628][192.168.56.129:7001] Exists vulnerability!
[23:03:14][INFO] [*][CVE-2018-2893][192.168.56.129:7001] Start...
[23:03:18][INFO] [+][CVE-2018-2893][192.168.56.129:7001] Exists vulnerability!
[23:03:18][INFO] [*][CVE-2018-2894][192.168.56.129:7001] Start...
[23:03:19][INFO] [+][CVE-2018-2894][192.168.56.129:7001] Found module!
[23:03:19][INFO] [*][CVE-2018-2894][192.168.56.129:7001] Please verify manually!
[23:03:19][INFO] [*][CVE-2018-3191][192.168.56.129:7001] Start...
[23:03:23][INFO] [+][CVE-2018-3191][192.168.56.129:7001] Exists vulnerability!
[23:03:23][INFO] [*][CVE-2018-3245][192.168.56.129:7001] Start...
[23:03:29][INFO] [-][CVE-2018-3245][192.168.56.129:7001] Not vulnerability.
[23:03:29][INFO] [*][CVE-2018-3252][192.168.56.129:7001] Start...
[23:03:36][INFO] [+][CVE-2018-3252][192.168.56.129:7001] Found module!
[23:03:36][INFO] [*][CVE-2018-3252][192.168.56.129:7001] Please verify manually!
[23:03:36][INFO] [*][CVE-2019-2618][192.168.56.129:7001] Start...
[23:03:36][INFO] [+][CVE-2019-2618][192.168.56.129:7001] Found module!
[23:03:36][INFO] [*][CVE-2019-2618][192.168.56.129:7001] Please verify manually!
[23:03:36][INFO] [*][CVE-2019-2725][192.168.56.129:7001] Start...
[23:03:46][INFO] [-][CVE-2019-2725][192.168.56.129:7001] Not vulnerability.
[23:03:46][INFO] [*][CVE-2019-2729][192.168.56.129:7001] Start...
[23:03:54][INFO] [-][CVE-2019-2729][192.168.56.129:7001] Not vulnerability.
[23:03:54][INFO] [*][CVE-2019-2888][192.168.56.129:7001] Start...
[23:03:56][INFO] [+][CVE-2019-2888][192.168.56.129:7001] Found module!
[23:03:56][INFO] [*][CVE-2019-2888][192.168.56.129:7001] Please verify manually!
[23:03:56][INFO] [*][CVE-2019-2890][192.168.56.129:7001] Start...
[23:03:58][INFO] [-][CVE-2019-2890][192.168.56.129:7001] Not vulnerability.
[23:03:58][INFO] [*][CVE-2020-2551][192.168.56.129:7001] Start...
[23:03:58][INFO] [+][CVE-2020-2551][192.168.56.129:7001] Found module!
[23:03:58][INFO] [*][CVE-2020-2551][192.168.56.129:7001] Please verify manually!
[23:03:58][INFO] [*][CVE-2020-2555][192.168.56.129:7001] Start...
[23:04:02][INFO] [+][CVE-2020-2555][192.168.56.129:7001] Exists vulnerability!
[23:04:02][INFO] [*][CVE-2020-2883][192.168.56.129:7001] Start...
[23:04:06][INFO] [+][CVE-2020-2883][192.168.56.129:7001] Exists vulnerability!
[23:04:06][INFO] [*][CVE-2020-14882][192.168.56.129:7001] Start...
[23:04:23][INFO] [-][CVE-2020-14882][192.168.56.129:7001] Not vulnerability.
[23:04:23][INFO] [*][CVE-2020-14883][192.168.56.129:7001] Start...
[23:04:23][INFO] [+][CVE-2020-14883][192.168.56.129:7001] Exists vulnerability!
```

私聊公众号发送 “**weblogic**” 获取下载链接。

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

“如侵权请私聊公众号删文”

**欢迎关注 系统安全运维** 

公众号

觉得不错点个 “赞”、“在看” 哦