> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.cnblogs.com](https://www.cnblogs.com/lovequitepcs/p/12864203.html)

本文仅为了学习交流，严禁非法使用！！！  
(随笔仅为平时的学习记录，若有错误请大佬指出)

1. 分析一下存在漏洞文件 `logincheck_code.php`

![](https://img2020.cnblogs.com/blog/1945846/202005/1945846-20200510162206209-811012641.png)  
![](https://img2020.cnblogs.com/blog/1945846/202005/1945846-20200510163018329-1754729861.png)  
![](https://img2020.cnblogs.com/blog/1945846/202005/1945846-20200510163032809-1198703711.png)

从代码中我们可以控制的有`UID,CODEUID`然后判断`$login_codeuid`是否存在，不存在或者为空就退出，然后将获取的`UID`，带入到`sql`语句进行查询，后面会验证查询的结果，如果信息核对正确，则将个人信息放入到`SESSION`中，`UID=1`的时候默认是管理员，而`UID`我们可以输入一个`1`，就可以满足`查询ql`, 而我们现在要绕过`if (!isset($login_codeuid) || empty($login_codeuid)){exit();}`，全局搜索一下`$login_codeuid`可不可以控制

2. 分析一下`/general/login_code.php`，发现存在`$login_codeuid`

![](https://img2020.cnblogs.com/blog/1945846/202005/1945846-20200510165714923-1156479536.png)  
![](https://img2020.cnblogs.com/blog/1945846/202005/1945846-20200510165750216-739345737.png)

如果`$login_codeuid`为空，则给`$login_codeuid`赋一个随机值，并且使用`echo`打印出来

3. 那我们的思路是: 先去访问`/general/login_code.php`, 获得`$login_codeuid`, 再去访问`logincheck_code.php`，同时`POST`传入获取的`$login_codeuid`和`UID=1`, 获得返回包的`PHPSESSID`, 在使用火狐浏览器伪造`COOKiE`, 登录后台

4. 漏洞复现

![](https://img2020.cnblogs.com/blog/1945846/202005/1945846-20200510170210411-80051878.png)

将获得的`code_uid`保存下来，进行下一步

![](https://img2020.cnblogs.com/blog/1945846/202005/1945846-20200510170321260-203587801.png)

将获取的`PHPSESSID`保存下来，使用火狐浏览器伪造`COOKIE`，同时访问`/general/index.php`，便可以进入后台

![](https://img2020.cnblogs.com/blog/1945846/202005/1945846-20200510170526791-2131080026.png)

5. 后台 GetShell(靶机环境 windows7 通道 OA 用的是 MYOA2017)

依次点击系统管理 - 附件管理 - 添加存储目录，选择根目录  
![](https://img2020.cnblogs.com/blog/1945846/202005/1945846-20200510170831913-929601240.png)  
![](https://img2020.cnblogs.com/blog/1945846/202005/1945846-20200510170914077-1973593563.png)

6. 依次点击组织 - 系统管理员 - 附件 (下图标注)

![](https://img2020.cnblogs.com/blog/1945846/202005/1945846-20200510171027828-555807909.png)

7. 直接上传`shell.php`不能成功，开启抓包，上传`shell.php.`进行绕过，`windows`系统会自动去掉`.`，不符合`windows`的命名

![](https://img2020.cnblogs.com/blog/1945846/202005/1945846-20200510171601436-167619187.png)

8. 使用冰蝎进行连接

![](https://img2020.cnblogs.com/blog/1945846/202005/1945846-20200510171733338-2045925697.png)

9. 实战 (由以上的思路，进入到某站的 OA 后台管理系统)

![](https://img2020.cnblogs.com/blog/1945846/202005/1945846-20200512224358588-914525354.png)

10. 修复建议  
升级通达 OA 到最新版  
参考文章  
https://www.chabug.org/audit/1516.html  
https://blog.csdn.net/sun1318578251/article/details/105728541/

此文档仅供学习，参与违法行为与笔者无关。