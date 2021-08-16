## 深入探究浏览器编码及XSS Bypass

原创： 辞令WhITECat [WhITECat安全小组](javascript:void(0);) *昨天*

通过浏览器编码深入探究XSS绕过

## **0x00背景**

​    本篇文章得自于**kaiputenku**大佬在内部分享中用生命表演的结晶，三个小时，滴水未进，晚饭都不吃了，也要给内部团队的我们这些菜鸟们把浏览器编码解码规则讲清楚，然后还要苦口婆心的教我们怎么应用到实际的XSS Bypass中，话不多说，刚给大佬tian完，下面把大佬的精华也分享给大家，希望有所收获！

  编辑完发现内容实在太多，满满都是干货，其中通过多个实例对浏览器编码解码过程进行讲解，需要各位看官花费充分时间消化吸收，建议收藏留存，有空了再拿出来瞅两眼。

## **0x01基本概念**

**为什么要进行编码？**

主要是因为某些数据不适合传输。原因多种多样，如Size过大，包含隐私数据，另外重要的一点就是有些字符会**引起歧意。**

 

**对于URL：**

&用于分割多个参数，倘若有某个参数键值为name=v&lue，就会因为name参数的值v&lue中携带了&而造成歧义。因此需要对&进行URL编码。

 

**对于HTML：**

当浏览器遇到<会识别为元素定义的开始，>会识别为元素的结束。倘若有<divid="1>" ></div>，由于标签的属性值携带了>，同样会造成歧义。因此需要属性值的>需要进行HTML编码，即使用字符实体。

 

### Ø HTML编码（字符实体）

字符实体是一个预先定义好的转义序列。

字符实体两种表示方法：

1、字符实体以&开头+预先定义的实体名称+;分号结束，如“<”的实体名称为&lt;

2、字符实体还可以以&开头+#符号+字符在ASCII对应的十进制数字+;分号结束，如<的实体编号为&#60;

**字符都是有实体编号的，但有些字符是没有实体名称
**

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)





 

### Ø JavaScript编码

最常用的，如\uXXXX这种写法的Unicode转义序列，表示一个字符，其中XXXX表示一个16进制数字，如<的Unicode编码为\u003c。

 

### Ø URL编码

RFC3986文档规定，URL中只允许包含英文字母（a-zA-Z）、数字（0-9）、-_.~4个特殊字符以及所有保留字符。

RFC3986中指定了以下字符为保留字符：! * ' ( ) ; : @ & = + $ , / ? # [ ]

 

**编码方式**

%加字符在ASCII码表中的十六进制值。

例如，/在ASCII码表中十六进制为0x2f，那么它对应的URL编码为%2f。

 

参考链接：https://www.cnblogs.com/jerrysion/p/5522673.html

 

JavaScript中提供了3个函数用来对URL编码以得到合法的URL：

1、escape()

2、encodeURI（）

3、encodeURIComponent（）

 

## **0x02浏览器解码规则**

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)



浏览器无论什么情况都会遵守一个这样的解码规则：

1、HTML解析器对HTML文档进行解析，完成HTML解码并且创建DOM树

2、JavaScript 或者 CSS解析器对内联脚本进行解析，完成JS、CSS解码

3、URL解码会根据URL所在的顺序不同而在JS解码前或者解码后

## **0x03HTML解析器**

**HTML中有五类元素：**

1、空元素(Void elements)，有area、base、br、col、command、embed、hr、img、input、keygen、link、meta、param、source、track、wbr

2、原始文本元素(Raw text elements)，有<script>和<style>

3、RCDATA元素(RCDATA elements)，有<textarea>和<title>

4、外部元素(Foreign elements)，例如MathML命名空间或者SVG命名空间的元素

5、基本元素(Normal elements)，即除了以上4种元素以外的元素

 

**五类元素的区别如下：**

1、空元素，不能容纳任何内容（因为它们没有闭合标签，没有内容能够放在开始标签和闭合标签中间）。

2、原始文本元素，可以容纳文本。

3、RCDATA元素，可以容纳文本和字符引用。

4、外部元素，可以容纳文本、字符引用、CDATA段、其他元素和注释

5、基本元素，可以容纳文本、字符引用、其他元素和注释

 

**HTML解析器以状态机的方式运行，它从文档输入流中消耗字符并根据其转换规则转换到不同的状态。
**

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)





 

示例：

- 
- 
- 
- 
- 

```
<html> <body>   Hello world </body></html>
```



1、初始状态为"Data State"，当遇到"<"字符，状态变为"Tag open state"，读取一个a-z的字符将产生一个开始标签符号，状态相应变为"Tag name state"，一直保持这个状态直到读取到">"，每个字符都附加到这个符号名上，例子中创建的是一个html符号。

2、当读取到">"，当前的符号就完成了，此时，状态回到"Datastate"，"<body>"重复这一处理过程。到这里，html和body标签都识别出来了。现在，回到"Data state"，读取"Helloworld"中的字符"H"将创建并识别出一个字符符号，这里会为"Hello world"中的每个字符生成一个字符符号。

3、这样直到遇到"</body>"中的"<"。现在，又回到了"Tag open state"，读取下一个字符"/"将创建一个闭合标签符号，并且状态转移到"Tag name state"，还是保持这一状态，直到遇到">"。然后，产生一个新的标签符号并回到"Data state"。后面的"</html>"将和"</body>"一样处理。

 

HTML解析器处于**数据状态（Data State）**、**RCDATA 状态（RCDATA State）**、**属性值状态（Attribute ValueState）**时，字符实体会被解码为对应的字符。

示例：

- 
- 
- 
- 
- 

```
<div>&#60;img src=x onerror=alert(4)&#62;</div><和>被编码为字符实体&#60;和&#62;。当HTML解析器解析完<div>时，会进入数据状态（Data State）并发布标签令牌。接着解析到实体&#60;时因为处在数据状态（Data State）就会对实体进行解码为<，后面的&#62;同样道理被解码为>。
```

这里会有个问题，被解码后，img是否会被解析为HTML标签而导致JS执行呢？

答案是否定的。因为解析器在使用字符引用后不会转换到标签打开状态（Tag Open State），不进入标签打开状态就不会被发布为HTML标签。因此，不会创建新HTML标签，只会将其作为数据来处理。

这也是为什么我们可以使用字符实体来避免用户不安全输入导致XSS的原因。

 

**原始文本元素(Raw text elements)**

在HTML中，属于Raw text elements的标签有两个：script、style。

在Raw text elements类型标签下的所有内容块都属于该标签。

**存在一条特性：**

Raw textelements类型标签下的所有字符实体编码都不会被HTML解码。HTML解析器解析到script、style标签的内容块（数据）部分时，状态会进入Script Data State，该状态并不在我们前面说的会解码字符实体的三条状态之中。

因此，<script>&#97;&#108;&#101;&#114;&#116&#40;&#57;&#41;&#59</script>这样字符实体并不会被解码，也就不会执行JS。

 

**RCDATA**

在HTML中，属于RCDATAElements的标签有两个：textarea、title。

RCDATA Elements类型的标签可以包含文本内容和字符实体。

解析器解析到textarea、title标签的数据部分时，状态会进入RCDATA State。

前面我们提到，处于RCDATA State状态时，字符实体是会被解析器解码的。

示例：

- 
- 
- 
- 
- 

```
<textarea>&#60;script&#62;alert(5)&#60;/script&#62;</textarea><和>被编码为实体&#60;和&#62;。解析器解析到它们时会进行解码，最终得到<textarea><script>alert(5)</script></textarea>。但是里面的JS同样还是不会被执行，原因还是因为解码字符实体状态机不会进入标签打开状态（Tag Open State），因此里面的<script>并不会被解析为HTML标签
```

**外部元素(Foreignelements)**

Foreign elemnts来源于MathML和SVG命名空间

<svg>遵循XML和SVG的定义

示例

<script>alert&#40;1)</script>不能弹窗，Raw text elements类型标签下的所有字符实体编码都不会被HTML解码<svg><script>alert&#40;1)</script>能弹窗，在XML中，&#40;会被解析成(，在XML中实体会自动转义,除了<![CDATA[和]]>包含的实体

##  

## **0x04JavaScript解析器**

形如 \uXXXX这样的Unicode字符转义序列或Hex编码是否能被解码需要看情况。

首先，JavaScript中有三个地方可以出现Unicode字符转义序列：

1、字符串中（in String）

Unicode转义序列出现在字符串中时，它只会被解释为普通字符，而不会破坏字符串的上下文。

例如，<script>alert("\u0031\u0030");</script>

被编码转义的部分为10，是字符串，会被正常解码，JS代码也会被执行。

 

2、标识符中（in identifier names）

若Unicode转义序列存在于标识符中，即变量名（如函数名等…），它会被进行解码。

例如，<script>\u0061\u006c\u0065\u0072\u0074(10);</script>

被编码转义的部分为alert字符，是函数名，属于在标识符中的情况，因此会被正常解码，JS代码也会被执行。

 

3、控制字符中（in control characters）

若Unicode转义序列存在于控制字符中，那么它会被解码但不会被解释为控制字符，而会被解释为标识符或字符串字符的一部分。

控制字符即'、"、()等。

例如，<script>alert\u0028"xss"); </script>，(进行了Unicode编码，那么解码后它不再是作为控制字符，而是作为标识符的一部分alert(。

因此函数的括号之类的控制字符进行Unicode转义后是不能被正常解释的。

 

**总结，Unicode序列不能出现在控制字符中，否则不能被解释。**

 

示例1：

- 
- 
- 
- 

```
<script>\u0061\u006c\u0065\u0072\u0074\u0028\u0031\u0031\u0029</script>被编码部分为alert(11)。该例子中的JS不会被执行，因为控制字符被编码了。
```

 

示例2：

- 
- 
- 

```
<script>\u0061\u006c\u0065\u0072\u0074(\u0031\u0032)</script>被编码部分为alert及括号内为12。该例子中JS不会被执行，原因在于括号内被编码的部分不能被正常解释，要么使用ASCII数字，要么加""或''使其变为字符串，作为字符串也只能作为普通字符。
```

 

示例3：

 

- 
- 
- 
- 

```
<script>alert('13\u0027)</script>被编码处为'。该例的JS不会执行，因为控制字符被编码了，解码后的'将变为字符串的一部分，而不再解释为控制字符。因此该例中字符串是不完整的，因为没有'来结束字符串。
```



示例4：

- 
- 
- 

```
<script>alert('14\u000a')</script>该例的JS会被执行，因为被编码的部分处于字符串内，只会被解释为普通字符，不会突破字符串上下文。
```

 

示例5：

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
<img src="1" onerror=\u0061\u006c\u0065\u0072\u0074\u0028\u0031\u0029>无法执行我们以浏览器的视角来看：首先读到<开始读取标签，然后读到onerror调用JS解析器。在JS中，单引号，双引号和圆括号等属于控制字符，编码后将无法识别。所以对于防御来说，应该编码这些控制字符。下面这种方式可以解析：<img src="1" onerror=\u0061\u006c\u0065\u0072\u0074('\u0031')>可以结合上面的HTML编码按照解析顺序反过去，先JS编码然后HTML解码<img src="1" onerror=&#92;&#117;&#48;&#48;&#54;&#49;&#92;&#117;&#48;&#48;&#54;&#99;&#92;&#117;&#48;&#48;&#54;&#53;&#92;&#117;&#48;&#48;&#55;&#50;&#92;&#117;&#48;&#48;&#55;&#52;&#40;&#39;&#92;&#117;&#48;&#48;&#51;&#49;&#39;&#41;>浏览器读到了<标签开始构造语法树，然后HTML解码，解码之后发现onerror于是进行一个JS解码，成功弹窗延伸：开发人员单纯的设置HTML实体编码为防御xss的手段，但是用户输入点在alert中<img src = "https://text.com" onclick = 'alert("输入点")'>如果用户正常输入的话凡是存在< ," 等都能被转码攻击者可以通过语句 ");alert("test，在服务端被转码：<img src = "https://gss1.bdstatic.com" onclick = 'alert("FIRST XSS&#34;&#41;&#59;&#97;&#108;&#101;&#114;&#116;&#40;&#34;&#116;&#101;&#115;&#116;")'>弹窗两次，是因为浏览器进行HTML解码发现存在两个alert()所以对于这种情况，正确防御XSS的方法：应该是先JavaScript编码然后再进行HTML编码用户输入 ");alert("test 后在服务端先JavaScript编码然后再进行HTML编码在在浏览器端：首先经过第一步HTML解码后变为\u0022\u0029\u003B\u0061\u006C\u0065\u0072\u0074\u0028\u0022\u0074\u0065\u0073\u0074JavaScript解析器工作，变为 ");alert("test ，刚才已经讲过JavaScript解析时只有标识符名称不会被当做字符串，控制字符仅会被解析为标示符名称或者字符串，因此\u0022被解释成双引号文本，\u0028和\u0029被解释成为圆括号文本，不会变为控制字符被解析执行。在这里采用的先JS编码后HTML编码中只弹窗了一次。
```

 

## **0x05URL解析器**

**通用URI的格式如下：**

[协议名]: //用户名:密码@主机名:端口/路径?查询参数#片段ID

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

URL解析器也被建模为状态机，文档输入流中的字符可以将其导向不同的状态。

首先，要注意的是URL的Scheme部分（协议部分）必须为ASCII字符，即不能被任何编码，否则URL解析器的状态机将进入No Scheme状态。

示例：

 

- 
- 
- 

```
<a href="%6a%61%76%61%73%63%72%69%70%74:%61%6c%65%72%74%28%31%29"></a>URL编码部分的是javascript:alert(1)。JS不会被执行，因为作为Scheme部分的"javascript"这个字符串被编码，导致URL解析器状态机进入No Scheme状态。
```



URL中的:也不能被以任何方式编码，否则URL解析器的状态机也将进入No Scheme状态。

- 
- 

```
<a href="javascript%3aalert(3)"></a>由于:被URL编码为%3a，导致URL状态机进入No Scheme状态，JS代码不能执行。
```

 示例：

- 
- 
- 
- 

```
<a href="&#x6a;&#x61;&#x76;&#x61;&#x73;&#x63;&#x72;&#x69;&#x70;&#x74;:%61%6c%65%72%74%28%32%29">"javascript"这个字符串被实体化编码，:没有被编码，alert(2)被URL编码。成功执行首先，在HTML解析器中我们谈到过，HTML状态机处于属性值状态（Attribute Value State）时，字符实体时会被解码的，此处在href属性中，所以被实体化编码的"javascript"字符串会被解码。其次，HTML解析是在URL解析之前的，所以在进行URL解析之前，Scheme部分的"javascript"字符串已被解码，而并不再是被实体编码的状态。
```

## **0x06解析顺序**

首先浏览器接收到一个HTML文档时，会触发HTML解析器对HTML文档进行词法解析，这一过程完成HTML解码并创建DOM树。

接下来JavaScript解析器会介入对内联脚本进行解析，这一过程完成JS的解码工作。

如果浏览器遇到需要URL的上下文环境，这时URL解析器也会介入完成URL的解码工作，URL解析器的解码顺序会根据URL所在位置不同，可能在JavaScript解析器之前或之后解析。HTML解析总是第一步。

 

URL解析和JavaScript解析，它们的解析顺序要根据情况而定。

示例1：

- 
- 
- 
- 
- 

```
<a href="UserInput"></a>该例子中，首先由HTML解析器对UserInput部分进行字符实体解码；接着URL解析器对UserInput进行URL decode；如果URL的Scheme部分为javascript的话，JavaScript解析器会再对UserInput进行解码。所以解析顺序是：HTML解析->URL解析->JavaScript解析。
```

 

示例2：

- 
- 
- 
- 
- 

```
<a href=# onclick="window.open('UserInput')"></a>该例子中，首先由HTML解析器对UserInput部分进行字符实体解码；接着由JavaScript解析器会再对onclick部分的JS进行解析并执行JS；执行JS后window.open('UserInput')函数的参数会传入URL，所以再由URL解析器对UserInput部分进行解码。因此解析顺序为：HTML解析->JavaScript解析->URL解析。
```

 

示例3：

- 
- 
- 
- 
- 
- 
- 

```
<a href="javascript:window.open('UserInput')">该例子中，首先还是由HTML解析器对UserInput部分进行字符实体解码；接着由URL解析器解析href的属性值；然后由于Scheme为javascript，所以由JavaScript解析；解析执行JS后window.open('UserInput')函数传入URL，所以再由URL解析器解析。所以解析顺序为：HTML解析->URL解析->JavaScript解析->URL解析。
```

 

综合实例：

- 
- 
- 
- 
- 
- 
- 
- 
- 
- 
- 

```
<a href="&#x6a;&#x61;&#x76;&#x61;&#x73;&#x63;&#x72;&#x69;&#x70;&#x74;&#x3a;&#x25;&#x35;&#x63;&#x25;&#x37;&#x35;&#x25;&#x33;&#x30;&#x25;&#x33;&#x30;&#x25;&#x33;&#x36;&#x25;&#x33;&#x31;&#x25;&#x35;&#x63;&#x25;&#x37;&#x35;&#x25;&#x33;&#x30;&#x25;&#x33;&#x30;&#x25;&#x33;&#x36;&#x25;&#x36;&#x33;&#x25;&#x35;&#x63;&#x25;&#x37;&#x35;&#x25;&#x33;&#x30;&#x25;&#x33;&#x30;&#x25;&#x33;&#x36;&#x25;&#x33;&#x35;&#x25;&#x35;&#x63;&#x25;&#x37;&#x35;&#x25;&#x33;&#x30;&#x25;&#x33;&#x30;&#x25;&#x33;&#x37;&#x25;&#x33;&#x32;&#x25;&#x35;&#x63;&#x25;&#x37;&#x35;&#x25;&#x33;&#x30;&#x25;&#x33;&#x30;&#x25;&#x33;&#x37;&#x25;&#x33;&#x34;&#x28;&#x31;&#x35;&#x29;"></a>
```

首先HTML解析器进行解析，解析到href属性的值时，状态机进入属性值状态（Attribute Value State），该状态会解码字符实体

接着由URL解析器进行解析并解码

再接着由于Scheme为javascript，因此由JavaScript解析器解析并解码，加上编码部分是函数名，属于标识符，因此可以正常解码解释

经过三轮解析解码后得到结果：<ahref="javascript:alert(15)"></a>



到这里，你们以为就结束了吗?

+++哼，太天真了.gif+++

Look!!!

Look!!!

Look!!!

剩下的是往期精彩^_^

**往期经典推荐：**

**----------------------------------------------------------------------**

[**Apache Solr远程命令执行复现**](http://mp.weixin.qq.com/s?__biz=MzAwMzc2MDQ3NQ==&mid=2247483749&idx=1&sn=da603bc578831b4bb5af92503def88ae&chksm=9b370951ac408047ca92cdb27f90c8c430187b15379e565ec0460069f1ca68f2924c5f0e1694&scene=21#wechat_redirect)

[**“最后”的Bypass CDN 查找网站真实IP**](http://mp.weixin.qq.com/s?__biz=MzAwMzc2MDQ3NQ==&mid=2247483674&idx=1&sn=da83b8ed28783252a1798974f91ea8aa&chksm=9b37092eac40803805d2662f93d77ce88bdb5e18efeec9bc684dfdaa6daeb8d95cf6f92669db&scene=21#wechat_redirect)

[**渗透实战（一）|BSides Vancouver 2018 (Workshop)**](http://mp.weixin.qq.com/s?__biz=MzAwMzc2MDQ3NQ==&mid=2247483709&idx=1&sn=d08306000317cdc1bd19e102298c9762&chksm=9b370909ac40801f834e5aad83b3d58733f7a85a548b437ee814bd3e8a9f67678976a3e0fce0&scene=21#wechat_redirect)

[**移动安全（一）|Android设备root及神器Xposed框架安装**](http://mp.weixin.qq.com/s?__biz=MzAwMzc2MDQ3NQ==&mid=2247483763&idx=1&sn=413f469223f181f48f82a344ce1d9c06&chksm=9b370947ac4080514962fbe586ba525ddc9029ab02ed72ffc9aac458859ac428e9432bb76f6b&scene=21#wechat_redirect)

[**内网信息收集篇**](http://mp.weixin.qq.com/s?__biz=MzAwMzc2MDQ3NQ==&mid=2247483767&idx=1&sn=1b8f18771eff3afd25bce1e41e1f7fe3&chksm=9b370943ac408055fb20907f7a06b06fc0c60f051d560e3661704c04c289485129d28fff9bbd&scene=21#wechat_redirect)

[**MSF 下域内渗透**](http://mp.weixin.qq.com/s?__biz=MzAwMzc2MDQ3NQ==&mid=2247483757&idx=1&sn=b61cdd38f5ade560af07f9321176d58e&chksm=9b370959ac40804f7384f4484e2586be29969306bd3eef06b90767fa2363d57436bf1e180424&scene=21#wechat_redirect)

## 本篇相关参考资料：

https://sakuxa.com/2019/03/07/0x00-XSS%E5%AD%A6%E4%B9%A0%E7%B3%BB%E5%88%97%E4%B9%8B%E8%A7%A3%E6%9E%90HTML%E6%96%87%E6%A1%A3/

https://xz.aliyun.com/t/5863

https://www.w3.org/html/ig/zh/wiki/HTML5/syntax

https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element

\>>关于我们：

WhITECat安全小组隶属于起源实验室分支安全小组，主要致力于分享小组成员技术研究成果、最新的漏洞新闻、安全招聘以及其他安全相关内容。团队成员暂时由起源实验室核心成员、一线安全厂商、某研究院、漏洞盒TOP10白帽子等人员组成。

欢迎各位大佬关注^_^

![img](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

 

 

 



![img](https://mp.weixin.qq.com/mp/qrcode?scene=10000004&size=102&__biz=MzAwMzc2MDQ3NQ==&mid=2247483785&idx=1&sn=f04335646590f49b23e37be4381a949d&send_time=)

微信扫一扫
关注该公众号