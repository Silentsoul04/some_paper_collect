> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzAxMjE3ODU3MQ==&mid=2650477052&idx=2&sn=1734682ea37783cecd5decb52af71079&chksm=83ba7418b4cdfd0ea5a77e84697bc38a581189031ab0613b75dca63bdd85763fcb35245344ba&mpshare=1&scene=1&srcid=0819aYRtGnTYhO6aL3nuq3Pi&sharer_sharetime=1597810639188&sharer_shareid=c051b65ce1b8b68c6869c6345bc45da1&key=10b5f81a683662234382cfa682c4367a7fb9e5a12008ee8a9b3928eff6d42d53eb183bcefa079b844bfa6849429e99fbfe6dd3f2c5c42be4a79c1957ad44a21c123acd3ff449b072aace41b8bc6067117072f151b7d07e81f3d3f0326393cf252486d5f94ff49822f5cff5ba6e0c0ffdef5e21156d2887d4a1bb36384b244c22&ascene=1&uin=ODk4MDE0MDEy&devicetype=Windows+10+x64&version=62090529&lang=zh_CN&exportkey=AVeyDEIXF5fIEJ9zEx2YjX8%3D&pass_ticket=QUkfifECucyiVv936NsxrIyhEw4S9KLasENoEVN%2F4Ro1xFkFjyg9q88s%2FDI4p7kZ)

![图片](https://mmbiz.qpic.cn/mmbiz_gif/3xxicXNlTXLicoXI3DQHCVPD0eHuzTPichibaSwnnBEQzhV6tSIkG6wmy6S0D2NkFRnhBARHvGj9oDF2dBt60KNh6A/640?wx_fmt=gif&tp=webp&wxfrom=5&wx_lazy=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/YWib8MBGhMBuIBSUxZUzU0MGcojTgqZEugCSLVcmLqRVkuUalSFT52dIdpKnyeB9Vsw5DfI8libuN1nDzXomLSicw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

红队掌握了通达OA的REC，提醒通达OA用户注意更新！有所谓的0day，但出了也好几天，还叫0day不合适。我看到有在群里共开发布的都已经删除了。但我们公众号不合适发布Poc，可自行在群里寻找。

  

通达OA历史漏洞整理：  

通达OA 2015

**漏洞类型：SQL注射漏洞**

/inc/common.inc.php:

SQL注入general\document\models\mrecv.php:注入1:

![图片](https://mmbiz.qpic.cn/mmbiz_png/YWib8MBGhMBuIBSUxZUzU0MGcojTgqZEuB0Mh31ljlGJynbwwGTTBzicPynQv7L8uSJ6Ribibe9sXG2fNxrvxKM3Fw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

访问任意文件上传漏洞路径/ispirit/im/upload.php  
  

**漏洞：前台SQL注入**  

影响版本：2015

ispirit/retrieve_pwd.php

  

**漏洞：任意文件上传**

影响版本：2015-2017  
条件：需要任意用户登录  
general/reportshop/utils/upload.php

  

  
**漏洞：文件包含漏洞两个**  
影响版本：2015-2017  
利用条件：只能包含指定php文件  
inc/second_tabs.php  
  
利用条件：需要任意用户登录，只能包含指定php文件  
general/reportshop/utils/upload.php  
  
**漏洞：任意文件删除**  
影响版本：2015-2017  
利用条件：需要任意用户登录  
general/reportshop/utils/upload.php

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg== "微信公众号文章素材之分割线大全")

  

**![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)「华盟学园」 知识星球现已开启！** 一个学习网络安全知识和分享工具的星球，网络安全大佬在线分享技术文章，大家一起学习、共同进步！

  

如果你对我们星球内的分享的知识和工具感兴趣，可以随时加入我们的星球，安全大佬在里面等着你。

  

现在加入知识星球只需要：**365/年（1天1元，星球内所有内容免费学习获取）。**

  

加入星球：点击图片，扫描二维码，快快加入吧！

  

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

![](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)