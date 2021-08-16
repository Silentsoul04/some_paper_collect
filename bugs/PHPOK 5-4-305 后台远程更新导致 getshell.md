> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/XorPHW5owgpEJh0OHkti8w)

****文章源自【字节脉搏社区】- 字节脉搏实验室****

**作者 - 樱宁**

**扫描下方二维码进入社区：**

**![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnK3Fc7MgHHCICGGSg2l58vxaP5QwOCBcU48xz5g8pgSjGds3Oax0BfzyLkzE9Z6J4WARvaN6ic0GRQ/640?wx_fmt=png)**

**漏洞名称：**

**PHPOK 5.4.305 后台远程更新导致 getshell**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

****漏洞版本号：****

**PHPOK 5.4.305**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

**官方网站：**

**https://www.phpok.com/download-center.html**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

**漏洞危害：**

**严重**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

**漏洞描述：**

**全新的 PHPOK 出来了，支持手机版，小程序，PC 版。  
**

**后台采用 Layui 框架，前台使用 AmazeUI 框架（PC 版及手机版）**

**独有的开发模式及运行模式，功能不减，一如既往的在完善！**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

**漏洞 URL：**

**http://127.0.0.1:8081/admin.php?c=update&f=ma****in**  

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

**漏洞描述：**

 **搭建 ----> 进入后台，修改远程 url----> 上传恶意远程数据包 ----> 上传解压 ---->getshell**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

**1. 进入后台，选择程序升级，环境配置中的 url 改成我们的服务器 url。**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnLcf38ZYOULCn31hyWheA2AZSeyWlcibt4NHO6ITicLEeaCxA3MCXnhr7YQVypBIuWSfxX0sZF6RGEw/640?wx_fmt=png)

**2. 服务器上放上我们的内容被修改过的更新压缩包（在后面漏洞分析处有具体分析，以及源代码）。**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnLcf38ZYOULCn31hyWheA2AuHG7s1uxCfVr4k06dA1XKwFDwl6O2LAblkXaz1x3vymZ4a2yOian6ag/640?wx_fmt=png)

**3. 更新**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnLcf38ZYOULCn31hyWheA2AEicotluVopAo7su7iaP6ay9wFFtgmG0hf7tsh2sMoCP9kGIyuzFtjkQw/640?wx_fmt=png)

**4、查看发现已经创建了 111.php**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnLcf38ZYOULCn31hyWheA2A4Giby23lovVSMMMMnmlazWWh5fsRibAUyJ1xZWxKWicU4nFn1mS4AyBSA/640?wx_fmt=png)

**漏洞分析：**

**l 审查元素，找到程序升级对应的 php 代码。（framework-->admin-->update_control.php）** 

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnLcf38ZYOULCn31hyWheA2AGojTFJCuhWHjibkpJWK5iagn7SHyKQ7ojDWtU91NKM0y2CicOJm5domjA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnLcf38ZYOULCn31hyWheA2AnicFdbNlyYsM1XQdRpl6XPdgR7n52rU3R0705lZiaXe6SbsCXDokYR9w/640?wx_fmt=png)

**分析上述代码，$info = $this->service(4) 比较突出，我们追溯一下这个调用。Service 函数首先是判断是否能正常升级，然后拼接一个 xml 的网址（一直到 520 行才拼接完成），然后是读取 url 的内容，并返回。**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnLcf38ZYOULCn31hyWheA2ALB9icXHeGsRJs0tWAgBcPcohv8h7xicrlwzFMz8Bvh06dfBzZ0dxqNbg/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnLcf38ZYOULCn31hyWheA2Ap96A6hc11MCM7Q1Dd51u8gBzibm6UicT78QBK8VWukrNVsEt8RAWS2WA/640?wx_fmt=png)

**我们在 520 行后面打印一下拼接出来的 url（var_dump($url);die;），并尝试访问**

**(http://update.phpok.com/5/index.php?version=5.4.305&time=1589767905&type=4&domain=127.0.0.1&ip=127.0.0.1&onlyid=06af7c21f6d41d613ee8fabfbe5d9092&phpversion=5.6.9&server=Windows%20NT&soft=Apache%2F2.4.39%20%28Win64%29%20OpenSSL%2F1.1.1b%20mod_fcgid%2F2.3.9a%20mod_log_rotate%2F1.02&mysql=8.0.12)，得到一个 xml 页面，如下图所示：**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnLcf38ZYOULCn31hyWheA2A1xn67QJtU2okTZaKrOYSLmLGVHmoq7C5XyxdB0vPjdVgWicOf7aTGFg/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnLcf38ZYOULCn31hyWheA2AWOScgK66JvKdBmtLq547dJCSl5ByB2PwxATrribpS5odYkicjoHsZjRQ/640?wx_fmt=png)

**们继续接着在线升级代码分析，当 service 函数接收参数为 4 时，返回了跟新版本信息（为 xml 格式），我们继续在 104 行后面打印 $info, 如下图。打印了一些数据包参数。**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnLcf38ZYOULCn31hyWheA2Ap6nA3J1TeKXBUwVQFXC31icqI1FicI5mtHicWRniacibJicj56dD8SF2dOibw/640?wx_fmt=png)

**l 放行之后页面跳转，此时页面显示的是数据包的版本等信息。接着我们在开始更新处进行抓包，信息如下。**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnLcf38ZYOULCn31hyWheA2AHHBB0niaf5pS5ichxpOAGZrCYIia7yTOx8tQl8A867oDwcH4j4CtZqAdA/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnLcf38ZYOULCn31hyWheA2AoBATdibSibJq7ycYIXnsXXWtVeKOA0VR6ac1skz3WYlTK1KgwFKr0WQQ/640?wx_fmt=png)

**找到对应的代码（framework-->admin-->update_control.php），同样调用了 service 函数。我们继续上述的操作，打印此时的 url**

**http://update.phpok.com/5/index.php?version=5.4.305&time=1589769924&type=5&file=54325**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnLcf38ZYOULCn31hyWheA2AFb5KuSK1VonEBP5vXWQP3hdRs4skKQ36pDiagicgDwIFkcs2uc5lMhEQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnLcf38ZYOULCn31hyWheA2AjuzHGINrmVecOWGXcDmeEuciawUbA9k35cppibu55KNR8Z46niauytlTA/640?wx_fmt=png)

**访问上述网址，出现被 base64 加密的 xml 页面**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnLcf38ZYOULCn31hyWheA2AdgdYSl0O8eIPy2jxnicUGMHXbXp0u5PputJ5zE16ssoPJibMiagntV7JA/640?wx_fmt=png)

**我们回到文件升级处的函数，当传输参数 5 到 service 函数，得到 xml 页面信息，接着在 390 行**

**（file_put_contents($this->dir_data.'tmp.zip',$info);）将信息写到了 tmp.zip ，我们打印一下压缩包的存放路径（在 390 行后面写上 var_dump($this->dir_data.'tmp.zip');die;），此时文件夹下面生成了文件夹。**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnLcf38ZYOULCn31hyWheA2AMHp8WhZe2sibQ36vacYLIAljdcDicZfCibGpXro6WqkgOqtD0a3V10mNg/640?wx_fmt=png)

**此时我们对于远程更新的基本流程都大致清楚。，此时我们将手中的包丢掉，不让其更新。继续分析它的解压等后续代码，我们追溯到解压函数（update_load），会发现会先调用 run.php**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnLcf38ZYOULCn31hyWheA2AuQ92XKeThKYicnXuuI53NZhzWcBlCQQeRq2WQD8oEo1SsKCgLticXBWg/640?wx_fmt=png)

**此时我们就恶意整合一下 getshell 的操作：**

**1、 远程 url 的 index.php 中由 type 参数生成不同的 xml 页面信息（4—> 版本信息等；5—> 数据包内容）**

**注：版本信息直接复制 xml 页面源代码。**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnLcf38ZYOULCn31hyWheA2AWiafFcFzqMgDuAL0kU6oaLRL52l3Wbab3WaLAgUjs3US5iaBDZRZJjtA/640?wx_fmt=png)

**2、 当参数等于 5 时 xml 中的数据包内容为我们在 run.php 中添加了木马的 tmp.zip 信息。**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnLcf38ZYOULCn31hyWheA2A4HZ0QmXHCsMR8wOseFXnmicKH9gJngs3jXDYLdcx8NAf0AoiarqYLicZQ/640?wx_fmt=png)  

**漏洞修复：**

**接受远程 url 是进行白名单限制**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

**通知！**

**公众号招募文章投稿小伙伴啦！只要你有技术有想法要分享给更多的朋友，就可以参与到我们的投稿计划当中哦~ 感兴趣的朋友公众号首页菜单栏点击【商务合作 - 我要投稿】即可。期待大家的参与~**

**![](https://mmbiz.qpic.cn/mmbiz_jpg/ia3Is12pQKnKRau1qLYtgUZw8e6ENhD9UWdh6lUJoISP3XJ6tiaibXMsibwDn9tac07e0g9X5Q6xEuNUcSqmZtNOYQ/640?wx_fmt=jpeg)**

**记得扫码**

**关注我们**