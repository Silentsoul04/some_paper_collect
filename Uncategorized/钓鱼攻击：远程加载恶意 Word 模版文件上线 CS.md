> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/E8o2ju27N_ZDTLghRgihbA)

利用 Word 文档加载附加模板时的缺陷所发起的恶意请求而达到的攻击目的，所以当目标用户点开攻击者发给他的恶意 word 文档就可以通过向远程服务器请求恶意模板并执行恶意模板上的恶意代码。

这里，我们借助 CobaltStrike 生成 office 宏病毒，在将恶意宏嵌入到 Word 模板中，诱使受害者远程打开并加载带有宏的恶意 Word 模版，至目标主机成功上线 CobaltStrike。

*   缺点  
    目标主机的网速决定了加载远程模版的速度。有可能文件打开的会特别慢 (例如将远程模版放在 github)，受害者可能在文件打开一半的时候强制关闭 word。
    

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9o2IWMLmAs14CO0J0XzpdDZ1EpIv92CaYtOwJpy8b7jBqPKJw9zOrLmiarFicE4xFmlhPoYktxuBgQ/640?wx_fmt=png)

如上图正在加载位于 github 上的恶意模板内容，有点慢，我们可以放在 vps 上，这样速度就快些了。

*   优点  
    因为是远程加载，所以免杀效果十分不错。基本不会被杀毒软件拦截。
    

第一步：制作一个恶意的模版并确保能够上线  

这里以 cs 的宏木马为例。

依次点击攻击——> 生成后门——>MS Office Macro：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9o2IWMLmAs14CO0J0XzpdD6S4B3aTtdicQe0Ugrg2D00Oektn6icoGr8ZEFASCLicCeicBcwwqbcg5icA/640?wx_fmt=png)

然后选择一个监听器，点击 Generate：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9o2IWMLmAs14CO0J0XzpdDdGg8PzicGmMYANDC3MiaYRbmJ45lVb7jo4IFicNPUzdXSGgrJjMwA0H2w/640?wx_fmt=png)

然后点击 Copy Macro：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9o2IWMLmAs14CO0J0XzpdDnnlCJsQv6uhm5sS2nPf4AmuoH9luaL4dP69QEUn8HgFIVKKw2tff8A/640?wx_fmt=png)

此时便将恶意的 VB 代码获取到了剪切板中。

然后打开 word 编辑器，在工具栏的空白区域右键，点击自定义功能区：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9o2IWMLmAs14CO0J0XzpdDu5FGZI9kQEz1v33jADedYx160ALlI8D6DzLjD3ibcmL9D3vEkkiatjFA/640?wx_fmt=png)

勾选开发工具选项：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9o2IWMLmAs14CO0J0XzpdD4lBH7RxcMJEINu2ZuzwNhhGdQgWVY03ykCHbxxTUg2xHC813QGDumg/640?wx_fmt=png)

此时就会出现开发工具这一栏，并点击开发工具中的 “Visual basic”：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9o2IWMLmAs14CO0J0XzpdDSGA3Md0s0ibcOSoDoSKkwh3mh7h5bc75NibJwCc1UQsKH2v10uydAWxQ/640?wx_fmt=png)

将恶意代码复制到 project 的指定地点如下图所示：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9o2IWMLmAs14CO0J0XzpdDE1pFZ6XLUPWnuNm94XfEpc8uncYiaQiaUhZn6icJUnos4ibJSPCcgLKkyg/640?wx_fmt=png)

然后关闭代码框，将这个 word 文件另存为一个启用宏的 word 模版文件 (*.dotm)：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9o2IWMLmAs14CO0J0XzpdDkO1kQSZoMg4aibBZJKtab52icBvh23W3KgABHIvr6C11dlVI0X6ghl0A/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9o2IWMLmAs14CO0J0XzpdDQIxAgMdJaGxloHgpECWP37EiaM242FlapPPG6Hy0LTHFL4cAXHeKQcA/640?wx_fmt=png)

这时候可以先测试一下模版能否上线，操作为在模版文件上右键打开，双击是无法打开模版文件的，在模版文件上双击默认是以此模版创建新文件，切记。

右键打开后，CS 成功上线：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9o2IWMLmAs14CO0J0XzpdDuiamZH0B8iaZMQFUG8ymtttUFX4WHyXgM1IvmKMXmcBs9icaXZPlOwZYQ/640?wx_fmt=png)

测试完成。

#### 第二步：制作远程加载恶意宏模版的 docx 文件

1. 将恶意文件上传到服务器

首先将刚才已经制作好的含有恶意代码的模版文件上传到服务器上，这里采用 github 来做这个实验，如下图将恶意模板恶意文件上传到 github。

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9o2IWMLmAs14CO0J0XzpdDLKO2kbvAIhGIXNJOS6cyKfDFHyqCVoQJMaVOeFr38IysDuI7k1vCkQ/640?wx_fmt=png)

点击上图中上传的恶意文件，会进入下图这个页面：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9o2IWMLmAs14CO0J0XzpdDQhbBm35hEFbQUVic2pbankZ6ia6m3j6ttHdVBDJQjnzUpib4TVgDQIrPA/640?wx_fmt=png)

复制这个页面的 url：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9o2IWMLmAs14CO0J0XzpdD3dhAiam6dd5B7V3jnazE2fBkJK8vANw14IKMQBxsECQ3TEKCS5bGvDA/640?wx_fmt=png)

并在 url 后面加上? raw=true，最终结果如下，把这行保存下来等下会用到。

2. 加载服务器上的恶意文件  

打开 word 找一个任意的模版双击使用，然后什么都不用改直接保存在任意路径下。

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9o2IWMLmAs14CO0J0XzpdD6Kcag4WahqHhb2unVrnWeibcc3XRMKh8N4pKia0SBNKJmmsbWzwI5Ogw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9o2IWMLmAs14CO0J0XzpdDJG1LTjVd1AZcH2xaFrI3eJONbXdvwbPZIL5EEibic4P8C1GMb6MQfJGA/640?wx_fmt=png)

将文件改名，改为 zip 结尾。

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9o2IWMLmAs14CO0J0XzpdDNmGF9mpW4D8hNI2pHwKgibEFscpAmZQpbHdw6T33tBMPyibaOMFr7a6w/640?wx_fmt=png)

将其解压缩：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9o2IWMLmAs14CO0J0XzpdDIZrsjv0wlLUTsaQtooQmtAINJcCvwtTlUQSibQiac7ibXQ8evlWRPfq7Q/640?wx_fmt=png)

进入 word 文件夹中的_rels，找到 settings.xml.rels 文件

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9o2IWMLmAs14CO0J0XzpdD32bpwjoI2BudFc1gkQgBqS5xuMvwVbDK7iaa3ZqOHYy1amGFBCeHC3Q/640?wx_fmt=png)

编辑这个文件，将其的 target 属性的值改为我们上面的那个 url，也就是 https://github.com/MrAnonymous-1/test/blob/master/Doc1.dotm?raw=true

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9o2IWMLmAs14CO0J0XzpdDKzQraXLDp0mNtibj8owbricLjmqGjlCt7PgoiatNibP2SvJQeRrmTKgEqw/640?wx_fmt=png)

接下来将刚才解压生成的文件压缩回去：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9o2IWMLmAs14CO0J0XzpdD7eGCkPLzIPYib8w025uT0bRE0ykic48ibq3LWWENuQxD4qgLB2npnNXtg/640?wx_fmt=png)

并且将生成的压缩文件改名为后缀名为 docx 的文件。

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9o2IWMLmAs14CO0J0XzpdD1EagMIJx85XCIoTDBJgG8S0lZ8k6b3eWgjib06FohVDhbjiaSqm9b69g/640?wx_fmt=png)

将最终生成的恶意文件——我的简历. docx 用邮箱钓鱼、qq 或微信文件发送给受害者，当受害者双击打开 “我的简历. docx” 文件是，恶意代码会执行，目标主机会上线

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9o2IWMLmAs14CO0J0XzpdDGySDiaHb0FBNXCU5rkh8NVnXCCFRmj7FRCtFdGyL20750kCHfvjWFDA/640?wx_fmt=png)

进程名是 rundll32.exe。

然后把这个文件扔到 virustotal.com 上查杀病毒，发现只有两家公司的杀毒软件认为其是病毒：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9o2IWMLmAs14CO0J0XzpdDBauwMWC9HTPWMTeq09vTKIV1fuOCfhdej4uFbyBn5D7BzCg0fDIGzA/640?wx_fmt=png)

注意：将该恶意文档发给其他人，只要他是用 word 打开（wps 不行），并且开启了宏，我们的 CS 就会收到弹回来的 shell。

word 开启或禁用宏：文件——> 选项——> 信任中心——> 信任中心设置

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9o2IWMLmAs14CO0J0XzpdDQPymURFRzn4OLHlva1EL1UBFKqpGnrA7vWsC34zibfbtdRDAhC0BbaQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9o2IWMLmAs14CO0J0XzpdD8YFH4OWJyJNd4syYZ5O8zo847KnROcjLsibpyLmAK8oQD9pCBcvgGyQ/640?wx_fmt=png)

默认情况下是 “禁用所有宏，并发出通知”，这种情况下，当我们打开恶意文件时，会询问你是否 “启用内容”：

![](https://mmbiz.qpic.cn/mmbiz_png/Uq8Qfeuvou9o2IWMLmAs14CO0J0XzpdD28aAcFJqtPuuAZFZZ5TBd3QYqTvEmiac9RRCgjFyC2Z6EX1M39LX5icw/640?wx_fmt=png)

这时，受害者只有点击了 “启用内容” 后恶意代码才会执行，目标主机才能上线。

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)

**推荐阅读：**

[**干货 | 恶意代码分析之 Office 宏代码分析**](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247493020&idx=1&sn=a6e06e20e32fd14d7723ca014eac3a04&chksm=ec1cb0a3db6b39b5bd5c0bb1d77057be1faa86c3b5a8d1b7025220bbcbbe8872d828be18c024&scene=21#wechat_redirect)  

[**Office 如何快速进行宏免杀**](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247492416&idx=1&sn=c444b28f7aa67e9ee15d42bc1aeef10d&chksm=ec1cb67fdb6b3f69d33753fd68cad86f401c07f5fddb3157c81468cab144978a5fb3840da037&scene=21#wechat_redirect)  

[**红队攻防系列之花式鱼竿钓鱼篇**](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247489949&idx=1&sn=d2a0d19a0c9c4cff7e404be603d144a1&chksm=ec1f4ca2db68c5b487d152775be8b2f71fd1b79a2f4aa7fc083087e6b97d0a09a30b609aba9a&scene=21#wechat_redirect)  

[**红队攻防之邮箱打点入口**](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247489324&idx=1&sn=acec232cc8efb751a77a62fbaee288e4&chksm=ec1f4213db68cb05d6556ba6d3d5a79dd2c22c2626e1836881e0ba35fb4955ed819959467c78&scene=21#wechat_redirect)  

**[红队攻防系列之花式鱼竿钓鱼篇](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247489949&idx=1&sn=d2a0d19a0c9c4cff7e404be603d144a1&chksm=ec1f4ca2db68c5b487d152775be8b2f71fd1b79a2f4aa7fc083087e6b97d0a09a30b609aba9a&scene=21#wechat_redirect)**

[![](https://mmbiz.qpic.cn/mmbiz_jpg/Uq8QfeuvouibVuhxbHrBQLfbnMFFe9SJT41vUS1XzgC0VZGHjuzp8zia9gbH7HBDmCVia2biaeZhwzMt8ITMbEnGIA/640?wx_fmt=jpeg)](http://mp.weixin.qq.com/s?__biz=MzI5MDU1NDk2MA==&mid=2247493466&idx=1&sn=ebea4452c1f684fbcdde430d13d1ab1b&chksm=ec1cb265db6b3b73194d336274ea6c79ed35ebe2d65f9abbe5b4db760022debb2e533f163218&scene=21#wechat_redirect)

**点赞 在看 转发**  

作者：Mr.Anonymous

文章来源：whoamianony.top

如有侵权，请联系删除

![](https://mmbiz.qpic.cn/mmbiz_gif/Uq8QfeuvouibQiaEkicNSzLStibHWxDSDpKeBqxDe6QMdr7M5ld84NFX0Q5HoNEedaMZeibI6cKE55jiaLMf9APuY0pA/640?wx_fmt=gif)