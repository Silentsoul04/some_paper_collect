> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/hqlHl7G11UKokGKzXWJ4Fw)

对象存储 OSS
--------

阿里云对象存储 OSS（Object Storage Service）是一款海量、安全、低成本、高可靠的云存储服务，提供 99.9999999999%(12 个 9) 的数据持久性，99.995% 的数据可用性。多种存储类型供选择，全面优化存储成本。

劫持利用
----

访问某域名，提示 NoSuchBucket

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM4Eb85pWpqLQpTtUO00ZqcHHd1Sm0iaCXsCvPibG4Nh9SGEydVX4gkdw3CgZKZHmd53HyiaicUldRQgTg/640?wx_fmt=png)

获取信息，这个桶不存在

HostId : `baobao-tb.oss-cn-shenzhen.aliyuncs.com`

BucketName : `baobao-tb`

登陆阿里云，访问 (OSS 管理控制台)[https://oss.console.aliyun.com/overview]

### 创建 Bucket

注意填写 Bucket 名称和地域，和上面获取的信息保持一致

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM4Eb85pWpqLQpTtUO00ZqcHvpib7cGwDRjicTTIzaibqefpDEkTz7N655xsJ0w4sDKX0MGYSlpcqOcvw/640?wx_fmt=png)

查看 Bucket 域名，可以看到创建成功

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM4Eb85pWpqLQpTtUO00ZqcH7N8xOvEH0UbiaZyRwGAdl2VASVwcl7vsdDzeF7x2IngSFE2xJ4QnNAQ/640?wx_fmt=png)

### 上传文件

访问左侧菜单，选择上传文件，上传 HTML 文件，文件 ACL 选择公共读就可以了

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM4Eb85pWpqLQpTtUO00ZqcH0Bnic0kN3XVQQic2wbuqUKJb8N6m39FAODyicgvg7GeFEy1Z5ShNhKuow/640?wx_fmt=png)

访问该该地址，发现已经劫持成功

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM4Eb85pWpqLQpTtUO00ZqcHW7O8X12uTUBP9XyomrUyNdBcg0XElUicquBjibexHEDOuPcfUq9QjiciaA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM4Eb85pWpqLQpTtUO00ZqcH8PP3W3Fej5PDwnlEThteiaq9NuAdT9SoBhGPMWibzhumDfpEPl4SYppQ/640?wx_fmt=png)

fofa 全网检索
---------

精准检索：

```
body="NoSuchBucket" && body="BucketName" && body="aliyuncs.com"
```

粗略检索

```
body="NoSuchBucket" && body="BucketName"
```

![](https://mmbiz.qpic.cn/mmbiz_png/ewSxvszRhM4Eb85pWpqLQpTtUO00ZqcHP06En8xFKqPxY6ve5qcKE2iaEVOsvDbiariaV7iaR5A3emzAAUAtdCEvuQ/640?wx_fmt=png)

公众号

最后  

-----

**由于传播、利用此文所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，文章作者不为此承担任何责任。**

**无害实验室拥有对此文章的修改和解释权如欲转载或传播此文章，必须保证此文章的完整性，包括版权声明等全部内容。未经作者允许，不得任意修改或者增减此文章内容，不得以任何方式将其用于商业目的**