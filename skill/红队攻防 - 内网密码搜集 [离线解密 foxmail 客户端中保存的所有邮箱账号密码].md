> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Acx2OaPMER9-dmrzyt6xxw)

### 0x00 场景

假设我们此处通过发信钓到了一台目标单机, 在翻这台单机的时候, 发现了它装的有 foxmail 客户端, 然后在 forxmail 安装目录下还发现存在数据目录 [ 即 Storage 目录, 一般情 况下, 只有在有邮箱连接记录保存过账号密码时才会自动创建该目录, 当然啦, 这个密码肯定不会直接明文保存在本地 ], 也就说, 本地很可能保存的有邮箱账号密码, 现在的想法就是想 去解密对应邮箱目录下保存有密码的那个文件, 该怎么搞呢? 其实, 简单...

### 0x01 具体过程

一般情况下, 当我们钓到目标的某台单机之后, 都会选择一个固定的操作目录来进行后续的动作, 比如, 当前用户 [此处为未 bypassUAC 的管理权限] 的临时目录, 就是个还不错的选择, 因为这里几乎不存在什么不能读写的问题, 后续不管是传工具, 文件, 做维持... 啥的都很方便, 而且很规整, 后期便于集中清理, 发现很多弟兄在操作的机器多了之后都喜欢乱传一气, 导 致最后自己都不知道在哪台机器上还留的有东西忘了清, 这显然不是个什么好习惯, 所以... 多注意下就好, 反溯源很大一部分, 都是基于你平时点滴的操作习惯来得, 操作规整, 思维缜密, 水平一般的管理员其实很难发现, 而不是完全依赖你所用的技术到底有多高端

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM7iaiaZib7wY9LMfVGstjOXla7ibOyYj84H4E8Dog8MMGOesohw7UYb40iajmML03YpSrwP5O7iaIvXpQJQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM7iaiaZib7wY9LMfVGstjOXla7CShC6LZMOgGyQGt0v5riaFHRF3jk3Igj7icRjT4QCe9hNtIhlUWF2dyg/640?wx_fmt=png)

通过读取当前机器已安装的软件列表, 发现其存在 foxmail 客户端, 版本为 7.2.x, 特别注意下这个版本, 因为不同的版本, 账号密码保存的位置和文件名都是有所不同的, 比如, 此处 为 7.2.x 版本, 账号密码默认就保存在 Account 目录下的 Account.rec0 文件中, 7.x 版本貌似是保存在一个叫 Accounts.tdat 的文件中, 而 6.x 版本则是保存在一个叫 Account.stg 的文件中

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM7iaiaZib7wY9LMfVGstjOXla7g2zBafV2hRCGanNliavmoHabaQfnSLSaIDmfIG9XfJYjQQ572t7T90Q/640?wx_fmt=png)

紧接着, 在 Foxmail 安装目录下发现其存在 Storage 目录, 并且在该目录下还发现两个邮箱的连接记录

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM7iaiaZib7wY9LMfVGstjOXla7RuElwT3RsUAE3XOP874ibJcS8pPKxbBJibV00LqqRtRO67fsfgDrrU6Q/640?wx_fmt=png)

接着, 要做的事情就非常简单了, 只需要把对应邮箱目录下的 Account.rec0 文件想办法拖回本地

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM7iaiaZib7wY9LMfVGstjOXla740hLL452SiaIa3UEicjQIt2tRuVykMQHY4ib2BcREWJ7rTibyxd0342GlA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM7iaiaZib7wY9LMfVGstjOXla74arSibQC9wlfiamaY99ib9icyqLvKrXhQ6SjxjXEayI9Pcg1Lsbe7RMz2Q/640?wx_fmt=png)

然后在本地用 securityxploded [弟兄们应该都很清楚, 这是一个神奇的网站😊] 提供好的解密工具把文件拖进去解密即可, 最终, 分别得到两个邮箱的明文账号密码如下

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM7iaiaZib7wY9LMfVGstjOXla77AfLm0YupoLVIcR91gEBavibgs8DNMJjy7LgsXxXB42UianEscK3RpZA/640?wx_fmt=png)

### 0x02 小结

注意, 此处的所有操作都是在非管理员权限 [未 bypassUAC] 下进行的, 至于拿到这个邮箱账号密码之后的价值和用途, 想必就不用再多说了吧, 因为目标邮箱的历史邮件里可能保存有 大量的敏感资料信息, 所以在内网渗透中翻邮件应该成为你的日常操作, 退一步来讲, 如果当前是在域环境中, 邮箱账号密码很可能也是对应的域用户密码, 另外, 这个邮箱账号密码和 oa 系统有也可能是通用的, 所以, 你都懂了... 此处没涉及到什么具体原理, 比较简单, 偏实用为主, 所以就没去细扣那些, 有任何问题, 欢迎弟兄们及时反馈, 非常感谢, 祝好运 

本文原创作者：By Klion

公众号

最后  

-----

**由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，文章作者不为此承担任何责任。**

**无害实验室拥有对此文章的修改和解释权如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经作者允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的**