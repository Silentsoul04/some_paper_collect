> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/s_0-3-NZtjdpM-kFgsu6Rg)

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV01JVJusudJOQA9ic4SKUs1FnZDlxHnrWKYibqKYaTOoNKiaPoA783baTaiaibxAhldBL0ySVzWRgPXzDg/640?wx_fmt=png)

其功能包括：

*   检查令牌的有效性
    
*   测试已知漏洞：
    

*   (CVE-2015-2951) _alg=none_ 签名绕过漏洞
    
*   （CVE-2016-10555）_RS / HS256_ 公钥不匹配漏洞
    
*   (CVE-2018-0114) _密钥注入_漏洞
    
*   (CVE-2019-20933/CVE-2020-28637) _空白密码_漏洞
    
*   (CVE-2020-28042) _空签名_漏洞
    

*   扫描错误配置或已知弱点
    
*   模糊声明值以引发意外行为
    
*   测试机密 / 密钥文件 / 公共密钥 / JWKS 密钥的有效性
    
*   通过高速_字典攻击_识别_弱键_
    
*   伪造新的令牌标头和有效载荷内容，并使用密钥或通过其他攻击方法创建新签名
    
*   时间戳篡改
    
*   RSA 和 ECDSA 密钥生成和重建（来自 JWKS 文件）
    

要求
--

        该工具是使用通用库在 Python 3（版本 3.6+）中原生编写的，但是各种加密功能（以及一般的美感 / 可读性）确实需要安装一些通用的 Python 库。

安装
--

        安装只是下载`jwt_tool.py`文件（或`git clone`repo）的一种情况。  
（`chmod`如果您想将它添加到 _$PATH_ 并从任何地方调用它，该文件也是如此。）

```
$ git clone https://github.com/ticarpi/jwt_tool
$ python3 -m pip install termcolor cprint pycryptodomex requests
```

        首次运行时，该工具将生成一个配置文件、一些实用程序文件、日志文件以及一组各种格式的公钥和私钥。

项目地址：

https://github.com/ticarpi/jwt_tool