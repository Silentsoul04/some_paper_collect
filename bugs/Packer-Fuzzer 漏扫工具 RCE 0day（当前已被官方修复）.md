> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/asgZGJLlSO_TLxGiyTPGGQ)

### Packer-Fuzzer 一款针对 Webpack 等前端打包工具所构造的网站进行快速、高效安全检测的扫描工具

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2dUfVbCmogBWOPINaBvY4Y0CGzE2Jr7PKcxvCnh3hqNtResfSQpfZPU375fFt91dTNEOjpQytdIw/640?wx_fmt=png)

### **0x01 定位 js 执行点**

        简单搜索 execjs 就可以找到执行点，位置在 Recoversplit.py 的 57 行

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2dUfVbCmogBWOPINaBvY4YcrYb4aDRpLRjNicURqVSHpI8iaaMgTtABvVSS307A1lYxFg1ic5lmzaFg/640?wx_fmt=png)

### **0x02 防护绕过**

        作者在写这个代码的时候也意识到了被反打的可能性（“防止黑吃黑被命令执行”），所以当 js 出现了 exec 和 spawn 的时候就不会执行，但是这样的简单的防护几乎是没有用的。

且不说 nodejs 沙箱逃逸已经被师傅们玩出花来了，单是这里 eval 没有过滤掉就可以通过字符串拼接或者 url 编码的方式绕过这个限制

比如说这里就用了 url 编码了能弹出计算器的 payload，解码并 eval 之后就能弹出计算器

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2dUfVbCmogBWOPINaBvY4YJ837hp4lphicqp5Izxdj4VWy3qDMeXVqFap2KxQtUweJ3AeaWAiaTCDA/640?wx_fmt=png)

### **0x03 调用分析**

        那么这个执行点是怎么被调用到的呢

看了下代码，发现执行点在 RecoverSplit 类的 jsCodeCompile 里，这个函数被同一个类的 checkCodeSpilting 调用，而 checkCodeSpilting 又被这个类的 recoverStart 调用，recoverStart 被 Project 类的 parseStart 调用，而 parseStart, 而再往前追溯就是命令参数处理之类乱七八糟的地方了

知道了这些东西之后，我们就可以根据它的调用一步一步走，一点一点写出 RCE 的 POC 了

### **0x04 如何进入 recoverStart 函数？让程序相信这个网站使用了 webpack**

我们看到这个 parseStart 长这样

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2dUfVbCmogBWOPINaBvY4YNKGYlvnUHMVTz1XD0Jn8nib9O0e5qNaz74SWy1PCnh2sDq3jeM10icWg/640?wx_fmt=png)

发现如果想要让程序调用 recoverStart，就需要让前面的 checkStart 返回 1 或者 777，这里的 checkStart 的意思是检查网站是否真的用了 webpack 技术，如果真的用了才继续扫描下去

checkStart 调用了 checkHTML，如果能让 checkHTML 返回 1 就可以让 checkStart 返回 1

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2dUfVbCmogBWOPINaBvY4YKVyQAYrBiaJ6uDTwHFlpyfmefuiaxYicltctLAPsV5ZtdP8icYibDibmWMAA/640?wx_fmt=png)

跟过去看一下  

发现如果返回的 html 包含了 fingerprint_html 的某一项就返回 1

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2dUfVbCmogBWOPINaBvY4YYJiavibcXbMiaHG9M6icC2CLLt2a8jHJ67ibGtmKibq5YnonWWC8HYJ0HTCQ/640?wx_fmt=png)

在 fingerprint_html 中看到了这些片段，可以知道，只需要在 html 里面包含其中的任意一个片段就可以让扫描器相信网站使用了 webpack 技术

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2dUfVbCmogBWOPINaBvY4YUB1x8ibgj7yDxAliaLR9miaOys8Go8Xq1V02gvfskibKtLXwWORYraFxMg/640?wx_fmt=png)

因此在 POC 的 html 中，我加入了 <noscript>naive!</noscript > 来骗过扫描器

### **0x04 如何进入 checkCodeSpilting 函数？**

recoverStart 用于处理 js（而不是 html 了）

        这里并没有加入什么恼人的判断，那个 if 也只是判断文件后缀名不为 db。推测扫描器是先把 js 下到本地然后再读取的，因此在构造 POC 的时候，通过 script src 引入一个 js，就可以让它进入到这个函数

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2dUfVbCmogBWOPINaBvY4YnffeCDmU8XiaaSKz5rB8WOMocZXuZr05L48wy2ibuw2mUSo6mqIRujMA/640?wx_fmt=png)

### **0x04 如何进入 jsCodeCompile 函数并把我们想要的东西传入？**

    checkCodeSpilting 会读取文件，判断是否包含了 document.createElement("script"); 这个字符串（以检查是否有异步加载的 js 代码），如果是的话再做一个正则匹配，然后把值加一个前缀一个后缀之后传入 jsCodeCompile 函数

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2dUfVbCmogBWOPINaBvY4YFqsQvDHSjkAfJibknQzleicq330nfbXDA4xBYTPT6NAxN1946I2IPic7A/640?wx_fmt=png)

想要成功调用 jsCodeCompile 函数，在 js 中就得加入 document.createElement("script");，而想要传我们想传的参数进去就得研究这个正则了。

        我比较菜，对正则一直心怀恐惧，但是好在这个正则也不难懂，首先匹配一个字母或者数字或者下划线（\w），在匹配一个点和一个字母 p 以及加号（\.p+），接着就是匹配到的内容，正则会不断匹配知道遇到一个. 和一个字母 j 以及字母 s（\.js）

因此实际上就是 这个正则就是匹配如下内容

【随便一个字母数字下划线】p【我们想让他匹配的内容】.js  
匹配完了之后，前面加个 "，后面加个. js，变成 jsCode 传入 jsCodeCompile

### **0x04 如何在 jsCodeCompile 函数中实现 RCE？**

            这个函数是最后也是最重要的函数，它相对复杂，因此需要分几个步骤分析

首先看到这个 js 执行的地方，它进行了 “防止黑吃黑命令执行” 的检查之后，首先会去编译这个 js，接着从 nameList 里面取东西然后传入到 js 中的 js_compile 函数里。

因此我们要做的就是两件事情，一个是让 jsCodeFunc 里面变成我们的 RCE 代码，第二个是让 nameList 里面有东西可以传进去

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2dUfVbCmogBWOPINaBvY4Y8icj0q8njeFAfwXnXsdmmtk6VCMA13baBOnuicnf6qD3mq4EKFF8jeyA/640?wx_fmt=png)

首先我们来出了让 jsCodeFunc 有 RCE 代码的问题

jsCodeFunc 的生成过程如下

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2dUfVbCmogBWOPINaBvY4YSkU1lIupMzAPWjz8p2fjybuI3bO9GbsyHsD4icOEsnGTxOyfokCJIOA/640?wx_fmt=png)

        首先从 jsCode 中正则匹配出被 [] 包裹着的第一个内容，作为 js_compile 函数的参数，然后 jsCode 本身再被插入进去赋值给作为 js_url，看起来工具的作者是希望能够动态解析 js 以获取 url 地址

因此为了让里面能够接收一个参数，需要直接在 jsCode 里面加入一个 [s]，匹配到之后就会让 variable 为字母 s，这样前面的部分就是 js_compile(s)，解决了前面传入参数的问题

        接着就要处理如何加入了 jsCode 之后能执行恶意代码的问题了。

刚才的分析结果表明传入的内容前面被加了个 " 后面被加了个. js，而且我们还要在传入内容加一个 [s]，并且加完了这一堆东西之后还不能有问题。

        虽然看起来条件苛刻，实际上处理起来也不复杂，针对前面的 "，我们再加一个" 然后来个; 来结束语句即可。接着加入 RCE 语句，最后为了对付后面的. js 和 [s]，直接用 // 注释掉

        由于后面拼接的 return js_url} 前面有个 \ n，因此注释对其不起作用，因此我们不需要自己 return 一个值然后用大括号闭合

接着我们来处理如何让 nameList 有值的问题

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2dUfVbCmogBWOPINaBvY4Y4MXZnkeq3g0cXNRXq8L3TmO5ucl9JQXKsScSqqSv706djs6FkTkXwg/640?wx_fmt=png)

        发现 nameList 就是匹配了两个正则表达式之后加进来的，因此随便加一个能让某个表达式匹配到内容的字符串就可以了，这里我加的是 {114514:，会让第一个正则表达式匹配到 114514，并且加入到 nameList 中

我们把 {114514: 加入到刚才写的语句的注释中就可以了

以上就是 payload 的完整生成过程

### 0x05 RCE 展示 & POC

![](https://mmbiz.qpic.cn/mmbiz_png/aPmkR80bcV2dUfVbCmogBWOPINaBvY4YmTVChbyVeZCFWiaFgCbaRXFl30tgwDibpnH7pGccicMibWtOiaWTwgtzFbg/640?wx_fmt=png)

地址：

https://github.com/TomAPU/poc_and_exp/tree/master/Packer-Fuzzer-RCE

文献：

https://drivertom.blogspot.com/2021/01/packer-fuzzerrce-0day.html?m=1