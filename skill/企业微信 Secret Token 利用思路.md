> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/LMZVcZk7_1r_kOKRau5tAg)

在一次渗透过程中，获得了一台服务器的主机权限，但是提权未果。不得已只能翻一下主机上面的文件，看一下有没有什么可以利用的地方。在其中一个网站源码里面，翻到了这么一段代码

```
<add  />
    <add  />
        <add  />
```

百度了一下，发现是企业微信的开发密钥

![](https://mmbiz.qpic.cn/mmbiz_png/noZJ3Kqbu1cIZOue50lVXiaiblGvSQPNGWdGTOcTOFPibOwyiaW1zvrQ3bpovqEHkemYHMnDV4rbLn7EOiatS0jjcAw/640?wx_fmt=png)

阿里云的密钥大家都知道，拿到了密钥基本上等于拿到了阿里云账号的控制权限，直接可以执行命令的。

企业微信的密钥，还是接触比较少，翻了一下企微的开发 API，发现可以通过这个 ak 添加企微用户。

第一步：获取 AccessToken
------------------

API 地址：`https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=id&corpsecret=secrect`

传入获取到的 id 和 secrect 即可获取 AccessToken

![](https://mmbiz.qpic.cn/mmbiz_png/noZJ3Kqbu1cIZOue50lVXiaiblGvSQPNGWpBNzMkeJc9WzK8dbsmrcZWkjEWyO7yRPatdBOk6SuoJBvhKLjr3Trw/640?wx_fmt=png)

第二步：获取部门 ID
-----------

企微中会分为一个个的部分，通过企微的 API 我们可以获取到企业的架构和部门 ID，这个在添加成员的时候用的到。

在`https://open.work.weixin.qq.com/devtool/query?e=301002`中查询上一部获取到的 ak 权限就能，查询到部门名称，以及部门 ID

![](https://mmbiz.qpic.cn/mmbiz_png/noZJ3Kqbu1cIZOue50lVXiaiblGvSQPNGW21xYRtvb970TjWuJ8oEtdzOMbwtzmILdr4PaiasZS3V7AIIqqvp7cBg/640?wx_fmt=png)

第三步：添加企微成员
----------

![](https://mmbiz.qpic.cn/mmbiz_png/noZJ3Kqbu1cIZOue50lVXiaiblGvSQPNGWLXh7qX4tOwib0UyzrKmyTs2w2UBEqCbQHxUm1icibdRbUoIJQbWJroLdg/640?wx_fmt=png)

往`https://qyapi.weixin.qq.com/cgi-bin/user/create?access_token=ACCESS_TOKEN`

```
{
   "userid": "zhangsan",
   "name": "张三",
   "department": [6],
   "mobile":"1388888888"
}
```

调用成功后，即可通过手机号登录目标企业的企业微信。可选择企业微信通讯录中选择高管、信息部门等姓名进行伪装。然后再采取社工钓鱼的方式，信任度和成功率都会高很多。

第四部：善后
------

当使用手机号登录成功以后，可通过修改成员 API 来修改姓名进行伪装，并且可以将手机号修改为空来防溯源。

![](https://mmbiz.qpic.cn/mmbiz_png/noZJ3Kqbu1cIZOue50lVXiaiblGvSQPNGWxQKNDiaHVL2EGlPiaLUMKvsoB8TTLq8du9ic4hb6FSBcjyXZicVsyVC62Q/640?wx_fmt=png)

最后，删除成员以绝后患

![](https://mmbiz.qpic.cn/mmbiz_png/noZJ3Kqbu1cIZOue50lVXiaiblGvSQPNGWicD8SupSxYPJVCzfvjHS0KKVw49R6VyjiaQXeP79Bb7jxCkdyn38t1OQ/640?wx_fmt=png)

> 企业微信 API 文档地址
> 
> https://qydev.weixin.qq.com/wiki/index.php?title=%E7%AE%A1%E7%90%86%E6%88%90%E5%91%98#.E6.9B.B4.E6.96.B0.E6.88.90.E5.91.98