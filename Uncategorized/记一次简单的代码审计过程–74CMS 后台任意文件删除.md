\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/IOHVanjOK0tohO2pNizy8g)

****文章源自【字节脉搏社区】- 字节脉搏实验室****

**作者 - 樱宁 i**

**扫描下方二维码进入社区：**

**![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnK3Fc7MgHHCICGGSg2l58vxaP5QwOCBcU48xz5g8pgSjGds3Oax0BfzyLkzE9Z6J4WARvaN6ic0GRQ/640?wx_fmt=png)**

**献丑了，(╥╯^╰╥) 能力有限，思路不对处还望各位斧正了啦~**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

**漏洞版本：全版本**

**下载网址：74cms 下载网址**

**漏洞描述：**

**下面的测试版本为 74cms\_Home\_Setup\_v6.0.4**

 **搭建 ---->进入后台 “工具” 选项的 “数据安全”----> 备份数据库 ---->选择删除备份文件 ---->抓包，并在文件名后面拼接../---->放包可达到格盘的效果**

**1. 安装时注意不可 php7.0 以上版本**

**2\. 进入后台，备份数据库，此时会在 data 文件夹下形成 backup 文件存储备份信息。**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnLqa2xDZmmXZedyW8OkY6P7AFjibCO3P5KzbuXqD9e0VO6Ms1YX88FSaFyOuMtuUISicVXLxbu5CSlQ/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnLqa2xDZmmXZedyW8OkY6P7svA7lWrkZKEIZGpg6oCG5WvGocHbwJ7HLKqhwicibltIVzJu8erZiaddA/640?wx_fmt=png)

**3\.  备份后跳转到漏洞网址，此时我们选择删除，并进行抓包。**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnLqa2xDZmmXZedyW8OkY6P7U8LtAcBVialSJACEGZYzF5o6CepYIV60Sg2Nu8uzia6f3SzCUYfYIUgg/640?wx_fmt=png)

**4\. 我们对备份文件名进行恶意拼接（name=20200516\_2/../），此时已经没有存储备份文件的 database 文件夹**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnLqa2xDZmmXZedyW8OkY6P7ia6HNf5hm6sEA9Hxlqr7zwOX0KiaX8cDGbpuYkvIDzc1manyial4TpMGg/640?wx_fmt=png)![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnLqa2xDZmmXZedyW8OkY6P7JibtHgSyPWqJrNVicYQiaTd7w9L0PJvaEkq6VdbdAzQkzYL4Kvowf1HZA/640?wx_fmt=png)

**5\. 我们可以继续拼接，发现可以格盘。**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

**漏洞分析：**

**审查元素，通过删除，找到对应的控制器**

**/index.php?=admin&c=database&a=del&name=20200516\_1**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnLqa2xDZmmXZedyW8OkY6P7ZtubaB4nV6ZJOh1rqG5hUlxUY0xv7c5IHpsEdWqJibfGnZB8vViacwIg/640?wx_fmt=png)

**在**

**application->admin->Controller->DatabaseController.class.php 中我们找到对应的删除代码。发现是将传来的文件名放到数组中，然后在遍历，拼接路径之后采用 rmdirs（）方式进行删除。**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnLqa2xDZmmXZedyW8OkY6P7siaP5Q3CWuz9XJpINbMIYtDM5A2TX2kllxibSiaxBWoDxCrYiaIvoFzX8A/640?wx_fmt=png)

**我们继续追踪一下 rmdirs（），发现此处就是简单的递归删除。没有做任何的防护。**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnLqa2xDZmmXZedyW8OkY6P7hOTB2AVA0NpeQ7mKfX4oUGLia9qyIp4K52PMoe1KC2INvDbSDplcSxw/640?wx_fmt=png)

**上述的两段代码结合可以发现，每次我们传入参数为**

**name=20200516\_2/../，拼接 DATABASE\_BACKUP\_PATH. $val 的结果为./data/backup/database/20200516\_3/../，接着在 rmdirs 接受该参数并进行递归删除。此时由于递归删除存在../ 就会导致删除上一级**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

**漏洞修复：**

**传入 rmdirs（）之前，首先用 basename（）进行验证传入的参数是否包含../ 等敏感字符，如果没有则进行下一步的删除，如果存在则进行报错。**

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

![](https://mmbiz.qpic.cn/mmbiz_png/ia3Is12pQKnLqa2xDZmmXZedyW8OkY6P74SgPKrMfa4sz20PvCnAWjnh2ibibZLhHO3gdcmejtYSKD4oMuvibXGicAg/640?wx_fmt=png)

**呀呀呀，到这就看完了哦~** 

****![](https://mmbiz.qpic.cn/mmbiz_png/Ljib4So7yuWgiazacZwcozhIIJkbibEWTcfRmJfpFw8RCkn9iaZOyT4YJ5JCqCIvRvCLC5RznuKbdPrlfXuXPkevEQ/640?wx_fmt=png)****

**通知！**

**公众号招募文章投稿小伙伴啦！只要你有技术有想法要分享给更多的朋友，就可以参与到我们的投稿计划当中哦~ 感兴趣的朋友公众号首页菜单栏点击【商务合作 - 我要投稿】即可。期待大家的参与~**

**![](https://mmbiz.qpic.cn/mmbiz_jpg/ia3Is12pQKnKRau1qLYtgUZw8e6ENhD9UWdh6lUJoISP3XJ6tiaibXMsibwDn9tac07e0g9X5Q6xEuNUcSqmZtNOYQ/640?wx_fmt=jpeg)**

**记得扫码**

**关注我们**