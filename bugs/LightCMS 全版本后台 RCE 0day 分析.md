> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/7O3IDinOpFdmUdcjECro8Q)

Author：颖奇 L’Amore

Blog：www.gem-love.com

**0x00 前言**

LightCMS 是一款基于 Laravel 框架的 CMS，但前台没什么东西，主要作为一个后台管理系统。这个 CMS 我在春节期间就挖过了，但是因为全部都是一些数据库操作，最终放弃了。今天在伟大的郭院士的指导下，终于调出了这个郭院士挖到的 0day。

**0x01 文件上传**

这个 0day 是一个 Phar 反序列化打 Laravel RCE 的洞，因此需要能够将 phar 文件上传，在后台的内容管理中不难发现图像上传位点

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM4IF41ic13KJHTV02p1ZuUqmrOsPsEBegwyICsLNEOlZprYTlJv3gmpia8jRp8zoKUpZ7Vkaib2ia2Wibw/640?wx_fmt=png)

查看其 HTML 源代码，js Event 提交图片到一个上传接口

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM4IF41ic13KJHTV02p1ZuUqm50kyazVAMMA5rL9boov6spoS86JbQut0Y3ufJ387l31tOvBnmtT6sg/640?wx_fmt=png)

跟进其模板中来看一下

resources/views/admin/content/add.blade.php

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM4IF41ic13KJHTV02p1ZuUqmpLaFS5bojVPl5rUBJpzYQq8vNIZEwH0X111KKpMRccDouXgEa8iahlA/640?wx_fmt=png)

上传接口是渲染的一个 Laravel 的路由，来看一下这个路由，使用了 `NEditorController` 这个控制器

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM4IF41ic13KJHTV02p1ZuUqm52FITS5JYL7liawgBzE4Ovl2GUH4M7fpnTXpicXcPo7dIyOJuyYKvibHQ/640?wx_fmt=png)

跟到控制器，`uploadImage` 方法处理图像上传，没啥可以利用的

app/Http/Controllers/Admin/NEditorControllers.php

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM4IF41ic13KJHTV02p1ZuUqmicTThkr0muR1d7ic9Pk4GdSuwst3BG9jQnmRQaD84W3xDLrU1WBJFMzQ/640?wx_fmt=png)

##### 

**0x02 继续深入**

尽管这个上传没找到什么有价值的东西，我们可以在这个控制器下找到另一个比较有趣的方法

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM4IF41ic13KJHTV02p1ZuUqmvUEibabmjjUScVst8KjxAaf5cs4npylgxQrIeKMxAh6aCic3X0Ecm15A/640?wx_fmt=png)

`catchImage()` 方法接收 file 参数并传入 `fetchImageFile()` ，跟进

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM4IF41ic13KJHTV02p1ZuUqmjB2vwaMjakibtxeIhHeH8Un51UOSGrYB2Ecg64jjic7anpCORfLspHKw/640?wx_fmt=png)

在 `fetchImageFile()` 中，它会 curl 访问这个 url 并将读取到的内容传入 `Image::make()` 中

通过 debug 的不断跟进，最终来到了这个 `init()`，然后传入 `decoder->init()`

vendor/intervention/image/src/intervention/Image/AbstractDriver.php

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM4IF41ic13KJHTV02p1ZuUqmcTeyLr2LXhUFxvbXtn4E8DwuZUPcWsicBOnxfiaiakdIo2QQVRP9xwdFA/640?wx_fmt=png)

而接下来的这个 `init()` 则是一个 switch case，根据传入内容的类型返回不同的东西

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM4IF41ic13KJHTV02p1ZuUqmhMOVPmHPBwOZbZBHJM7N7ibrlvfgYBxHysibiaT3jm4duYCib2wSMAf8Gg/640?wx_fmt=png)

注意我们现在传入 `init()` 中的 `$data` 是提交的一个 url 的 curl 读取结果，而 `case $this->isUrl()` 看上去很有趣，因为 url 的内容似乎还可以是 url。那么不妨我们就直接将 url 的内容设置为一个新的 url 并传入，来看看 `initFormUrl()` 到底做了什么。

这里它继续读取了这个 url 的内容，然后作为 binary 数据处理

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM4IF41ic13KJHTV02p1ZuUqm2sufR4y6IzUEaSEB2PM3eibJzg6U8f4stB2Tck6zrtonrb3DOUib24Cw/640?wx_fmt=png)

然后我们来看看这个 `case` 的判断函数 `isUrl` 做了什么

```
public function isUrl()    {        return (bool) filter_var($this->data, FILTER_VALIDATE_URL);    }
```

这个方法只是利用 FILTER VAR 判断是否为 url，这意味着前面的 http 协议可以替换成其他协议，比如 phar 协议。

于是我们将 url 内容改成一个 phar，再次下断点，果然依旧进到了这里并且传给了 `file_get_contents()`

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM4IF41ic13KJHTV02p1ZuUqmAUMiaQaBvCspauqaAtfDzwtdicAmjKerxP61HBrN0nNHDITNQiaC5Rbjw/640?wx_fmt=png)

然后就会触发 phar 反序列化了。

**0x03 利用**

首先去网上找一个现成的 Laravel RCE 的 gadget，生成 phar 文件

然后来到内容管理 - 新增文章内容，上传文件，就会得到一个这样的图片 url：

```
http://127.0.0.1:12334/upload/image/202105/cbf1k61csMcM1pAheP34DxrcUIjDS1kF5bPaCYnC.gif

```

然后来到我们自己的 vps，新建一个 txt，内容为：

```
phar://./upload/image/202105/cbf1k61csMcM1pAheP34DxrcUIjDS1kF5bPaCYnC.gif

```

然后 POST 提交到这个路由即可触发 phar 反序列化

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM4IF41ic13KJHTV02p1ZuUqmCaDibGrVLuxcUGFzz3ibbgYFXd9nT1YzupHCDE48vCVCssPLYaDxByEQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_gif/ewSxvszRhM4IF41ic13KJHTV02p1ZuUqmXeVxHdG6028IqqHiaebHbHobOg7Rd1TOTmv45xd2Oy7z2Zaj7EfwNDA/640?wx_fmt=gif)

##### 

**0x04 后记**

这个洞还是比较容易被忽视的，虽然利用起来并不复杂，但是触发思路比较新颖，不容易被发现。只能说郭院士太强了。

公众号

最后  

-----

**由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，文章作者不为此承担任何责任。**

**无害实验室拥有对此文章的修改和解释权如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经作者允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的**