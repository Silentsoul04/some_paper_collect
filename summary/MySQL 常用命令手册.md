\> 本文由 \[简悦 SimpRead\](http://ksria.com/simpread/) 转码， 原文地址 \[mp.weixin.qq.com\](https://mp.weixin.qq.com/s/BzxQ7Fz4IVaiQeVTmhKx-Q)

> **作者：宁静. 致远**
> 
> **链接：https://www.cnblogs.com/zhangzhu/archive/2013/07/04/3172486.html**

**常用功能命令**

1\. 导出整个数据库  

1mysqldump -u 用户名 -p –default-character-set=latin1 数据库名 > 导出的文件名 (数据库默认编码是 latin1)    
2  
3mysqldump -u wcnc -p smgp\_apps\_wcnc > wcnc.sql    

2\. 导出一个表  

`1mysqldump -u 用户名 -p 数据库名 表名> 导出的文件名    
2  
3mysqldump -u wcnc -p smgp_apps_wcnc users> wcnc_users.sql` 

3\. 导出一个数据库结构  

`1mysqldump -u wcnc -p -d –add-drop-table smgp_apps_wcnc >d:wcnc_db.sql    
2  
3-d 没有数据 –add-drop-table 在每个create语句之前增加一个drop table` 

4\. 导入数据库  

`1A:常用source 命令    
2  
3进入mysql数据库控制台，    
4  
5如mysql -u root -p    
6  
7mysql>use 数据库    
8  
9然后使用source命令，后面参数为脚本文件(如这里用到的.sql)    
10  
11mysql>source wcnc_db.sql    
12  
13B:使用mysqldump命令    
14  
15mysqldump -u username -p dbname < filename.sql    
16  
17C:使用mysql命令    
18  
19mysql -u username -p -D dbname < filename.sql` 

**启动与退出**  

1、进入 MySQL：启动 MySQL Command Line Client（MySQL 的 DOS 界面），直接输入安装时的密码即可。此时的提示符是：mysql>  

2、退出 MySQL：quit 或 exit  

**库操作**  

1、、创建数据库  

命令：create database <数据库名>  

例如：建立一个名为 sqlroad 的数据库  

mysql> create database sqlroad;  

2、显示所有的数据库  

命令：show databases （注意：最后有个 s）  

mysql> show databases;  

3、删除数据库  

命令：drop database <数据库名>  

例如：删除名为 sqlroad 的数据库  

mysql> drop database sqlroad;  

4、连接数据库  

命令：use <数据库名>  

例如：如果 sqlroad 数据库存在，尝试存取它： 

mysql> use sqlroad;  

屏幕提示：Database changed  

5、查看当前使用的数据库  

mysql> select database();  

6、当前数据库包含的表信息： 

mysql> show tables; （注意：最后有个 s）  

**表操作，操作之前应连接某个数据库** 

1、建表  

`1命令：create table <表名> ( <字段名> <类型> [,..<字段名n> <类型n>]);    
2  
3mysql> create table MyClass(    
4  
5> id int(4) not null primary key auto_increment,    
6  
7> name char(20) not null,    
8  
9> sex int(4) not null default ’′,    
10  
11> degree double(16,2));` 

2、获取表结构  

`1命令：desc 表名，或者show columns from 表名    
2  
3mysql>DESCRIBE MyClass    
4  
5mysql> desc MyClass;    
6  
7mysql> show columns from MyClass;` 

3、删除表  

`1命令：drop table <表名>    
2  
3例如：删除表名为 MyClass 的表    
4  
5mysql> drop table MyClass;` 

4、插入数据  

`1命令：insert into <表名> [( <字段名>[,..<字段名n> ])] values ( 值 )[, ( 值n )]    
2  
3例如，往表 MyClass中插入二条记录, 这二条记录表示：编号为的名为Tom的成绩为.45, 编号为 的名为Joan 的成绩为.99，编号为 的名为Wang 的成绩为.5.    
4  
5mysql> insert into MyClass values(1,’Tom’,96.45),(2,’Joan’,82.99), (2,’Wang’, 96.59);` 

5、查询表中的数据  

`11)、查询所有行    
2  
3命令：select <字段，字段，...> from < 表名 > where < 表达式 >    
4  
5例如：查看表 MyClass 中所有数据    
6  
7mysql> select * from MyClass;    
8  
92）、查询前几行数据    
10  
11例如：查看表 MyClass 中前行数据    
12  
13mysql> select * from MyClass order by id limit 0,2;    
14  
15或者：    
16  
17mysql> select * from MyClass limit 0,2;` 

6、删除表中数据  

`1命令：delete from 表名 where 表达式    
2  
3例如：删除表 MyClass中编号为 的记录    
4  
5mysql> delete from MyClass where id=1;` 

7、修改表中数据：update 表名 set 字段 = 新值,…where 条件  

`1mysql> update MyClass set name=’Mary’where id=1;` 

8、在表中增加字段： 

`1命令：alter table 表名 add字段 类型 其他;    
2  
3例如：在表MyClass中添加了一个字段passtest，类型为int(4)，默认值为    
4  
5mysql> alter table MyClass add passtest int(4) default ’′    
6  
`

9、更改表名： 

`1命令：rename table 原表名 to 新表名;    
2  
3例如：在表MyClass名字更改为YouClass    
4  
5mysql> rename table MyClass to YouClass;    
6  
7更新字段内容    
8  
9update 表名 set 字段名 = 新内容    
10  
11update 表名 set 字段名 = replace(字段名,’旧内容’, 新内容’)    
12  
13update article set content=concat(‘　　’,content);` 

**字段类型和数据库操作**

1．INT\[(M)\] 型：正常大小整数类型  

2．DOUBLE\[(M,D)\] \[ZEROFILL\] 型：正常大小 (双精密) 浮点数字类型  

3．DATE 日期类型：支持的范围是 - 01-01 到 - 12-31。MySQL 以 YYYY-MM-DD 格式来显示 DATE 值，但是允许你使用字符串或数字把值赋给 DATE 列  

4．CHAR(M) 型：定长字符串类型，当存储时，总是是用空格填满右边到指定的长度  

5．BLOB TEXT 类型，最大长度为 (2^16-1) 个字符。 

6．VARCHAR 型：变长字符串类型  

7\. 导入数据库表  

`1创建.sql文件    
2  
3先产生一个库如auction.c:mysqlbin>mysqladmin -u root -p creat auction，会提示输入密码，然后成功创建。    
4  
5导入auction.sql文件    
6  
7c:mysqlbin>mysql -u root -p auction < auction.sql。    
8  
9通过以上操作，就可以创建了一个数据库auction以及其中的一个表auction。` 

8．修改数据库  

`1在mysql的表中增加字段：    
2  
3alter table dbname add column userid int(11) not null primary key auto_increment;    
4  
5这样，就在表dbname中添加了一个字段userid，类型为int(11)。` 

9．mysql 数据库的授权  

`1mysql>grant select,insert,delete,create,drop    
2  
3on *.* (或test.*/user.*/..)    
4  
5to 用户名@localhost    
6  
7identified by ‘密码’；    
8  
9如：新建一个用户帐号以便可以访问数据库，需要进行如下操作：    
10  
11mysql> grant usage    
12  
13　　-> ON test.*    
14  
15　　-> TO testuser@localhost;    
16  
17　　Query OK, 0 rows affected (0.15 sec)    
18  
19　　此后就创建了一个新用户叫：testuser，这个用户只能从localhost连接到数据库并可以连接到test 数据库。下一步，我们必须指定testuser这个用户可以执行哪些操作：    
20  
21　　mysql> GRANT select, insert, delete,update    
22  
23　　-> ON test.*    
24  
25　　-> TO testuser@localhost;    
26  
27　　Query OK, 0 rows affected (0.00 sec)    
28  
29　　此操作使testuser能够在每一个test数据库中的表执行SELECT，INSERT和DELETE以及UPDATE查询操作。现在我们结束操作并退出MySQL客户程序：    
30  
31　　mysql> exit` 

**DDL 操作**

1: 使用 SHOW 语句找出在服务器上当前存在什么数据库： 

mysql> SHOW DATABASES;  

2、创建一个数据库 MYSQLDATA  

mysql> Create DATABASE MYSQLDATA;  

3: 选择你所创建的数据库  

mysql> USE MYSQLDATA; (按回车键出现 Database changed 时说明操作成功！)  

4: 查看现在的数据库中存在什么表  

mysql> SHOW TABLES;  

5: 创建一个数据库表  

mysql> Create TABLE MYTABLE (name VARCHAR(20), sex CHAR(1));  

6: 显示表的结构： 

mysql> DESCRIBE MYTABLE;  

7: 往表中加入记录  

mysql> insert into MYTABLE values (“hyq”,”M”);  

8: 用文本方式将数据装入数据库表中（例如 D:/mysql.txt）  

mysql> LOAD DATA LOCAL INFILE “D:/mysql.txt”INTO TABLE MYTABLE;  

9: 导入. sql 文件命令（例如 D:/mysql.sql）  

mysql>use database;  

mysql>source d:/mysql.sql;  

10: 删除表  

mysql>drop TABLE MYTABLE;  

11: 清空表  

mysql>delete from MYTABLE;  

12: 更新表中数据  

mysql>update MYTABLE set sex=”f”where name=’hyq’;

**推荐↓↓↓**

 

![](https://mmbiz.qpic.cn/mmbiz_jpg/NVvB3l3e9aG5kWic5P8XOwFOhXKjibAt6YJ3ggABxW2baNFjLg3bImTncnCzDic7FXUlhx7KK7gksUlCPjjw5PXRw/640?wx_fmt=jpeg)

**Web 开发**