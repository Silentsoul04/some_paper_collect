> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/Fly_hps/article/details/81201904)

ThinkPHP在开启DEBUG的情况下会在Runtime目录下生成日志，而且debug很多站都没关的，所以影响应该很大吧

我们来看一下ThinkPHP3.2版本生成日志结构：

![](https://img-blog.csdn.net/20180725141430431?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

THINKPHP3.2 结构：Application\Runtime\Logs\Home\16_09_09.log

THINKPHP3.1结构：Runtime\Logs\Home\16_09_09.log

可以看到是 ：项目名\Runtime\Logs\Home\年份_月份_日期.log

这样的话日志很容易被猜解到，而且日志里面有执行SQL语句的记录，这里我随便找几个tp站测试一下：

http://demo.xxxxx.cc/Runtime/Logs/User/16_09_06.log 成功下载，并且找到一个用户的密码

![](https://img-blog.csdn.net/20180725141513882?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

成功登录：

![](https://img-blog.csdn.net/20180725141536502?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

我们再找一个案例:http://www.xxxxxx.com/Runtime/Logs/Home/16_09_06.log

![](https://img-blog.csdn.net/20180725141553730?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

成功登录：

![](https://img-blog.csdn.net/20180725141611866?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0ZseV9ocHM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**onethink官网测试**

http://www.onethink.cn/Runtime/Logs/16_09_07.log

**修复办法：**

删除Runtime/Logs下的所有文件，并将APP_DEBUG设置为false