> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.bylibrary.cn](https://www.bylibrary.cn/%E6%BC%8F%E6%B4%9E%E5%BA%93/01-CMS%E6%BC%8F%E6%B4%9E/DedeCMS/Dedecms%E4%BB%BB%E6%84%8F%E7%94%A8%E6%88%B7%E7%99%BB%E5%BD%95SSV-97087/)

> 白阁文库是白泽 Sec 团队维护的一个漏洞 POC 和 EXP 披露以及漏洞复现的开源项目，欢迎各位白帽子访问白阁文库并提出宝贵建议。

[](https://github.com/BaizeSec/bylibrary/blob/main/docs/%E6%BC%8F%E6%B4%9E%E5%BA%93/01-CMS%E6%BC%8F%E6%B4%9E/DedeCMS/Dedecms%E4%BB%BB%E6%84%8F%E7%94%A8%E6%88%B7%E7%99%BB%E5%BD%95SSV-97087.md "编辑此页")

#### 影响版本 [¶](#_1 "Permanent link")

dedecmsV5.7 SP2

#### 漏洞成因 [¶](#_2 "Permanent link")

dedecms 的会员模块的身份认证使用的是客户端 session，在 Cookie 中写入用户 ID 并且附上 ID__ckMd5，用做签名。主页存在逻辑漏洞，导致可以返回指定 uid 的 ID 的 Md5 散列值。原理上可以伪造任意用户登录。

#### 复现 [¶](#_3 "Permanent link")

现在我们的思路就是 先从 `member/index.php` 中获取伪造的 DedeUserID 和它对于的 md5 使用它登录 访问 member/index.php?uid=0000001 并抓包 (注意 cookie 中 last_vid 值应该为空)。 ![](https://www.bylibrary.cn/%E6%BC%8F%E6%B4%9E%E5%BA%93/01-CMS%E6%BC%8F%E6%B4%9E/DedeCMS/Dedecms%E4%BB%BB%E6%84%8F%E7%94%A8%E6%88%B7%E7%99%BB%E5%BD%95SSV-97087/f09e67a7e30cf8167f0e1f0e01ae01d9.png) 可以看到已经获取到了，拿去当做 DeDeUserID![](https://www.bylibrary.cn/%E6%BC%8F%E6%B4%9E%E5%BA%93/01-CMS%E6%BC%8F%E6%B4%9E/DedeCMS/Dedecms%E4%BB%BB%E6%84%8F%E7%94%A8%E6%88%B7%E7%99%BB%E5%BD%95SSV-97087/a599b95d431c365e3edc7ba540b363a1.png) 可以看到，登陆了 admin 用户。

#### 修复意见 [¶](#_4 "Permanent link")

M_ID 被 intval 后还要判断是否与未 intval 之前相同。

* * *

最后更新: 2021-03-24