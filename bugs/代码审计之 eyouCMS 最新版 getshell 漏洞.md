> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/JBzQ9xz7kVOm0Ll3yT-IRQ)

  

  

**奇安信攻防社区试运营**

试运营阶段发布、转载、互动皆有惊喜！

奇安信攻防社区是一个开放的社区，希望通过分享促进攻防技术的切磋与交流，以分享促成长，提升实战化攻防技术。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE6QW4ppq2JN2XnicWp1mTJI7pEZDBsubuRuGGcWXt8kUD1EoVmGdnhB23icj812Tx8w9VQu6M8Axazg/640?wx_fmt=png)

**扫码二维码体验**

**forum.butian.net**

**利用思路**
--------

1.  前台设置一个管理员的 session
    
2.  后台远程插件下载文件包含 getshell。
    

### **前台设置管理员 session**

在`application/api/controller/Ajax.php:219`

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE6QW4ppq2JN2XnicWp1mTJI7yoeiaSSmg2C13L19Q6ECsaz3EiaX5rUz9icxcNuapYPicS3cgNeNLBQpSA/640?wx_fmt=png)

get_token 函数是可以前台随意调用的，另外形参中的 $name 变量也是通过 http 传递进来的。跟进 token 函数，如下图所示。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE6QW4ppq2JN2XnicWp1mTJI7JYdMtIic3kict8PgOxRglWbunx6Xz96m4aWJrAib58uYpZ2j8NqBCqK8Q/640?wx_fmt=png)

箭头处有一个设置 session 的操作，名字是可控的，而值是请求时间戳 md5 的值。不可控。

既然可以设置任意 session 名字了，那么我们是否可以给自己一个管理员的 session 呢？

然后我们梳理一下后台管理员的登录逻辑。

在`application/admin/controller/Base.php:61`

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE6QW4ppq2JN2XnicWp1mTJI7pONYpFalO9BsbBdM4m2HHTscScjEOFN3kALZSo0KvpPibRLrscRNI5g/640?wx_fmt=png)

这里涉及到了两个 session，一个`admin_login_expire`，一个`admin_id`。

**admin_id** （该 session 有就即可，不会验证其值）

**admin_login_expire** （该 session 会做减法的校验，需要满足一定条件）

而我们设置的 session 中是 md5 字符串，因此在设置 **admin_login_expire** 时，需要挑选一个前面是很长一段数字的 **md5**，这样计算出来的结果就是负数，就满足该 if 条件了。

如图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE6QW4ppq2JN2XnicWp1mTJI7ibWyolp53ThGtQcKNKyFmRXMSluASzt0jV5cAaaqTueuSxMddWrrvAA/640?wx_fmt=png)

设置完这两个 session 后，我们继续看到 if 条件判断里还有一个 check_priv 函数，跟进查看：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE6QW4ppq2JN2XnicWp1mTJI7ogqea2iaXyZ4CAqtiac4PiaGj3AqEqSlP21AbpcJdNwPXsD6t0ibUGn4zg/640?wx_fmt=png)

这里就很简单了，继续设置一个 **admin_info.role_id**。满足比较小于 0 即可。

设置完三个 session 后，就可以进后台了，如图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE6QW4ppq2JN2XnicWp1mTJI7fwLdCoGSOnqf89vGozfLfnOEia2m5WIMmPVyTkrCddSCNnBV7vrHLHQ/640?wx_fmt=png)

### **后台远程插件下载 getshell**

在`application/admin/controller/Weapp.php:1285`

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE6QW4ppq2JN2XnicWp1mTJI7CWsSJOKWjyPjJrryAJOL6qImBCyibfdXBhxtY46zzQgiaWUgjZEeRdwg/640?wx_fmt=png)

这里传进来一个 $url，然后做一个 url 解析，需要满足 host 为`eyoucms.com`。

也就是程序限制只能从官网下载插件安装，但是这个校验太简单了，可以绕。

然后下文就是请求这个下载链接，做解压操作，并包含进来`config.php`。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE6QW4ppq2JN2XnicWp1mTJI7yyYoqqCGnEt6ylUSpuZzAvUloia9yia7wxuhicjic6f050zOA37HSv106g/640?wx_fmt=png)

然后开始准备制作恶意压缩包，也就是如下图所示的目录结构：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE6QW4ppq2JN2XnicWp1mTJI7qPlyibgFnvXTmVfeEFCsAb3r8sKvsib1JQ3V65eb4YgUGUV0x92krzCA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE6QW4ppq2JN2XnicWp1mTJI7ZC50PbQk8iaUDFVkm30PhV23rDzrOQ6kvLezdrJWdfF9HYU8VPF8MTw/640?wx_fmt=png)

然后去官网转一转，看看有没有上传的地方，还真有！在提问功能处可以上传图片

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE6QW4ppq2JN2XnicWp1mTJI7zqJfVR3r4pzQDAJTwnD1Qmh6y127zzqzC1BUKr2pKOmicKJqQK48TOA/640?wx_fmt=png)

然后我们把恶意压缩包改成图片后缀传上去，得到一个上传后的图片路径，在构造报文触发文件包含。

注: 文章中出现的 zc.com 的地址，为本地测试地址 127.0.0.1，请勿进行其他测试

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE6QW4ppq2JN2XnicWp1mTJI7syeQegFPzZVhibxoEHvzzm38YicpCyXvulVia54fwU5PxBvWFmXyA3oWA/640?wx_fmt=png)

生成 webshell。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE6QW4ppq2JN2XnicWp1mTJI7r8B6w4Mm9uvQTHGJbA8CQDKib9TncmU2icawTTqpFtEOWNjlibmYeicEJQ/640?wx_fmt=png)

END

  

【版权说明】本作品著作权归 balis0ng 所有，授权补天漏洞响应平台独家享有信息网络传播权，任何第三方未经授权，不得转载。

**文章首发于奇安信攻防社区**

  

  

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/WdbaA7b2IE6QW4ppq2JN2XnicWp1mTJI75YdZgicwbj8A2kYEy6CAibDTcEy9PXlWZnCRubjHMn4B3roGY9BNpt5Q/640?wx_fmt=jpeg)

balis0ng

  

爱吃鸭肠的黑客，擅长代码审计、漏洞挖掘。

  

  

  

  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE6QW4ppq2JN2XnicWp1mTJI72eh3mNiaQ3pXO5xFNM30ETiasomq9X0Asic5iciczGF5VDbia7MehLiarhVWg/640?wx_fmt=png)

  

分享、点赞、在看，一键三连，yyds。

![](https://mmbiz.qpic.cn/mmbiz_gif/FIBZec7ucChYUNicUaqntiamEgZ1ZJYzLRasq5S6zvgt10NKsVZhejol3iakHl3ItlFWYc8ZAkDa2lzDc5SHxmqjw/640?wx_fmt=gif)