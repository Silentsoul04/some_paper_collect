> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/iUKTznxtSNrk5PfDxKnFfQ)

作者前面详细介绍了 Powershell 基础知识，包括常见的用法，涉及基础概念、管道和重定向等。**这篇文章将继续讲解 Powershell 基础知识，包括** **Powershell 条件语句、循环语句、数组、函数 、字符串操作、注册表访问等。Powershell 被广泛应用于安全领域，甚至成为每一位 Web 安全必须掌握的技术。**

本文参考了 Bilibili 的老师的课程，同时也结合了作者之前的编程经验进行讲解。作者作为网络安全的小白，分享一些自学基础教程给大家，希望你们喜欢。同时，更希望你能与我一起操作深入进步，后续也将深入学习网络安全和系统安全知识并分享相关实验。总之，希望该系列文章对博友有所帮助，写文不容易，大神请飘过，不喜勿喷，谢谢！

> 从 2019 年 7 月开始，我来到了一个陌生的专业——网络空间安全。初入安全领域，是非常痛苦和难受的，要学的东西太多、涉及面太广，但好在自己通过分享 100 篇 “网络安全自学” 系列文章，艰难前行着。感恩这一年相识、相知、相趣的安全大佬和朋友们，如果写得不好或不足之处，还请大家海涵！  
> 接下来我将开启新的安全系列，叫 “系统安全”，也是免费的 100 篇文章，作者将更加深入的去研究恶意样本分析、逆向分析、内网渗透、网络攻防实战等，也将通过在线笔记和实践操作的形式分享与博友们学习，希望能与您一起进步，加油~
> 
> 推荐前文：网络安全自学篇系列 - 100 篇
> 
> https://blog.csdn.net/eastmount/category_9183790.html

话不多说，让我们开始新的征程吧！您的点赞、评论、收藏将是对我最大的支持，感恩安全路上一路前行，如果有写得不好或侵权的地方，可以联系我删除。基础性文章，希望对您有所帮助，作者目的是与安全人共同进步，加油~

文章目录：

*   **一. Powershell 操作符**
    
*   **二. Powershell 条件语句**
    
    1.if 条件判断
    
    2.switch 语句  
    
*   **三. Powershell 循环语句**
    
    1.foreach 循环
    
    2.while 循环
    
    3.break 和 continue 关键词
    
    4.for 循环
    
    5.switch 循环  
    
*   **四. Powershell 数组**
    
    1. 数组定义
    
    2. 访问数组
    
*   **五. Powershell 函数**
    
    1. 自定义函数及调用
    
    2. 函数返回值
    
*   **六. Powershell 字符串及交互**
    
    1. 定义文本及转义字符
    
    2. 用户交互
    
    3. 格式化字符串  
    4. 字符串操作
    
*   **七. Powershell 注册表操作**  
    

作者的 github 资源：  

*   逆向分析：https://github.com/eastmountyxz/
    
    SystemSecurity-ReverseAnalysis
    
*   网络安全：https://github.com/eastmountyxz/
    
    NetworkSecuritySelf-study
    

> 声明：本人坚决反对利用教学方法进行犯罪的行为，一切犯罪行为必将受到严惩，绿色网络需要我们共同维护，更推荐大家了解它们背后的原理，更好地进行防护。该样本不会分享给大家，分析工具会分享。

一. Powershell 操作符
=================

常见的比较运算符包括：

*   -eq 等于
    
*   -ne 不等于
    
*   -gt 大于
    
*   -lt 小于
    
*   -le 小于等于
    
*   -contains 包含
    
*   -notcontains 不包含
    

```
67 -eq 5050 -eq 501gb -gt 1tb(1,2,3) -contains 1(1,2,3) -contains 2(1,2,3) -contains 4
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWr7ueaiaUyv4evUvX4rNe27QUn3HXBdW7xIKVhcnq3qXaIC40FCkjPPg/640?wx_fmt=png)求反运算符：

*   -not
    

```
$a=89 -gt 50$a-not $a
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWrBoISXdJYfZAtP7ZsWo9MuDTbgFWHbt2AEJZaRckPGU8GcDviaKicSvA/640?wx_fmt=png)

逻辑运算：

*   -and 与运算
    
*   -or 或运算
    
*   -not 非运算
    
*   -xor 异或运算
    

```
$true -and $true$true -and $false$true -or $false$false -or $false-not $true$true -xor $true
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWIokrcV9kTuLxeS8LB9gpr2ia8LyKoOZ0IgUeHQfibJFK23fR47Y6R4UA/640?wx_fmt=png)

比较数组和集合，从中筛选出不等于 0 的数字。

```
1,5,8,0,9 -ne 0
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWibdhZRMwk9icIytVLic0vPxxymRIuQgnx9eJicUSTlXSPl4OURseFzOYeQ/640?wx_fmt=png)

二. Powershell 条件语句
==================

1.if 条件判断
---------

if-elseif-else 条件判断，执行操作用大括号表示。

```
$num=100if($num -gt 90) {"大于90"} else {"小于等于90"}if($num -gt 100) {"大于100"} else {"小于等于100"}
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWxiawJSLFhT3cbKVic5LmraVgYXib7rCguR23MJq4wP36fibB4afqeHjnGQ/640?wx_fmt=png)

注意，if-else 中间可以增加新的判断 elseif，如下所示：

```
if($num -gt 100) {"大于100"} elseif ($num -eq 100) {"等于100"} else {"小于100"}
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWlVDkC70amibdCHvu1Ikq0KfSKBicDqbySWKG5FDYTJhGcZXVBEFkBP2w/640?wx_fmt=png)

2.switch 语句
-----------

Switch 语句主要用于多种情况的判断，这里在本地创建一个 test01.ps1 文件，并执行该代码。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWWZcC9Pk56rXFf73RP84ZDUsoR0AV0WE7kV8nnkQ1zriclhV8piaZ1ZPw/640?wx_fmt=png)

传统的 if 判断如下：

```
$num=56if($num -gt 50 -and $num -lt 60) {    "大于50并且小于60"} elseif ($num -eq 50) {    "等于50"}else{    "小于50"}
```

去到桌面 1019 文件夹，输入 “.\test01.ps1” 执行代码，再打印该文件的源代码。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWzv8miaQuzK0Ulv4aItjRyDyrfpYicdr3icxoL7kZcBvvKydKuUQ1FsQ9A/640?wx_fmt=png)

switch 语句如下：$_表示对变量取值。

```
$num=56switch($num){    {$_ -lt 50} {"此数值小于50"}    {$_ -eq 50} {"此数值等于50"}    {$_ -gt 50} {"此数值大于50"}}
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWQbtecQPLn1wEg1TLMVkPJgVMqEJQskQZ66edGU7ZTLaYssA8ibZoVcw/640?wx_fmt=png)

三. Powershell 循环语句
==================

1.foreach 循环
------------

这里定义数组采用 “$arr=1…10” 实现，表示 1 到 10 的数字，在调用 foreach 循环输出。

```
$arr=1..10foreach ($n in $arr){ $n*$n }
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWcWWoWxNcoqB77nlWMRDTB8dEWiamKRia41jcWfBEH86hFjrQrbVVTRxw/640?wx_fmt=png)

定义文件 “test03.ps1”，只输出偶数内容。

```
$arr=1..10foreach ($n in $arr){    if (($n%2) -eq 0 )    {        $n    }}
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWdw9TdqfNGXOl5kibczfLCN557TBNichkOhYNzDOs90NA6cGUknic2JIkw/640?wx_fmt=png)

接着利用 foreach 操作文件目录，将 C 盘 python34 文件夹下的路径全部提取出来，赋值到 file 中输出。

```
foreach ($file in dir c:\python34){    if($file.length -gt 1kb)    {        $file.name        $file.length    }}
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWHxEBz95QzFVwqwIiaJayYbibOJE6DbFZiaC6DIKRm60B2dDWzFekgH6Bw/640?wx_fmt=png)

原始文件内容如下所示：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWqDh0mqkFy4MoeTcTjGdWhNY1n3Edsfw1vRqLo8bC2CqLqWkwggzV5A/640?wx_fmt=png)  
也可以定义变量来指定路径

```
$path_value = dir c:\python34foreach ($file in $path_value){    if($file.length -gt 1kb)    {        $file.name        $file.length    }}
```

2.while 循环
----------

while 循环需要注意循环的终止条件，防止出现死循环，而 do_while 循环是先执行一次循环体，再进行判断。

下面这段代码是经典运算：1+2+3+…+99，文件名为 “test05.ps1”。

```
$i=1$s=0while($i -lt 100){    $s = $s + $i    $i = $i + 1}$s$i
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJW0Xiciaw1ocfCuEqKzBamn3e6YSEjibEa8aXnuAHsDWiboCj2RBfrO1StCQ/640?wx_fmt=png)

do_whlie 先执行循环体，再进行条件判断，如下所示：

```
$num=15do{    $num    $num=$num-1}while($num -gt 10)
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWGJKsq6PkF8EZDeiavbDmcpJE2De7TsWPgtdyGOgCryzPs54QtqvnItA/640?wx_fmt=png)

3.break 和 continue 关键词  

-------------------------

break 跳出整个循环，停止执行；continue 跳出当前循环一次，继续执行下一个判断。

break：下面这个代码当数值小于 6 继续执行，当其等于 4 停止循环。

```
$i=1while($i -lt 6){    if($i -eq 4)    {        break    }    else    {        $i        $i++    }}
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWyjC6QmgQfMHItRS3Kcry5iadNw1qvys3R7jBLNalLQFLZtwCgM0DDTg/640?wx_fmt=png)

continue：跳过了中间等于 4 的内容。

```
$i=1while($i -lt 6){    if($i -eq 4)    {        $i++        continue    }    else    {        $i        $i++    }}
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWIm57yWiciavNqXdiciclwTs0q5sXP4zJ5rhia2lo0dlg2L8pvHoerXG3CKA/640?wx_fmt=png)

4.for 循环
--------

利用 for 循环实现 1+2+…+100 的代码如下（test09.ps1）。

```
$sum=0for($i=1;$i -le 100;$i++){    $sum=$sum+$i}$sum
```

学习 Powershell 基础语法之后，更重要的是解决实际问题，后续作者将继续深入学习。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJW2juibocA8M83r5NicCB8CsxBHkdY1vRo7jUJDnECWkuMEMkgYrq1ejNQ/640?wx_fmt=png)

5.switch 循环
-----------

使用 switch 循环实现输出数组 1 到 10，并进行奇数和偶数判断。

```
$num=1..10switch($num){    default{"number=$_"}}$num=1..10switch($num){    {($_ % 2) -eq 0} {"$_ 数值是偶数"}    {($_ % 2) -ne 0} {"$_ 数值是奇数"}}
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWwcYhnm1MuaFAjicjoWlU6Aia4F54L23wJLFSlGib4iaZN3PHpfC8EjwOkg/640?wx_fmt=png)

四. Powershell 数组
================

1. 数组定义
-------

数组定义一种方法是逗号隔开不同的元素，另一种是通过两个点来定义数组。

```
$arr=1,2,3,4,5$arr=1..5
```

判断是否是一个数组，使用如下语句。

```
$arr -is [array]
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWDtrAyezuSGWh6drLLsS3nicLZcPSMjRHFtqzicIS6pFQ1CHu3fpC1YoA/640?wx_fmt=png)

数组可以接受不同的数值。

```
$arr=1,3.14,"yangxiuzhang"$arr$arr -is [array]
```

空数组定义如下：

```
$arr=@()$arr$arr -is [array]
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWLly9ePjm5w4EGjpeB9SNGujorzGIZibZLHVUw5XKpgTfydx6clnUAkw/640?wx_fmt=png)

下面简单比较只有一个元素数组和变量的对比。

```
$arr=,"hello"$arr$arr -is [array]$arr=1$arr$arr -is [array]
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJW8Ev9ViaP0hfscia8L7YuP0HQ88t9DLKgxPU78iauxqshoN8vpOsE5VO0A/640?wx_fmt=png)

数组也可以是一个变量或命令，此时它仍然是一个数组。

```
$arr=ipconfig$arr$arr -is [array]
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWZPqpWP7rUywZqNICyiaibpWDKqibbkquB9ibLX1PibLMGWbZRibZknicG5HZA/640?wx_fmt=png)

2. 访问数组
-------

首先定义一个多种类型的数组。

```
$arr=1,"hello world",(get-date)$arr
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWM7dWts2mQcHTI6AEFgqVFOkvicP4UuQlPcYYeyJBgMjhILib3zUh70bQ/640?wx_fmt=png)

访问数组特定元素，第一个元素，获取两个元素，获取最后一个元素。

```
$arr[0]$arr[0,1]$arr[-1]//提取部分元素$arr[0..2]
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWhaesnctmSdu7C33Lib9sZIdWwaXc0Xe19sqO4MdhPnHQWC4Yfnjse4w/640?wx_fmt=png)

获取数组元素大小调用 count 实现。

```
$num = $arr[0..2]$num.count
```

如何将数组倒序输出呢？如下所示。

```
$arr[($arr.count)..0]
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWfn3zpI2jkM6epX37OsTXUGJSfjlVEfIOJribKxTTj4V1t5l7FyqDUjQ/640?wx_fmt=png)

数组添加一个元素代码如下：

```
$arr=1,"hello world",(get-date)$arr+="csdn"$arr$arr.count
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWyM0Tib7qtg33rnNfvibvmT6GsVEQpOFBicjXbLf0xnp3Kz7x2ZKGtFQhg/640?wx_fmt=png)

更多数组操作，推荐读者结合实际应用进行学习。

五. Powershell 函数
================

1. 自定义函数及调用
-----------

函数通常包括函数名、参数、函数体，下面是定义及调用一个 myping 函数的代码（test11.ps1）。

```
function myping(){    ping www.baidu.com   }myping
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWjsXFAZtNqj9q1ia4ZA5bFgwKMP0ib0nVh3gTEHSoYOnh6Mh5ibZJTLrwA/640?wx_fmt=png)

同样，上面的代码可以修改为指定参数。

```
function myping($site){    ping $site}myping www.baidu.com
```

下面这个代码是接收两个参数并显示的功能。

```
function myinfo($name,$age){    $info="I am $name, and i am $age years old."    write-host $info}myinfo yxz,28
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWgY6z285jOShicU87WgaJdsicRlriaemtp9HKiccxNGHyGzGFZribbTb5AwA/640?wx_fmt=png)

2. 函数返回值
--------

函数返回值通过 return 实现，可以返回多个值。下面是 test13.ps1 例子。

```
function add($num1,$num2){   $sum=$num1+$num2   return $sum}add 2 4function fun($num1,$num2){   $sum=$num1+$num2   $sum.gettype()   $sum.gettype().fullname   return $sum}fun 2.2 4.3//多个返回值function other($num1,$num2,$num3){    $value=$num1,$num2,$num3    return $value}other 2.2 4 6
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWamYnnNyxvYuDbec6tLw8T73HeSIqTJ4FsSliaUDKicnE44JHzZ0J6aibw/640?wx_fmt=png)

六. Powershell 字符串及交互
====================

1. 定义文本及转义字符
------------

表达式中可以定义只，如下所示。同时，单引号和双引号可以相互嵌套，这和 JAVA、PHP、Python 中的变量套接类似。

```
"hello world $(get-date)""hello world $(5*7)""hello, my name is 'yangxiuzhang'"
```

输出结果如下图所示：  
![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWP911SC5dhMmmP2tPIIxehYkaEDCKAqdW0xUYjhQtnicrvkBaibObLtGw/640?wx_fmt=png)

在 Powershell 中，转义字符不再是斜杠（\）而是（`），如下所示。

*   `n 换行
    
*   `r 回车符
    
*   `t tab 键
    
*   `b 退格符
    
*   `’ 单引号
    

```
"hello,`n my name is `'yangxiuzhang`'"
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWh7BZMVaWW8vMvAWglumVgtp11ibH2TYgc8vRKP35e244b1VwquF5l7A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJW8F92lOGl3jjK79Ualqry8wPctRuOR5oJNMjZ2XQfjDAKib4mSykzzBQ/640?wx_fmt=png)

2. 用户交互  

----------

read-host 读取用户的输入。

```
$input = read-host "请输入您的姓名""您好！您输入的姓名是：$input"
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWgRS6ic16QL2azX0faict633SnczHnrIYDOm2BVdZvAfoIkj3cAYrT8XA/640?wx_fmt=png)

3. 格式化字符串  

------------

传统的多个变量输出方法：

```
$
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWRSXibOawsO2DMQdsKOvwG4uAz7EiaEHaMRepzAqc6C4s2sdDESXc7AgA/640?wx_fmt=png)

格式化字符串输出方法：

```
"My name is {0}, i am {1} years old, and my body is {2}, my height is {3}" -f $name,$age,$body,$height
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJW9UsOo41SMtTwEvFA2UqJJHqRSpCp1VYlSNQzkyDQVAM7HIHIMKp8hw/640?wx_fmt=png)

4. 字符串操作
--------

任何编程语言，都绕不过字符串操作，在网络安全领域，获取 ip 地址、URL 拼接、图片或脚本文件获取等都涉及字符串操作，下面进行简单分享。

字符串分割

```
$str="c:\windows\system32\demo.txt"$str.split("\")//数组类型，可以通过数组下标访问$str.split("\").gettype()
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJW8XwJrdE2tqYiaibonuEKmhv6I2utKoMxnIMDnPYvN9SmjCX9pwtkzvvA/640?wx_fmt=png)

获取图片名称

```
$str="https://blog.csdn.net/Eastmount/102781411/logo.png"$str.split("/")[-1]
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWdLf5wWxxzlyjib9Z4PYSU2nxezjCnia0mZOgxtlEB3iciahzblpGvmY0Lw/640?wx_fmt=png)

是否以某个字符结尾和是否包含某个字符。

```
$str.endswith("png")$str.contains("csdn")
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJW3iagGgQxTrZ3ZN2wvZs0xjDa6bdfld0n2xBSiaDiaOjd7kMr80IWgDqdQ/640?wx_fmt=png)

字符串比较，-1 表示两个字符串不一样，相等输出 0。

```
$str="https://blog.csdn.net/Eastmount/102781411/logo.png"$str.compareto("window")$str.compareto("https://blog.csdn.net/Eastmount/102781411/logo.png")
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWDsibfB5q0ibSjWPvTzPg2QBdXo88nicgUq6uiczrgolPe4DicGUmZibY16yg/640?wx_fmt=png)

其他操作如下：

```
//获取下标$str.indexof("s")//字符插入$str.insert(4,"yxz")//字符串替换$str.replace("n","N")
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWxlC93JI4iafMu310MArMryKFoJZUENvL6AtN0fnEJhia5m56ar3Aibqvg/640?wx_fmt=png)

七. Powershell 注册表操作
===================

**注册表（Registry，繁体中文版 Windows 操作系统称之为登录档）是 Microsoft Windows 中的一个重要的数据库，用于存储系统和应用程序的设置信息。**

早在 Windows 3.0 推出 OLE 技术的时候，注册表就已经出现。随后推出的 Windows NT 是第一个从系统级别广泛使用注册表的操作系统。但是，从 Microsoft Windows 95 操作系统开始，注册表才真正成为 Windows 用户经常接触的内容，并在其后的操作系统中继续沿用至今。

在 CMD 中输入 regedit 即可打开注册表，如下图所示。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWpUhctrickznme0oZzkBaMXiaGMOZD2fCDuZE5nj5iaIByPbU36WSQ1GXQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWxBbK0Pt4AyWwQkgXOPg4zNoh7D7V5kGBNoa0NvWHMSMeoicKj4UrRIA/640?wx_fmt=png)

注册表图形化界面显示如下，包括各种程序的配置信息，不能随便修改它，很容易造成系统故障。

*   **HKEY_CLASSES_ROOT**
    
    定义文档的类型 \ 类以及与类型关联的信息以及 COM 组件的配置数据
    
*   **HKEY_CURRENT_USER**
    
    包含当前登录到 Windows 的用户的配置信息
    
*   **HKEY_LOCAL_MACHINE**
    
    包含与计算机相关的配置信息, 不管用户是否登录
    
*   **HKEY_USERS**
    
    包含有关默认用户配置的信息
    
*   **HKEY_CURRENT_CONFIG**
    
    包含有关非用户特定的硬件的配置信息
    

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWpJXSQr4ZjcHfsnRXjYMFmQ47lgaavOKZbibHHgcMaRz7rJmhfS8JM7Q/640?wx_fmt=png)

在 Powershell 中显示注册表指令如下：

```
cd hkcu:dir
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWCS2kXVroRoutG73j0zPHwWzvcGlrjWr2fb5iaZyzYT3sYG40oBSNNqw/640?wx_fmt=png)

对应注册表图形界面。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWiaIKOnjpwjTWoPKHbIMEleJqKk7KNmxcDvRqB0JbmGfLy2icMlIX0Qww/640?wx_fmt=png)

```
cd system
dir
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJW5AzyH0iaaBE9dpepVlXfl7qPNfYv0gyAD6zk65GE7rfJl4O2Giaxia0Hg/640?wx_fmt=png)

对应图形界面。

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWN0CA0ahObgPQsO6AbVCdrG1cX7NPIMbsgglMDicU1qzwwj8a9QS8eYQ/640?wx_fmt=png)

其他访问也类似。

```
cd HKLM:
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJWbDRsLkqC9jOzc3yrricdK1uuFTgJsEeFrnBuV8okuL5vmk0qYVCtzmQ/640?wx_fmt=png)

对应图形界面：

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJW4BgOoS5iapQkxjsuCKqY8HT2v5nkVBR0mrB2GX3X3ic9iaZVpTkNTWBag/640?wx_fmt=png)

读取键值

```
get-itemproperty
```

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDRPZM80elyx40zXvLXY2CaJW9EicSG9HicRP390hibJnwQw20TYAaoBZWtf3u7w2zQbibRBMJhoP3iakgCw/640?wx_fmt=png)

设置键值

```
set-itemproperty
```

由于注册表不能随便修改，很容易造成系统故障，后续随着作者深入学习，了解更多网络安全中 Powershell 及注册表工作再来分享，希望读者喜欢该系列文章。

八. 总结
=====

写到这里，这篇文章介绍结束，主要内容：

*   **一. Powershell 操作符**
    
*   **二. Powershell 条件语句**
    
*   **三. Powershell 循环语句**
    
*   **四. Powershell 数组**
    
*   **五. Powershell 函数**
    
*   **六. Powershell 字符串及交互**
    
*   **七. Powershell 注册表操作**
    

如果你是一名新人，一定要踏踏实实亲自动手去完成这些基础的逆向和渗透分析，相信会让你逐步提升，过程确实很痛苦，但做什么事又不辛苦呢？加油！希望你能成长为一名厉害的安全工程师或病毒分析师，到时候记得回到这篇文章的起点，告诉你的好友。

学安全一年，认识了很多安全大佬和朋友，希望大家一起进步。这篇文章中如果存在一些不足，还请海涵。作者作为网络安全和系统安全初学者的慢慢成长路吧！希望未来能更透彻撰写相关文章。同时非常感谢参考文献中的安全大佬们的文章分享，深知自己很菜，得努力前行。编程没有捷径，逆向也没有捷径，它们都是搬砖活，少琢磨技巧，干就对了。什么时候你把攻击对手按在地上摩擦，你就赢了，也会慢慢形成了自己的安全经验和技巧。加油吧，少年希望这个路线对你有所帮助，共勉。

前文分享（下面的超链接可以点击喔）：

*   [[网络安全] 一. Web 渗透入门基础与安全术语普及](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247483786&idx=1&sn=d9096e1e770c660c6a5f4943568ea289&chksm=cfccb147f8bb38512c6808e544e1ec903cdba5947a29cc8a2bede16b8d73d99919d60ae1a8e6&scene=21#wechat_redirect)
    
*   [[网络安全] 二. Web 渗透信息收集之域名、端口、服务、指纹、旁站、CDN 和敏感信息](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247483849&idx=1&sn=dce7b63429b5e93d788b8790df277ff3&chksm=cfccb104f8bb38121c341a5dbc2eb8fa1723a7e845ddcbefe1f6c728568c8451b70934fc3bb2&scene=21#wechat_redirect)
    
*   [[网络安全] 三. 社会工程学那些事及 IP 物理定位](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247483994&idx=1&sn=1f2fd6bea13365c54fec8e142bb48e1d&chksm=cfccb297f8bb3b8156a18ae7edaba9f0a4bd5e38966bdaceeff03a5759ebd216a349f430f409&scene=21#wechat_redirect)
    
*   [[网络安全] 四. 手工 SQL 注入和 SQLMAP 入门基础及案例分析](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247484068&idx=1&sn=a82f3d4d121773fdaebf1a11cf8c5586&chksm=cfccb269f8bb3b7f21ecfb0869ce46933e236aa3c5e900659a98643f5186546a172a8f745d78&scene=21#wechat_redirect)
    
*   [[网络安全] 五. XSS 跨站脚本攻击详解及分类 - 1](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247484381&idx=1&sn=a1d459a7457b56b02e217f39e5161338&chksm=cfccb310f8bb3a06442b001fc7b38a0363b9fbd4436f450b0ce6fa2eeb5c796fc936ceb5d6fa&scene=21#wechat_redirect)
    
*   [[网络安全] 六. XSS 跨站脚本攻击靶场案例九题及防御方法 - 2](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485174&idx=1&sn=245b812489c845e875cf4bc4763747b7&chksm=cfccb63bf8bb3f2d537f36093de80dbeed5a340b141001d3ef8a9ac9d6336e0aaf62b013a54c&scene=21#wechat_redirect)
    
*   [[网络安全] 七. Burp Suite 工具安装配置、Proxy 基础用法及暴库入门](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485381&idx=1&sn=9a0230cf22eba0a24152cb0e73a37224&chksm=cfccb708f8bb3e1ecf68078746521191921f41d19a0b82cb3f097856dad7a85c4d9c34750b3f&scene=21#wechat_redirect)
    
*   [[网络安全] 八. Web 漏洞及端口扫描之 Nmap、ThreatScan 和 DirBuster](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485437&idx=1&sn=2a7179464207fa68b708297ec0db6f00&chksm=cfccb730f8bb3e2629edb5ca114de79723e323512be9538a4d512297f8728a3a9d7718389b60&scene=21#wechat_redirect)
    
*   [[网络安全] 九. Wireshark 安装入门及抓取网站用户名密码 - 1](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485465&idx=1&sn=8e7f1f5790bfe754affe0599a3fce1ee&chksm=cfccb8d4f8bb31c2ca36f6467d700f4e4d7821899a6d5173ac0b525f0f6227c8392252b5c775&scene=21#wechat_redirect)
    
*   [[网络安全] 十. Wireshark 抓包原理、ARP 劫持、MAC 泛洪及数据追踪](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485551&idx=1&sn=15f00e14f4376e179a558444de8ef0a5&chksm=cfccb8a2f8bb31b456499a937598e750661841b5ca166a12073e343a049737fa3131fd422dc5&scene=21#wechat_redirect)
    
*   [[网络安全] 十一. Shodan 搜索引擎详解及 Python 命令行调用](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485599&idx=1&sn=0c60c042911fc79287417c2385550430&chksm=cfccb852f8bb3144a89f6b0d0df6c185a208aa989d98f8c7e3b7d741dedc371b3ecb4e70a747&scene=21#wechat_redirect)
    
*   [[网络安全] 十二. 文件上传漏洞 (1) 基础原理及 Caidao 入门知识](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485787&idx=1&sn=0c75cf81c4234031273bced4dff0b25c&chksm=cfccb996f8bb3080fe9583043b43665095fd6935a4147a2bb0d1ab9b91a6cde99da4747c5201&scene=21#wechat_redirect)
    
*   [[网络安全] 十三. 文件上传漏洞 (2) 常用绕过方法及 IIS6.0 解析漏洞](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485833&idx=1&sn=a613116633338ca85dfd1966052b0b02&chksm=cfccb944f8bb305296a32dac7f0942e727d66dc9f710bfb82c3597500e97d39714ecd2ed18cf&scene=21#wechat_redirect)
    
*   [[网络安全] 十四. 文件上传漏洞 (3) 编辑器漏洞和 IIS 高版本漏洞及防御](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247485871&idx=1&sn=e6d0248e483dea9616a5d615f852eccb&chksm=cfccb962f8bb3074516c1ef8e01c7cb00a174fa5b1a51de3a49b13fd8c7846deeaf6d0e24480&scene=21#wechat_redirect)
    
*   [[网络安全] 十五. 文件上传漏洞 (4)Upload-labs 靶场及 CTF 题目 01-10](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488340&idx=1&sn=5b7bf5602294586f819340bd6190a34d&chksm=cfcca399f8bb2a8f746fc09c7142facc8ea17c008ba46dee423b90ff6abb3cd4486edf52d201&scene=21#wechat_redirect)
    
*   [[网络安全] 十六. 文件上传漏洞 (5) 绕狗一句话原理和绕过安全狗](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488396&idx=1&sn=67c1b13f041040c09c236bba99edfe0a&chksm=cfcca341f8bb2a5729778490db7441a4ddfdfa05dcc5f6322b4860db7780056f9f05f5bc0b3d&scene=21#wechat_redirect)  
    
*   [[网络安全] 十八. Metasploit 技术之基础用法万字详解及 MS17-010 漏洞](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488255&idx=1&sn=28b1f54fd420a0145cb95b842a36c567&chksm=cfcca232f8bb2b243bf4cbf5c1741c6af2c1fc666985d34b4f6b4a6ee3161d18975bb5ea18fc&scene=21#wechat_redirect)
    
*   [[网络安全] 十九. Metasploit 后渗透技术之信息收集和权限提权](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488639&idx=1&sn=dddd54eb0ba7cfdf71113a1f4a5c6548&chksm=cfcca4b2f8bb2da44c975ca12f16b4b76af351be4711ac7e77ca8622450a15c3af0172be3f9e&scene=21#wechat_redirect)
    
*   [[网络安全] 二十. Metasploit 后渗透技术之移植漏洞、深度提权和后门](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488738&idx=1&sn=8106362219d99ae6deb8aeca1f6b1dff&chksm=cfcca42ff8bb2d397c44b839700d92fd22e4ac60c403b96cba734bc523cb258dbd0db5309952&scene=21#wechat_redirect)
    
*   [[网络安全] 二十一. Chrome 密码保存渗透解析、Chrome 蓝屏漏洞及音乐软件漏洞复现](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247488883&idx=1&sn=65c362cc4c3958aa747716d17b29eeb3&chksm=cfcca5bef8bb2ca895525a1964425d1dfe74001a33e3b59b18bf902539cfc4d941dd96c33863&scene=21#wechat_redirect)
    
*   [[网络安全] 二十二. Powershell 基础入门及常见用法 - 1](http://mp.weixin.qq.com/s?__biz=Mzg5MTM5ODU2Mg==&mid=2247489093&idx=1&sn=216374f1db9af3e1bb4f9431b66237a3&chksm=cfcca688f8bb2f9e9fc25c1d1e21d3bceae0a9ff026f57e6e6df2ffa20597aa8356c15ea2280&scene=21#wechat_redirect)
    
*   [网络安全] 二十三. Powershell 基础入门之常见语法及注册表操作 - 2
    

2020 年 8 月 18 新开的 “娜璋 AI 安全之家”，主要围绕 Python 大数据分析、网络空间安全、人工智能、Web 渗透及攻防技术进行讲解，同时分享 CCF、SCI、南核北核论文的算法实现。娜璋之家会更加系统，并重构作者的所有文章，从零讲解 Python 和安全，写了近十年文章，真心想把自己所学所感所做分享出来，还请各位多多指教，真诚邀请您的关注！谢谢。2021 年继续加油！

![](https://mmbiz.qpic.cn/mmbiz_png/0RFmxdZEDROZePZ27y7oibNu4BGibRAq4HydK4JWeQXtQMKibpFEkxNKClkDoicWRC06FHBp99ePyoKPGkOdPDezhg/640?wx_fmt=png)

(By:Eastmount 2021-03-15 周日夜于武汉)

**参考文献：**

*   https://www.bilibili.com/video/av66327436 [推荐 B 站老师视频]
    
*   《安全之路 Web 渗透技术及实战案例解析》陈小兵老师
    
*   https://baike.baidu.com/item/Windows Power Shell/693789
    
*   https://www.pstips.net/powershell-piping-and-routing.html
    
*   https://www.pstips.net/using-the-powershell-pipeline.html