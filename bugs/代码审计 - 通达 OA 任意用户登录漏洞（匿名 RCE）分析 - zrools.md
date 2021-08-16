> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.zrools.org](https://www.zrools.org/2020/04/23/%E4%BB%A3%E7%A0%81%E5%AE%A1%E8%AE%A1-%E9%80%9A%E8%BE%BEOA-%E4%BB%BB%E6%84%8F%E7%94%A8%E6%88%B7%E7%99%BB%E5%BD%95%E6%BC%8F%E6%B4%9E%EF%BC%88%E5%8C%BF%E5%90%8DRCE%EF%BC%89%E5%88%86%E6%9E%90/)

官网更新了 v11.5 版本后，漏洞分析和 PoC 逐渐浮出了水面，其实这漏洞结合后台的一些功能是可以进一步实现匿名 RCE 的。

[](#漏洞说明 "漏洞说明")漏洞说明
--------------------

伪造任意用户（含管理员）登录漏洞的触发点在扫码登录功能，服务端只取了 UID 来做用户身份鉴别，由于 UID 是整型递增 ID，从而导致可以登录指定 UID 用户（admin 的缺省 UID 为 1）。

[](#影响版本 "影响版本")影响版本
--------------------

通达 OA v2017、v11.x < v11.5 支持扫码登录版本。

[](#漏洞分析 "漏洞分析")漏洞分析
--------------------

v11.5 更新修复了 2 个地方：

*   客户端扫码登录接口；
*   Web 端扫码登录接口。

### [](#Web端扫码登录过程 "Web端扫码登录过程")Web 端扫码登录过程

Web 端扫码登录流程大致是这样：

*   **第一步**

Web 端访问 `/general/login_code.php?codeuid=随机字符串` 生成一个二维码，`codeuid`作为这个二维码凭证。

*   **第二步**

Web 端通过循环请求`/general/login_code_check.php`，将`codeuid`发送到服务端，判断是否有人扫了这个二维码。

*   **第三步**

移动端扫码这个二维码，然后将`codeuid`等数据发送到`/general/login_code_scan.php`服务端进行保存。

*   **第四步**

Web 端通过`login_code_check.php`取得`codeuid`等扫码数据后（其实取数据这一步已经产生`$_SESSION["LOGIN_UID"]`登录了），再通过 Web 端发送到`logincheck_code.php`进行登录。

Web 端登录请求脚本如下：

[![](https://www.zrools.org/images/2020/04/23/01.png)](https://www.zrools.org/images/2020/04/23/01.png)

可以看到最终的登录数据只发送了`UID`到服务端，从而导致了任意用户登录。

### [](#Web端补丁分析 "Web端补丁分析")Web 端补丁分析

这里对比一下`v11.4`和`v11.5`修复前和修复后的源代码：

源文件： /logincheck_code.php

[![](https://www.zrools.org/images/2020/04/23/02.png)](https://www.zrools.org/images/2020/04/23/02.png)

修复后的版本对 UID 进行了初始化，并增加 token 的校验。

从 redis 中取得 token 并进行`td_authcode()`解密，然后从中取出`UID`，避免了前段直接控制`UID`。

再看下 token 的生成方式，源文件： /general/login_code_scan.php

[![](https://www.zrools.org/images/2020/04/23/03.png)](https://www.zrools.org/images/2020/04/23/03.png)

通过`PHPSESSID`从在线用户表`user_online`中取`UID`，然后封装数据 MD5 哈希后存入 redis 中。

### [](#客户端补丁分析 "客户端补丁分析")客户端补丁分析

客户端和 Web 端大同小异，登录也加了`token`校验：

```
/ispirit/interface/login.php
```

[![](https://www.zrools.org/images/2020/04/23/04.png)](https://www.zrools.org/images/2020/04/23/04.png)

同样是增加了`token`校验。

[](#漏洞利用 "漏洞利用")漏洞利用
--------------------

手动漏洞复现可以在浏览器直接打断点修改数据或者 BurpSuite 修改。

脚本构造数据包的时候只需两步即可：

*   将数据写入缓存： /general/login_code.php
*   从缓存读取登录： /logincheck_code.php

### [](#任意用户登录-PoC "任意用户登录 PoC")任意用户登录 PoC

知道了漏洞过程，那么实现 PoC 就很简单了，以获取 Web 目录绝对路径为例：

*   tongda_v11.4_get_webroot_poc.py

修改脚本里的`oa_addr`为目标 OA 地址，成功后会输出当前 OA 的安装绝对路径。

```
$ python3 tongda_v11.4_get_webroot_poc.py

webroot:  C:\\MYOA\\webroot
cookies:  PHPSESSID=xxxxx
```

### [](#匿名-RCE-ExP "匿名 RCE ExP")匿名 RCE ExP

有了后台管理权限和 Web 目录绝对路径，可以利用 MySQL 日志进一步写 Shell，实现匿名 RCE：

*   tongda_v11.4_rce_exp.py

修改脚本里的`oa_addr`为目标 OA 地址，成功后会在`/api/`目录下生成一个`test.php`，密码为：`cmd`，`GET`请求方式。

```
$ python3 tongda_v11.4_rce_exp.py

webroot:  C:\\MYOA\\webroot
cookies:  PHPSESSID=xxxxx
webshell: (GET) http://192.168.0.3:8080/api/test.php?cmd=ipconfig
```

[![](https://www.zrools.org/images/2020/04/23/05.png)](https://www.zrools.org/images/2020/04/23/05.png)

[](#扩展思考 "扩展思考")扩展思考
--------------------

我们脑洞一下这个补丁是否可以绕过：

### [](#认证地址一： "认证地址一：")认证地址一：

源文件： /logincheck_code.php

```
$authInfo = $redis->get("OA:authcode:token:" . $token);

$Info = td_authcode($authInfo, "DECODE");
```

如果我们能找到一个可控的:

```
TRedis::redis()->setex($xxx, $xxx);
```

而`td_authcode()`的 key 是固定的，从而就可以伪造`token`进行任意用户登录。

### [](#认证地址二： "认证地址二：")认证地址二：

源文件： /ispirit/login_code_check.php

```
$login_codeuid = TD::get_cache("CODE_LOGIN_PC" . $codeuid);

...

$code_info = TD::get_cache("CODE_INFO_PC" . $login_codeuid);

...

$UID = intval($code_info["uid"]);
```

如果我们能找到一个可控的：

```
TD::set_cache($xxx, $xxx);
```

也是可以伪造任意用户登录的。

**PoC & ExP 传送门：** [https://github.com/zrools/tools/tree/master/python](https://github.com/zrools/tools/tree/master/python)