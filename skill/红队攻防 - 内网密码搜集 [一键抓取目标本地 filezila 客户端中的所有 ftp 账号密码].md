> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/qR-L-MTXMT2JFFZHQm_uxg)

### 场景

跟之前 foxmail 一样, 假设依然是通过发信打到的一台目标单机, 然后翻机器时发现上面装有 filezila 客户端, 然后现在就想把客户端中保存的所有 ftp 账号密码都获取下, 非常简单, 过程   

如下

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM7iaiaZib7wY9LMfVGstjOXla7Xr1WZtXQxAZoSibhOTEN1NTJlOG4fMEHFptBZhnZSfxSXFR0WXdJicCg/640?wx_fmt=png)

正常抓取当前机器的已安装软件列表, 发现其存在 filezila 客户端, 那接下来的事情就很清晰了

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM7iaiaZib7wY9LMfVGstjOXla7WuSAtjKFTIJZzWoWhTs6Nab7AYbEmBOobsKk6MmM3XMFPs2Q2NSoMg/640?wx_fmt=png)

特别注意, 此处想成功抓到密码的前提是目标 filezila 客户端中必须事先保存的有密码才行, 怎么知道它保存的有没有呢? 其实, 我也不知道, 只能通过后续翻下对应的 xml 文件才知道, 不 过, 可以保证的是, 不翻翻永远都不会知道

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM7iaiaZib7wY9LMfVGstjOXla7lrr7BmKjqYbIibcb0gIbuXQyibWY8er0T51oCvYFYxjH8C0yClprugwA/640?wx_fmt=png)

保存密码的 xml 文件默认都被放在当前用户数据库目录下的 filezila 目录中

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM7iaiaZib7wY9LMfVGstjOXla7R9a7HPDzb4b4lD9Mh9ia9ZQYyh6icCbaGypfq5Klibre9QAZ2zdrh1BIg/640?wx_fmt=png)

由于主机, 端口, 账号, 密码字段都是已知的, 所以直接一把梭哈出来就好了

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM7iaiaZib7wY9LMfVGstjOXla7YUNmrAKrSibIRFfSyVibonDxOyWyImRsM1Bk16HcjXTvhwKR3ICt3FhA/640?wx_fmt=png)

由于密码只是用 bs64 编码了下, 最终, 批量解码下即可得到所有的明文账号密码

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM7iaiaZib7wY9LMfVGstjOXla7kqfib4bSoNys8iaSiasn6snTibL0ENj9kEm2Yvy3Kz0x8sDOFCGBDSMMaQ/640?wx_fmt=png)

### **小结**

注意, 此处的演示全部都是用最新版的客户端, 非常简单, 其实压根也没什么太多好说的, 遇到目标装有 filezila 客户端就去对应的目录搜下就好了, 实在没有搜不到就算了, 没啥好纠结的, 拿到这些 ftp 账号之后的事情, 还是按正常流程走就好了, 去各个 ftp 上多翻翻敏感文件资料便于后期拓展... 就先到这儿吧, 有任何问题, 欢迎弟兄们及时反馈, 非常感谢, 祝好运

本文原创作者：By Klion

公众号

最后  

-----

**由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，文章作者不为此承担任何责任。**

**无害实验室拥有对此文章的修改和解释权如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经作者允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的**