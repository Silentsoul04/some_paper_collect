> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/H6G87ezaQqCrRmi6SfWZ5Q)

**本文首发于****奇安信攻防社区**  

**社区有奖征稿**

· 基础稿费、额外激励、推荐作者、连载均有奖励，年度投稿 top3 还有神秘大奖！

· 将稿件提交至奇安信攻防社区（点击底部 阅读原文 ，加入社区）

[点击链接了解征稿详情](https://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247489051&idx=1&sn=0f4d1ba03debd5bbe4d7da69bc78f4f8&scene=21#wechat_redirect)

**前言**
------

在近期的授权项目中，遇到了一个目标，使用了 youdiancms，需要获取权限，进而进行审计，本次代码审计过程先发现 SQL 注入漏洞，继续审计发现 getshell 的漏洞，本文将本次审计过程书写下来, 仅作为学习研究，请勿用作非法用途。

**0x01 未授权 SQL 注入**
-------------------

首先拿到源码一看，发现该系统是基于 THINKPHP3 开发的。

在`App/Lib/Action/HomeBaseAction.class.php:16`

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE7sC0c1ImrvqiaDj0JIgP4cyINDTlMOwL7xOJmGxGW0zQX355IZAqjeX9fNyepc9Zacb0wpsbmHVcQ/640?wx_fmt=png)

cookie 可控，然后赋值给了`$this->_fromUser`

跟踪一下`$this->_fromUser`的引用。

在`App/Lib/Action/Home/ChannelAction.class.php:732`

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE7sC0c1ImrvqiaDj0JIgP4cy43X6ULRztLQbA8w9d4tHV5CyULnegXiajEMpGkom9ib5G45KosXYAF2w/640?wx_fmt=png)

这里将`$this->_fromUser`带入到了`hasVoted`函数中，跟进该函数：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE7sC0c1ImrvqiaDj0JIgP4cyxIicYxpOBq9cXUthIShpKj4YichB5WnvUO3bkMhwNM8cGtMXujqNmmKQ/640?wx_fmt=png)

很明显，TP3 的 where 注入。

延时注入 payload 如下:

```
GET /index.php/Channel/voteAdd HTTP/1.1
Host: localhost
Content-Length: 2
Accept: application/json, text/javascript, */*; q=0.01
X-Requested-With: XMLHttpRequest
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Cookie: youdianfu[0]=exp;youdianfu[1]==(select 1 from(select sleep(3))a)
Connection: close
```

**0x02 绕过登录到 getshell 过程**
--------------------------

### **0x0201 流程思路**

1.  验证码处可以设置任意 session
    
2.  碰撞 md5 让 AdminGroupID==1（超级管理员）
    
3.  后台修改模板插入 phpcode 实现代码执行
    

### **0x0x202 任意 session 设置**

在`App/Lib/Action/BaseAction.class.php:223`

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE7sC0c1ImrvqiaDj0JIgP4cyajoEl5DnNNx8oic6CFUpPuiaBMoicibDHQgpYgBhJSibNUa8qz1y6TYgHKg/640?wx_fmt=png)

这个函数挺有意思的，本来是个生成验证码的操作，但是没想到所有的参数都是用户可以控制的，特别是这个`$verifyName`还可控。跟进`buildImageVerify`看看如何设置的`session`。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE7sC0c1ImrvqiaDj0JIgP4cyYyNIfftdibOicRwv16LE6dHnuavR1icF6L7eKFFFHNB5aSSXgB8dKbTibg/640?wx_fmt=png)

红框处设置了 session，并且 session 的键名我们是可控的，但是值不可控，是个 md5 值。

然后我们去看看管理员的校验函数。在`App/Lib/Action/AdminBaseAction.class.php:7`

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE7sC0c1ImrvqiaDj0JIgP4cyH2kVicaKpgsSHKHZjLCRI16tia3HsWW8TJz21tzLWdXL0dibBIkSrIxiaw/640?wx_fmt=png)

起作用的就两个函数，`isLogin`和`checkPurview`。跟进第一个看看：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE7sC0c1ImrvqiaDj0JIgP4cyOiaARibaeWSHic5KcvOyAXazyOS6XYtoEF5qNSZZ54ZH1UyFWaKmNpXAA/640?wx_fmt=png)

这个函数很简单，就简单的判断 session 是否存在，我们可以通过上文的验证码函数来设置。

然后就是 checkPurview 函数。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE7sC0c1ImrvqiaDj0JIgP4cy5Ozr8ndv15qtbDlI2HNiclNa4r8jev84wSA13p08DMFG7thjaaIuzmQ/640?wx_fmt=png)

这里判断了`AdminGroupID`的值，当等于 1 的时候就是超级管理员，由于这里是个弱类型比较。所以上文设置 session 中的 md5 是可以碰撞的。

编写脚本得到超级管理员的 session 了，然后登录。

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE7sC0c1ImrvqiaDj0JIgP4cyaX2vLBFmiaznyibMhteyOXIeiabsa94VicSx4SmlegTZN8Z9wxd3TZIciag/640?wx_fmt=png)

### **0x0203 后台 getshell**

后台模板管理，可以修改模板，但是对 <?php 有检测，如图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE7sC0c1ImrvqiaDj0JIgP4cyMj5saicHw9OicE1MfHn3HlzePYLZStUeQctQb92xvhl60ypCCD02K1sQ/640?wx_fmt=png)

我们可以用`<?=?>`来绕过这个检测。

如图所示：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE7sC0c1ImrvqiaDj0JIgP4cybsoriatSHpiadRVVAxpQCVcia3TS5tcWN0BBLcia4t7IQ6JCggjVicUqk0w/640?wx_fmt=png)

访问首页即可触发：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE7sC0c1ImrvqiaDj0JIgP4cy3tEjk5Q0fW1g8y6heNJ1lxJzsgrG9OOa7TEK99YPV8MWAMdO45TvpA/640?wx_fmt=png)

注：zc.cn 为本地 127.0.0.1 的地址，并非 zc.cn 的域名

END

  

【版权说明】本作品著作权归带头大哥所有，授权补天漏洞响应平台独家享有信息网络传播权，任何第三方未经授权，不得转载。

  

  

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/WdbaA7b2IE7sC0c1ImrvqiaDj0JIgP4cy15M14TPFUGdGuvV0LRiaknURXhaZicASYo6JTfDHRcOh2YOTg7BRT4Nw/640?wx_fmt=jpeg)

带头大哥

  

一个神秘且优秀的补天白帽子

**敲黑****板！转发≠学会，课代表给你们划重点了**

**复习列表**

  

  

  

  

  

[记一次文件上传的曲折经历](http://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247489568&idx=1&sn=56beddb5ef58d9556d75bbd8dd146dd2&chksm=eafa506cdd8dd97a9420b312770c8ea1bdeb0394ce38ef54834e60e91c0b404f934ac494ca3c&scene=21#wechat_redirect)

  

[代码审计之 eyouCMS 最新版 getshell 漏洞](http://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247489781&idx=1&sn=a2d0ccd466dfa95067f223c8318a316d&chksm=eafa50b9dd8dd9af45ef4fcf23074aeecc196dc72b3ff447282a9e6ea9904dcc08fe72430d30&scene=21#wechat_redirect)

  

[硬核黑客笔记 - 怒吼吧电磁波 (上)](http://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247489491&idx=1&sn=4ab4db01f63ca3c82c155d82c92b2662&chksm=eafa5f9fdd8dd689bc8cbcde1bb488372f50008619d25ca292753b0356eba4ea405db20349b4&scene=21#wechat_redirect)

  

[从 WEB 弱口令到获取集权类设备权限的过程](http://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247489456&idx=1&sn=a156b1a398e53e0c0d1cc1b8f4bc78f7&chksm=eafa5ffcdd8dd6eae463303a99720247160a79218e86ee494c5defbf6e9d4be0a9b63b13775c&scene=21#wechat_redirect)

  

[一个域内特权提升技巧 | 文末双重福利](http://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247489414&idx=1&sn=f9addeb81e8a2ea160e043ee2b19a4cf&chksm=eafa5fcadd8dd6dc815cdbd43b7311a447ccabb35c98519237448cb643d183b2c264e073bc16&scene=21#wechat_redirect)

  

[php 无文件攻击浅析](http://mp.weixin.qq.com/s?__biz=MzI2NzY5MDI3NQ==&mid=2247489820&idx=1&sn=5fe5827ab1f5ef7175449be8a822bf08&chksm=eafa5150dd8dd8463acab35b71b0db213923508055ed0e1dd106788206e42de8dc80c5d58232&scene=21#wechat_redirect)

  

![](https://mmbiz.qpic.cn/sz_mmbiz_png/WdbaA7b2IE6D8InhXuGX2q6Cbw7zhMJLFcmlcnz38EApnEkFiaISicklcwbo3gnI17t54PqyYOE8LV4yczIfjdqw/640?wx_fmt=png)  

  

分享、点赞、在看，一键三连，yyds。

![](https://mmbiz.qpic.cn/mmbiz_gif/FIBZec7ucChYUNicUaqntiamEgZ1ZJYzLRasq5S6zvgt10NKsVZhejol3iakHl3ItlFWYc8ZAkDa2lzDc5SHxmqjw/640?wx_fmt=gif)

  

点击阅读原文，加入社区，获取更多技术干货！