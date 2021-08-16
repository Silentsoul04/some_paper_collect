\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/pj4KDZYxW94wtwBLd5lhJA)

对于任意文件读取漏洞，每个人的理解都不同，此处仅提出一种较为谨慎的思路。

  

  

  

  

1

**壹**

1

找到系统存在一个读取点，利用文件 ID 或者文件名读取文件的读取点，确定成功读取的状态

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08POPicyIfFX2vH5H7oTt3AWjSiaZMwK1mY15ngWWqsOXKpPB7NdjNRhoE8xHG662FeUAcj2NLv9Rwg/640?wx_fmt=png)

1

**贰**

1

读取一个不存在的文件，确定读取不存在文件时的状态

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08POPicyIfFX2vH5H7oTt3AWfnWuDOQl7XZVOnUcpzTnkTv8CZGSUib7E2edTYYpQbnmlsciaDk8aoOQ/640?wx_fmt=png)

1

**叁**

1

尝试使用 /./、a/../ 确定可以使用跨目录符, 确定是否可以使用跨目录符

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08POPicyIfFX2vH5H7oTt3AWLVRKZlf6NibA65ZjMWKEyyJQlhzWMEib52ibo08nXg44Fgc8ElpiabVE8g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08POPicyIfFX2vH5H7oTt3AWYvZ4jK4pu9Ls4qZlcgc99aWia5icX2I8Fgj08U6icibEmBiahPiaHDe3TbPQ/640?wx_fmt=png)

1

**肆**

1

确定可以使用跨目录符后尝试跨目录读取 / etc/passwd（linux）、boot.ini(windows) 等标志性文件, 标志性文件可随意拓展，注意选取标志性文件时选取读取权限要求最低的文件

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08POPicyIfFX2vH5H7oTt3AWjBB8VuTaqaRewm0KteicnbY8962PDTHrdqBKmF8vD7aGWXicLr8vId1A/640?wx_fmt=png)

读取 / etc/passwd 失败
------------------

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08POPicyIfFX2vH5H7oTt3AW3cyduOnZux1ickyJLRBMHU9VvheGIibOicicHRIJS9Y4iaDGTpCJvZq1zVg/640?wx_fmt=png)
-------------------------------------------------------------------------------------------------------------------------------------------------

**boot.ini 读取失败**

1

**伍**

1

无法读取系统文件，暂时无法分析出是权限问题还是其他原因，尝试读取 web 目录文件，建议选取 web.xml，读取方式为：(../){0,10}/WEB-INF/web.xml 依次递增../ 的数量，直到读取到 web.xml 文件，一般不会超过 10 个路径深度。

注：涉及到上传文件目录和 web 目录分离的情况无法读取到 web.xml 可考虑其他参照文件，多开脑洞就行

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08POPicyIfFX2vH5H7oTt3AWk5lU3sO8noAB2ibqjJfA32vCbrq52u9ibib7leR75Zz9eK0ibYtCAkiccHg/640?wx_fmt=png)

1

陆  

1

此时已经可以确定可以任意文件读取了，从 web.xml 中我们看到配置了 log4j 并且是 spring 框架，尝试读取../WEB-INF/classes/log4j.properties，查看 log4j 配置文件查看系统日志存放记录，看看能不能找到一些系统路径相关的数据

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08POPicyIfFX2vH5H7oTt3AWj3rQnaJLtv7NichZ4XWXPvINABsOOVzo9yaD0mmWEI3h5lBr9GrXuVQ/640?wx_fmt=png)  

1

**柒**

1

从上图中我们看到一个路径

#log4j.appender.vcc.File=C\\:\\\\log\\\\wxenterprise\\\\vcc.lo g，初步判断是

windows 服务器，并且 web 目录在 C 盘（这些都是推测），尝试读取 C 盘下

的一些文件来证明推测，成功读取

到../../../../../../../../../../../../../../../windows/win.ini（这里可能有同学要想，为啥不直接一开始就 fuzz 一大波 fuzz 个天地？这些都是个人习惯，除非无路可走，个人一般不采用大量数据的 fuzz，以免打草惊蛇）

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08POPicyIfFX2vH5H7oTt3AWwk13DxP3pGnGJj5v4lIicQlpscP1ssE8lx5RI98ywn9kGQXCCmZ7Pcg/640?wx_fmt=png)

1

**捌**

1

后续就大家自己疯狂脑补输出吧，最好能拿个权限不是？  

  

  

  

  

**后续：**
=======

在确定主机类型、可读取文件路径后，对主机进行端口扫描，查看除 web 端口外开放的端口，发现开了 21 端口

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08POPicyIfFX2vH5H7oTt3AWnWhbe42rZRK2Q1JM45f1HxVNQEnQ9Xr0jCicvCQOMYicPPKp3XIIwMGA/640?wx_fmt=png)

使用 nc 查看具体的 ftp 服务信息，发现是 FileZilla Server 0.9.60 beta

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08POPicyIfFX2vH5H7oTt3AW3pWLsAKsky7c4yjQX63cXzxtIKibO5s2jJcIbPhGibwFia2RqH0y3e5ibw/640?wx_fmt=png)

这个软件很有名，但是我不了解其目录构造，不过没关系，现下现看，去官网下一个同版本 (https://filezilla-project.org/download.php?type=server), 刚好是最新版

![](https://mmbiz.qpic.cn/mmbiz_jpg/RXib24CCXQ08POPicyIfFX2vH5H7oTt3AWG4eOaXfYAIAKFqaZ2reHs6xbZbroyRLbU8TqiaWN0yVJfDh8htx43vA/640?wx_fmt=jpeg)

搭建好环境后使用 everything 搜索 FileZilla 相关文件，发现只有两个目录不涉及到用户名

C:\\ProgramData\\Microsoft\\Windows\\StartMenu\\Programs\\FileZilla 

 Server\\Stop FileZi

lla Server.lnk

C:\\Program Files (x86)\\FileZilla Server\\FileZilla Server.xml

![](https://mmbiz.qpic.cn/mmbiz_jpg/RXib24CCXQ08POPicyIfFX2vH5H7oTt3AWX8RJbHL2mrgFoPXUPwA7GSUOU4QllCfHEzm686HiazVy4DYVTwUYYww/640?wx_fmt=jpeg)

分别尝试读取, 直接读配置文件目录，没读取成功，可能是路径不对

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08POPicyIfFX2vH5H7oTt3AWDcqKKVX43Eb3lQBIaOUbIuUYib5cF3wgS7Ug8x4tYXJRJfG8uxOLnvQ/640?wx_fmt=png)

但是咱们可以读快捷方式啊，默认创建的快捷方式，桌面上被删除不管，但是 Program Data 下的肯定是没毛病能读到的，读到之后可以查看快捷方式属性来找路径

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08POPicyIfFX2vH5H7oTt3AWmqRjffvEFYPkFibGMzpujFjGm7aDUnJq38xsWia9hk8qMibNtAGtgTYsw/640?wx_fmt=png)

读到后查看属性，果然，自定义目录了，咱们直接读配置文件，C:\\filezille\\FileZilla Ser ver\\ FileZilla Server.xml

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08POPicyIfFX2vH5H7oTt3AWFPtWoHFqd9ImqLDiaYaMk2kwKibmWBNNcsUtEJ1ZfYBcpZjWvmRXHkSQ/640?wx_fmt=png)

能读到，可以确定就是这个目录了

![](https://mmbiz.qpic.cn/mmbiz_jpg/RXib24CCXQ08POPicyIfFX2vH5H7oTt3AWZmFk9OBh2mVSicAu2icXPg8h7C3opkjMa0ggfJAtcBGJbLwic79qWdOHw/640?wx_fmt=jpeg)

配置文件里有账号密码，直接用 ftp 客户端连。ftp 目录从配置文件中可以看到在 C:\\wexin

![](https://mmbiz.qpic.cn/mmbiz_jpg/RXib24CCXQ08POPicyIfFX2vH5H7oTt3AW8ibnf3LamTxjj0RusoGEGUOCV4MnXeDgF7GU8TzWYcwiaALgXiav8YSiaQ/640?wx_fmt=jpeg)

发现 tomcat 在 ftp 目录下，查看 conf 目录下的 server.xml 文件，确认 webapps 目录（很多同学又问，为啥不直接去 webapps 下看了，我看了啊是空的，但是我要告诉你，作为一个稳健猥琐的人，我要先去看配置文件，确定 web 目录）

![](https://mmbiz.qpic.cn/mmbiz_jpg/RXib24CCXQ08POPicyIfFX2vH5H7oTt3AWfGQ4plCBW6kN7gXfNiaicZRB9JZffhPyC4Whl1NMncwK1t7dKvJhTDzA/640?wx_fmt=jpeg)

运气不错，docBase 刚好在 ftp 有权限上传下载的目录下

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08POPicyIfFX2vH5H7oTt3AWyk7iaW7EIe9zam0b656MtbHupwyWJA214aPSbUulpW5tqsyaYE3AVQQ/640?wx_fmt=png)

直接上传 webshell

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08POPicyIfFX2vH5H7oTt3AWhbOF23Xms5cfIBS0NvVu5R8eFW6C7kWQu6lKapD66cS0Vvk3oRIdlw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08POPicyIfFX2vH5H7oTt3AW5BosH7YAHk5fOcTUz5FvNImM6B548H0kficL971Gf3UprMk5tjbN6xA/640?wx_fmt=png)

Shell Get

end

  

![](https://mmbiz.qpic.cn/mmbiz_png/RXib24CCXQ08POPicyIfFX2vH5H7oTt3AWIicVuqXd7ibKDBaZWJhg9icdVyHGNgBr6vAv5VwtD687xVx4zGW8tibzzw/640?wx_fmt=png)