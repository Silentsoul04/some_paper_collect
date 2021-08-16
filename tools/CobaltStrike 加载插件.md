> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s?__biz=MzI2NDQyNzg1OA==&mid=2247485377&idx=1&sn=1e93b38971c0c3c4e1cdfdaa7625ec9e&chksm=eaad87fcddda0eea6469afd46a059b6ebff47e1dfa0a5d5fc8ca22722f5db61141580ea12c9f&scene=21#wechat_redirect)

 CobaltStrike 有两种加载插件的方法，一种是在客户端加载，一种是在服务端加载。在客户端加载，当客户端没连接上服务端后，该插件即不会被加载。所以有时候需要在服务端加载某些插件。

  

![图片](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFiaN9PiaczdMbQOHacDxuPdLxHjk5oKk5rB66ng1HCNUfsEpVsygyGYBTzkpDwaC3FgibAJQCZL2cNVA/640?wx_fmt=gif)

  

**客户端加载**'

点击 CobaltStrike--> 脚本管理器

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2c64ocPkSQTNElNicTicVGVU6XYY9X3icrftPDFicVYyXCEvoB8KA4KBTPc1OSUdh8uWqPXAbjmIfKuWw/640?wx_fmt=png)

然后点击 Load 加载我们的插件，插件后缀格式为 .cna

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2c64ocPkSQTNElNicTicVGVU64BB3PBu5jpW6NXlibQ1c0O6fYKqSGJcDbWpvAsh342m50iabcrUXP1LA/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFiaN9PiaczdMbQOHacDxuPdLxHjk5oKk5rB66ng1HCNUfsEpVsygyGYBTzkpDwaC3FgibAJQCZL2cNVA/640?wx_fmt=gif)

  

**服务端加载  
**

CobaltStrike 服务器端有个 agscript 文件，他是用来在服务器端运行 cna 插件文件的

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2c64ocPkSQTNElNicTicVGVU6dL3YboB1DuM1cQe18FYicUTuaeA9989hIMkibHfG0a0hrhk9GiczNHvJg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2c64ocPkSQTNElNicTicVGVU6xX2BJItQkPkiaMBib7NOIkQanVZicl8D1rYvYQWSRgo3cAsjkSZ1Kf0TQ/640?wx_fmt=png)

```
./agscript [host] [port] [user] [pass] </path/to/file.cna>
[host] #cs服务器的ip地址 
[port] #cs的端口号 
[user] #用户名，用来运行这个脚本的用户名，随便即可。
[pass] #cs的密码，就是启动cs时你设置的密码。
[path] #cna文件的路径。
```

![](https://mmbiz.qpic.cn/mmbiz_png/rSyd2cclv2c64ocPkSQTNElNicTicVGVU6JMJHxUbGiaDtskuyTZM5whjKxRgQrGhGcHWict3wCXQgkLI1uP7Jsib2w/640?wx_fmt=png)

但是我们一般会将其运行在后台

```
nohup ./agscript cs的ip cs的端口 任意用户名 密码 插件路径 &
```

  

![](https://mmbiz.qpic.cn/mmbiz_gif/UZ1NGUYLEFiaN9PiaczdMbQOHacDxuPdLxHjk5oKk5rB66ng1HCNUfsEpVsygyGYBTzkpDwaC3FgibAJQCZL2cNVA/640?wx_fmt=gif)

  

CobaltStrike 常见插件  

传送门：[Cobaltstrike 扩展插件整理](http://mp.weixin.qq.com/s?__biz=MzU4MDIzMDcyNw==&mid=2247483720&idx=1&sn=4539640699a1bcaec99d0005343e7664&chksm=fd5b4a60ca2cc3767106a4001cbf05130ab5b6fc054461ba1257501acb80add64403b28ee3d4&scene=21#wechat_redirect)

相关文章：https://www.cobaltstrike.com/aggressor-script/index.htm