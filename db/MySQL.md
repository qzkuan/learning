

## install

1.从官网下载mysql5.7.tar.gz

2.使用ftp把mysql的压缩包上传到服务器上（指定文件夹 /mysqldata）

3.解压mysql压缩包
`tar -zxvf mysql-5.7.27-linux-glibc2.12-x86_64.tar.gz `

4.解压后删除压缩包
`rm -rf mysql-5.7.27-linux-glibc2.12-x86_64.tar.gz `

5.修改mysql目录的名称，简单点方便后面的配置
`mv mysql-5.7.27-linux-glibc2.12-x86_64 mysql`

6.进入mysql目录，在该目录中创建data目录(用于存放日志的目录)
`mkdir data`

7.创建mysql的用户群组
`groupadd mysql`

8.创建mysql群组下的用户（第一个mysql是群组名称，第二个是用户名称）
`useradd -r -g mysql mysql -d /home/mysql`

9.为创建的mysql新用户进行授权
`chown -R mysql:mysql /home/mysql`

`chown -R mysql:mysql /mysqldata`

10.初始化mysql数据库（在mysql目录下的bin目录中进行）

创建相关文件夹并赋权限

修改my.cnf文件`vim /etc/my.cnf`
进入到该配置文件中，进行配置修改，修改如下，完成后保存退出

```
[mysqld]
port=3306
user=mysql
basedir=#安装目录#/mysql
datadir=#安装目录#/mysql/data
socket=#安装目录#/mysql/mysql.sock
pid-file=#安装目录#/mysql/data/mysqld.pid
character-set-server=utf8
lower_case_table-names=1
max_connections=800
max_connect_errors=1000
default-storage-engine=INNODB
transaction_isolation-READ-COMMITTED
sql_mode=ANSI_QUOTES,STRICT_TRANS_TABLES,NO_ENGINE_SUBSITITUTTON,NO_ZERO_DATE,NO_ZERO_IN_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER
read_buffer_size=#每8G内存配1M#
read_rnd_buffer_size=32M
back_log=1024
log-output=FILE
general_log=0
log-error=#安装目录#/mysql/log/mariadb.log
innodb_buffer_pool_size=2G
innodb_buffer_pool_instances=16

```



```
[mysqld]
basedir=/home/apps/mysql  
datadir=/home/apps/mysql/data 
socket=/tmp/mysql.sock
user=mysql
port=3306
character_set_server=utf8
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

[mysqld_safe]
log-error=/home/apps/mysql/data/error.log
pid-file=/home/apps/mysql/data/mysqld.pid
tmpdir=/tmp
```

！！！注意在初始化mysql数据库的时候会出现一个默认的登录密码，记录下来，后面需要用到！！！

注意启动的是 mysqld 程序 而不是 mysql 

`./bin/mysqld --initialize --user=mysql --explicit_defaults_for_timestamp `
 执行初始化命令后的得到如下提示，其中 fU5gkNj*Jxtp 就是初始密码（每个人密码有所不同）

`A temporary password is generated for root@localhost: fU5gkNj*Jxtp`

11.打开SSL加密（可选）

`./bin/mysql_ssl_rsa_setup --datadir=/mysqldata/mysql/data`

12.把mysql添加到系统服务中

在mysql目录下进行，将mysql目录下的support-files目录中的mysql.server文件复制到路径  /etc/init.d/mysqld

`cp support-files/mysql.server /etc/init.d/mysql`
mysqld文件并不存在(也就是说在init.d目录下并不存在mysqld)，是把mysql.server文件复制过去后修改了名字，mysqld 其实就是mysql.server文件

13.配置环境变量mysql

`ln -s /**/mysql/bin/mysql /usr/bin`

14.启动mysql服务（成功时会提示：SUCCESS!）
`service mysql start/stop/restart`

15.登录mysql(需要用到第十步的初始密码)

在mysql的bin目录下，注意此处启动的是 mysql 程序而不是 mysqld 

`./mysql -u root -p`
输入上面的密码，建议复制粘贴，密码输入正确后可看到欢迎提示 ：

```
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.7.27

Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```

16.修改root用户的密码（初始密码不方便使用）

注意：以下的sql命令在mysql> 后执行，必须要带分号";"，而且所有的字符串都必须使用单引号''，不能使用双引号

root123是自己的密码，可以自行更改

`set password for 'root'@localhost=password('root123');`

或者`alter user 'root'@'localhost' identified by 'root123'`

或者`update user set authentication_string=PASSWORD('root123') where user='root;'`

17.配置mysql的远程访问

mysql默认不支持远程访问，为了便于使用mysql可视化工具进行数据库操作（Navicat等工具）

查看所有的数据库：   show databases;
使用mysql数据库：   use mysql;
查看mysql数据库下的所有表名：show tables;

可以看到mysql数据库下有一个user表，执行：

`update user set host='%' where user='root' limit 1;`

18.提交和刷新数据库

`flush privileges;`

19.新建用户和数据库并赋予相关权限

```sql
create user 'admin'@'%' identified by 'admin1234';
create database demodb;
grant all privileges on demodb.* to guwpadmin@'%';
flush privileges;
```



### 问题及解决方案

1. 提示密码过期或者忘记密码

   - 修改/etc/my.cnf，在[mysqld]下添加`skip-grant-tables`
   - service mysql restart
   - mysql -u root -p
   - `use mysql;` `update user set password_expired='N' where user='root'`

   [注]初始密码可以在`/mysql/log/mariadb.log`里搜索`root`找到

2. mysql -u root -p 登录提示少库

   - yum install libncurses*

3. 提示/tmp/mysql.sock相关问题

   - ln -s /mysql/mysql.sock /tmp/mysql.sock
   
4. 建表语句不能用`""`包裹

   - session生效 set @@session.sql_mode=select concat(@@session.sql.mode,'ANSI_QUOTES');

   - 修改/etc/my.cnf文件，加上sql_mode=ANSI_QUOTES


------------------------------------



## lexical



DDL ：数据定义语言 `create table .../ drop table ... / rename ... to..../ truncate table.../alter table ...`

DML : 数据操纵语言

```sql
insert into ... values ...
update ... set ... where ...
delete from ... where ...
```

DCL : 数据控制语言  `commit : 提交 / rollback : 回滚 / 授权grant...to...  /revoke`

DQL: 数据查询语言 `select`

- `select ...` 组函数(MIN()/MAX()/SUM()/AVG()/COUNT())

- `from ...join ... on ...` 左外连接：left join ... on ... 右外连接: right join ... on ...

- `where ...`

  ```sql
  select col_name,[col_name1,...] from table_name where where_definition
  #条件比较操作符
  =    #等值比较
  <=>  #等值比较，包括与NULL的安全比较
  <>或!=  #不等值比较
  <，<=，>，>=  #其它比较符
  IN  #指定范围内值的存在性测试
  BETWEEN … AND …  #在某取值范围内
  IS NULL  #是否为空值
  IS NOT NULL  #是否为非空
  LIKE  #可使用通配符:%, _
  RLIKE或REGEXP  #可使用正则表达式的模式
  #逻辑操作符
  AND 
  OR
  NOT
  ```

  

- `group by ...` (oracle,SQL server中出现在select 子句后的非分组函数，必须出现在 group by子句后)

- `having ...` 用于过滤 组函数

- `order by ...` asc 升序， desc 降序

- `limit (0,4)` 限制N条数据 如: topN数据

> - union 并集
> - union all(有重复)
> - intersect 交集
> - minus 相减

```sql
select * from table limit 2,1;                
-- 含义是跳过2条取出1条数据，limit后面是从第2条开始读，读取1条信息，即读取第3条数据
select * from table limit 2 offset 1;     
-- 含义是从第1条（不包括）数据开始取出2条数据，limit后面跟的是2条数据，offset后面是从第1条开始读取，即读取第2,3条
```







## EXPLAIN Output Columns

| 列名          | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| id            | 执行编号，标识select所属的行。如果在语句中没子查询或关联查询，只有唯一的select，每行都将显示1。否则，内层的select语句一般会顺序编号，对应于其在原始语句中的位置 |
| select_type   | 显示本行是简单或复杂select。如果查询有任何复杂的子查询，则最外层标记为PRIMARY（DERIVED、UNION、UNION RESUlT） |
| table         | 访问引用哪个表（引用某个查询，如“derived3”）                 |
| type          | 数据访问/读取操作类型（ALL、index、range、ref、eq_ref、const/system、NULL） |
| possible_keys | 揭示哪一些索引可能有利于高效的查找                           |
| key           | 显示mysql决定采用哪个索引来优化查询                          |
| key_len       | 显示mysql在索引里使用的字节数                                |
| ref           | 显示了之前的表在key列记录的索引中查找值所用的列或常量        |
| rows          | 为了找到所需的行而需要读取的行数，估算值，不精确。通过把所有rows列值相乘，可粗略估算整个查询会检查的行数 |
| Extra         | 额外信息，如using index、filesort等                          |

### id

id是用来顺序标识整个查询中SELELCT 语句的，在嵌套查询中id越大的语句越先执行。该值可能为NULL，如果这一行用来说明的是其他行的联合结果。

   **id是SQL执行的顺序的标识,SQL从大到小的执行**

1. id相同时，执行顺序由上至下

2. 如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行
3. id如果相同，可以认为是一组，从上往下顺序执行；在所有组中，id值越大，优先级越高，越先执行

### select_type

表示查询的类型

| 类型               | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| simple             | 简单子查询，不包含子查询和union                              |
| primary            | 包含union或者子查询，最外层的部分标记为primary               |
| subquery           | 一般子查询中的子查询被标记为subquery，也就是位于select列表中的查询 |
| derived            | 派生表——该临时表是从子查询派生出来的，位于form中的子查询     |
| union              | 位于union中第二个及其以后的子查询被标记为union，第一个就被标记为primary如果是union位于from中则标记为derived |
| union result       | 用来从匿名临时表里检索结果的select被标记为union result       |
| dependent union    | 顾名思义，首先需要满足UNION的条件，及UNION中第二个以及后面的SELECT语句，同时该语句依赖外部的查询 |
| subquery           | 子查询中第一个SELECT语句                                     |
| dependent subquery | 和DEPENDENT UNION相对UNION一样                               |

### table

对应行正在访问哪一个表，表名或者别名

- 关联优化器会为查询选择关联顺序，左侧深度优先
- 当from中有子查询的时候，表名是derivedN的形式，N指向子查询，也就是explain结果中的下一列
- 当有union result的时候，表名是union 1,2等的形式，1,2表示参与union的query id

注意：MySQL对待这些表和普通表一样，但是这些“临时表”是没有任何索引的。

```sql
mysql> explain select * from (select * from ( select * from t1 where id=2602) a) b;
+----+-------------+------------+--------+-------------------+---------+---------+------+------+-------+
| id | select_type | table      | type   | possible_keys     | key     | key_len | ref  | rows | Extra |
+----+-------------+------------+--------+-------------------+---------+---------+------+------+-------+
|  1 | PRIMARY     | <derived2> | system | NULL              | NULL    | NULL    | NULL |    1 |       |
|  2 | DERIVED     | <derived3> | system | NULL              | NULL    | NULL    | NULL |    1 |       |
|  3 | DERIVED     | t1         | const  | PRIMARY,idx_t1_id | PRIMARY | 4       |      |    1 |       |
+----+-------------+------------+--------+-------------------+---------+---------+------+------+-------+
```

### type

type显示的是访问类型，是较为重要的一个指标，结果值从好到坏依次是：
 system > const > eq_ref > ref > fulltext > ref_or_null > index_merge > unique_subquery > index_subquery > range > index > ALL ，一般来说，得保证查询至少达到range级别，最好能达到ref。

| 类型   | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| All    | Full Table Scan，最坏的情况,全表扫描                         |
| index  | Full Index Scan，和全表扫描一样。只是扫描表的时候按照索引次序进行而不是行。主要优点就是避免了排序, 但是开销仍然非常大。如在Extra列看到Using index，说明正在使用覆盖索引，只扫描索引的数据，它比按索引次序全表扫描的开销要小很多 |
| range  | 范围扫描，一个有限制的索引扫描。key 列显示使用了哪个索引。当使用=、 <>、>、>=、<、<=、IS NULL、<=>、BETWEEN 或者 IN 操作符,用常量比较关键字列时,可以使用 range |
| ref    | 一种索引访问，它返回所有匹配某个单个值的行。此类索引访问只有当使用非唯一性索引或唯一性索引非唯一性前缀时才会发生。这个类型跟eq_ref不同的是，它用在关联操作只使用了索引的最左前缀，或者索引不是UNIQUE和PRIMARY KEY。ref可以用于使用=或<=>操作符的带索引的列。 |
| eq_ref | 最多只返回一条符合条件的记录。使用唯一性索引或主键查找时会发生 （高效） |
| const  | 当确定最多只会有一行匹配的时候，MySQL优化器会在查询前读取它而且只读取一次，因此非常快。当主键放入where子句时，mysql把这个查询转为一个常量（高效） |
| system | 这是const连接类型的一种特例，表仅有一行满足条件。            |
| Null   | 意味说mysql能在优化阶段分解查询语句，在执行阶段甚至用不到访问表或索引（高效） |

### possible_keys

显示查询使用了哪些索引，表示该索引可以进行高效地查找，但是列出来的索引对于后续优化过程可能是没有用的

指出MySQL能使用哪个索引在表中找到记录，查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询使用

### key

key列显示MySQL实际决定使用的键（索引）。如果没有选择索引，键是NULL。要想强制MySQL使用或忽视possible_keys列中的索引，在查询中使用FORCE INDEX、USE INDEX或者IGNORE INDEX。

### key_len

key_len列显示MySQL决定使用的键长度。如果键是NULL，则长度为NULL。使用的索引的长度。在不损失精确性的情况下，长度越短越好 。

### ref

ref列显示使用哪个列或常数与key一起从表中选择行。

### rows

rows列显示MySQL认为它执行查询时必须检查的行数。注意这是一个预估值。

### Extra

Extra是EXPLAIN输出中另外一个很重要的列，该列显示MySQL在查询过程中的一些详细信息，MySQL查询优化器执行查询的过程中对查询计划的重要补充信息。

| 类型                         | 说明                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| Using filesort               | MySQL有两种方式可以生成有序的结果，通过排序操作或者使用索引，当Extra中出现了Using filesort 说明MySQL使用了后者，但注意虽然叫filesort但并不是说明就是用了文件来进行排序，只要可能排序都是在内存里完成的。大部分情况下利用索引排序更快，所以一般这时也要考虑优化查询了。使用文件完成排序操作，这是可能是ordery by，group by语句的结果，这可能是一个CPU密集型的过程，可以通过选择合适的索引来改进性能，用索引来为查询结果排序。 |
| Using temporary              | 用临时表保存中间结果，常用于GROUP BY 和 ORDER BY操作中，一般看到它说明查询需要优化了，就算避免不了临时表的使用也要尽量避免硬盘临时表的使用。 |
| Not exists                   | MYSQL优化了LEFT JOIN，一旦它找到了匹配LEFT JOIN标准的行， 就不再搜索了。 |
| Using index                  | 说明查询是覆盖了索引的，不需要读取数据文件，从索引树（索引文件）中即可获得信息。如果同时出现using where，表明索引被用来执行索引键值的查找，没有using where，表明索引用来读取数据而非执行查找动作。这是MySQL服务层完成的，但无需再回表查询记录。 |
| Using index condition        | 这是MySQL 5.6出来的新特性，叫做“索引条件推送”。简单说一点就是MySQL原来在索引上是不能执行如like这样的操作的，但是现在可以了，这样减少了不必要的IO操作，但是只能用在二级索引上。 |
| Using where                  | 使用了WHERE从句来限制哪些行将与下一张表匹配或者是返回给用户。**注意**：Extra列出现Using where表示MySQL服务器将存储引擎返回服务层以后再应用WHERE条件过滤。 |
| Using join buffer            | 使用了连接缓存：**Block Nested Loop**，连接算法是块嵌套循环连接;**Batched Key Access**，连接算法是批量索引连接 |
| impossible where             | where子句的值总是false，不能用来获取任何元组                 |
| select tables optimized away | 在没有GROUP BY子句的情况下，基于索引优化MIN/MAX操作，或者对于MyISAM存储引擎优化COUNT(*)操作，不必等到执行阶段再进行计算，查询执行计划生成的阶段即完成优化。 |
| distinct                     | 优化distinct操作，在找到第一匹配的元组后即停止找同样值的动作 |





## 导入导出

### **一. **mysqldump

**工具基本用法，不适用于大数据备份**

1. 备份所有数据库: `mysqldump -u root -p --all-databases > all_database_sql`

2. 备份mysql数据库：`mysqldump -u root -p --databases mysql > mysql_database_sql`

3. 备份指定的多个数据库：`mysqldump -u root -p --databases db1 db2 db3 > bak.sql`

4. 备份mysql数据库下的user表：`mysqldump -u root -p mysql user > user_table`

> 如果不用`--databases`选项，在后期进行数据还原操作时，如果该数据库不存在，必须先创建该数据库；而在例子3指定多个数据库时，必须要加--databases参数，否则db2会被认为是db1库的表。

**把备份的所有数据文件还原：**

`mysql -u root -p < all_database_sql`，这里不需要指定库，因为是全部数据库

`mysql -u root -p mysql < mysql_database_sql` #这里就需要指定是mysql库了

 

### **二. 导出纯文本数据**

在某些情况下，为了一些特定的目的，经常需要将表里的数据导出为某些符号分割的纯数据文本，而不是 SQL 语句，因为LOAD DATA 的加载速度比普通的 SQL 加载要快 20 倍以上

 

#### 1.**SELECT ...INTO OUTFILE ...**

`mysql> SELECT * FROM tablename INTO OUTFILE 'target_file' [option];`

其中 option 参数可以是以下选项：

| 命令参数                                 | 说明                                                   |
| ---------------------------------------- | ------------------------------------------------------ |
| fields terminated by '字符'              | 字段分隔符，默认字符为制表符'\t'，16进制使用0x0F       |
| fields [optionally] enclosed by '单字符' | 字段引用符，加上optionally后在数字类型上不会有引用符号 |
| fields escaped by '单字符'               | 转义字符，默认为'\'                                    |
| lines starting by '字符'                 | 每行前都加此支付，默认为空                             |
| lines terminated by '字符'               | 行结束符，默认为'\n'，16进制使用0x0A                   |

 

**例子1，将 emp 表中数据导出为数据文本，其中，字段分隔符为“,”，每个字段用双引号引用起来，记录结束符为回车符(默认如此，可以不写)**

mysql> select * from emp into outfile '/tmp/emp.txt' fields terminated by "," enclosed by '"';

输出结果如下

```
mysql> system more /tmp/emp.txt
"1","z1","aa"
"2","z1","aa"
"3","z1","aa"
```

 

 

**例子2，发现例1中第一列是数值型，如果不希望字段两边用引号，则语句改为如下**

mysql> select * from emp into outfile '/tmp/emp.txt' fields terminated by ","  optionally enclosed by '"' ;

 结果输出如下

```
mysql> system more /tmp/emp.txt
1,"z1","aa"
2,"z1","aa"
3,"z1","aa"
```

 

下面测试一下转义字符，大概包括三类，转义字符本身，字段分隔符（导出的文本中用什么符号分隔），记录分隔符（每条记录之间用什么分隔，默认是回车）

**例子3，更改上面例子中id=1的name为`\"##!aa`**

[![复制代码](MySQL.assets/copycode.gif)](javascript:void(0);)

```
update employee  set name ='\\"##!aa' where id=1;   #更新操作中转义字符本身也需要用转义字符来转义，所以这里有2个\\
# 然后做导出操作
select * from employee into outfile '/tmp/employee' fields terminated by "," optionally enclosed by ‘”’；

导出结果如下
mysql> system more /tmp/emp.txt
1,"\\\"##!aa","aa"
2,"z1","aa"
3,"z1","aa"
```

[![复制代码](MySQL.assets/copycode.gif)](javascript:void(0);)

说明：name 中含有转义字符本身“\”，域引用符“”“，因此，在输出的数据中我们发现这两种字符前面都加上了转义字符“\”

 

**例子4，将id=1的name更新为含有字段分隔符”，“的字符串**

[![复制代码](MySQL.assets/copycode.gif)](javascript:void(0);)

```
update employee set name='\\"#,#,!aa' where id=1;

然后导出
mysql> system rm /tmp/emp.txt  #需要删掉重名文件

mysql> select * from emp into outfile '/tmp/emp.txt' fields terminated by "," optionally enclosed by '"' ;

输出结果如下
mysql> system more /tmp/emp.txt
1,"\\\"#,#,!aa","aa"
2,"z1","aa"
3,"z1","aa"
```

[![复制代码](MySQL.assets/copycode.gif)](javascript:void(0);)

说明：发现数据中的字符","没有被转义，原因是它和后面真正的字段分隔符之间没有冲突，因为name字段包含在双引号之间。

 

**例子5：继续做测试，将输出文件的字段引用符去掉，这个时候，我们的预期是数据中的“,”将成为转义字符而需要加上“\”**

[![复制代码](MySQL.assets/copycode.gif)](javascript:void(0);)

```
mysql> system rm /tmp/emp.txt
mysql> select * from emp into outfile '/tmp/emp.txt' fields terminated by "," ;

输出结果如下

mysql> system more /tmp/emp.txt
1,\\"#\,#\,!aa,aa
2,z1,aa
3,z1,aa
```

[![复制代码](MySQL.assets/copycode.gif)](javascript:void(0);)

说明：果然，现在的“,”前面加上了转义字符“\”。而刚才的引用符“””却没有被转义，因为它已经没有什么歧义，不需要被转义

 

注意：SELECT…INTO OUTFILE...产生的输出文件如果在目标目录下有重名文件，将不会创建成功，源文件不能被自动覆盖

 

 

#### 2. mysqldump 导出数据为文本

mysqldump –u username –T target_dir dbname tablename [option]

其中 option 参数可以是以下选项：

1) --fields-terminated-by=name（字段分隔符）；
2) --fields-enclosed-by=name 字段引用符，比如每个字段用双引号括起来；
3) --fields-optionally-enclosed-by=name（字段引用符，只用在 char、varchar 和 text 等字符型字段上
4) --fields-escaped-by=name（转义字符）；
5) --lines-terminated-by=name（记录结束符）。

 

常用参数

 --compact ：使得输出结果简洁，不包括默认选项中的各种注释，例如例中对 test 数据库中的表 emp 进行简洁导出 mysqldump -uroot -p  --compact test emp >a

 -F --flush-logs（备份前刷新日志）：加上此选项后，备份前将关闭旧日志，生成新日志。使得进行恢复的时候直接从新日志开始进行重做，大大方便了恢复过程

 -l --lock-tables（给所有表加读锁）：可以在备份期间使用，使得数据无法被更新，从而使备份的数据保持一致性，可以配合-F选项一起使用。

 

注意：

MyISAM 存储引擎在备份的时候需要加上-l 参数，表示将所有表加上读锁，在备份期间，所有表将只能读而不能进行数据更新。

但是对于事务存储引擎（InnoDB 和 BDB）来说，可以采用更好的选项--single-transaction，此选项将使得 InnoDB 存储引擎得到一个快照（Snapshot），使得备份的数据能够保证一致性

 

例子，采用 mysqldump 生成指定分隔符分隔的文本：

mysqldump -uroot -T /tmp test emp --fields-terminated-by ',' --fields-optionally-enclosed-by '"'

输出结果如下

more /tmp/emp.txt

1,"\\\"#,#,!aa","aa"
2,"z1","aa"
3,"z1","aa"

注意：

1）ubuntu测试中会出错，解决方法如下。将输出目录改为/var/lib/mysql-files/. 例子中test是库名，emp是表名，如果不写则备份全部表

[![复制代码](MySQL.assets/copycode.gif)](javascript:void(0);)

```
mysql> mysql> select * from person into outfile '/tmp/x.txt' fields terminated by ",";
ERROR 1290 (HY000): The MySQL server is running with the --secure-file-priv option so it cannot execute this statement
mysql> show global variables like "%secure%";
+--------------------------+-----------------------+
| Variable_name            | Value                 |
+--------------------------+-----------------------+
| require_secure_transport | OFF                   |
| secure_auth              | ON                    |
| secure_file_priv         | /var/lib/mysql-files/ |
+--------------------------+-----------------------+
```

[![复制代码](MySQL.assets/copycode.gif)](javascript:void(0);)

2）如果非要用/tmp，则需要做如下操作 (测试了下又不行了...)

```
在/etc/apparmor.d/usr.sbin.mysqld文件中加入一行代码如下
/data/export/** rw, 保存退出后
/etc/init.d/apparmor reload  #重新载入配置 
再次导出即可。
```

 

### **三. 导入纯文本数据**

> 这里只讨论用 SELECT… INTO OUTFILE 或者 mysqldump 导出的纯数据文本的导入方法。和导出类似，导入也有两种不同的方法，分别是 LOAD DATA INFILE…和mysqlimport，它们的本质是一样的，区别只是在于一个在 MySQL 内部执行，另一个在 MySQL 外部执行。

需要注意的是在导入纯文本数据之前，需要事先创建好表结构

#### 1. 一个简单的完整例子

首先准备表结构和文本数据，新建test库：create database test，然后在test下新建表person

[![复制代码](MySQL.assets/copycode.gif)](javascript:void(0);)

```sql
create table person(  
id int not null auto_increment,  
name varchar(40) not null,  
city varchar(20),  
salary int,  
primary key(id)  
)
engine=innodb default charset=utf8; 
```

[![复制代码](MySQL.assets/copycode.gif)](javascript:void(0);)

```
张三　　北京　　3000 
李四　　杭州　　4000 
王五　　/N　　4500 
小明　　天津　　/N
```

每一项之间用Tab键进行分隔，如果该字段为NULL，则用/N表示。

 

进入mysql并且切换到对应的库后，执行下面命令导入数据：

```sql
load data local infile  "/var/lib/mysql/test/data.txt"  into table person(name,city,salary); 
```

 

注意

1. data.txt存放的位置，其他地方会报错

2. 在mysql配置文件中修改字符编码

[![复制代码](MySQL.assets/copycode.gif)](javascript:void(0);)

```
mysql 5.6
sudo vim /etc/mysql/my.cnf，重启mysql来完成字符集的设置。添加以下两行 （如果是上传安装包，vim /etc/my.cnf)
[mysqld]
character_set_server=utf8
binlog_format=row

mysql 5.7 可能需要在etc/mysql/mysql.conf.d/mysqld.cnf中额外增加如下代码，有时候也不需要
[client]
default-character-set=utf8
```

[![复制代码](MySQL.assets/copycode.gif)](javascript:void(0);)



#### 2. 使用“LOAD DATA INFILE…”命令导入数据详解

mysql > LOAD DATA [LOCAL] INFILE ‘filename’ INTO TABLE tablename [option]

1. FIELDS TERMINATED BY 'string'（字段分隔符，默认为制表符'\t'）；
2. FIELDS [OPTIONALLY] ENCLOSED BY 'char'   字段引用符，如果加 OPTIONALLY 选项则只会做用在char, varchar和text等字符型字段上，其他类型字段默认不使用引用符
3. FIELDS ESCAPED BY 'char'（转义字符，默认为'\'）；
4. LINES STARTING BY 'string'（每行前都加此字符串，默认''）；
5. LINES TERMINATED BY 'string'（行结束符，默认为'\n'）；
6. IGNORE number LINES（忽略输入文件中的前 n 行数据）；
7. (col_name_or_user_var,...) （按照列出的字段顺序和字段数量加载数据）；
8. SET col_name = expr,... 将列做一定的数值转换后再加载。

**注意：**

1.  其中 char 表示此符号只能是单个字符，string 表示可以是字符串。
2.  FILELD 和 LINES 和前面 SELECT …INTO OUTFILE…的含义完全相同，不同的是多了几个不同的选项

 

**例子1**：将文件“/tmp/emp.txt”中的数据加载到表 emp 中

mysql> load data infile '/tmp/emp.txt' into table emp fields terminated by ',' enclosed by*'"' ;*

 

**例子2**：如果不希望加载文件中的前2行，加上ignore字段

mysql> load data infile '/tmp/emp.txt' into table emp fields terminated by ','  enclosed by '"'  ignore 2 lines;

 

**例子3**：如果发现文件中的列顺序和表中的列顺序不符，或者只想加载部分列，可以在命令行中加上列的顺序

mysql> load data infile '/tmp/emp.txt' into table emp fields terminated by ',' enclosed by '"' ignore 2 lines (id,content,name);

如果只想加载第一列，字段的列表里面可以只加第一列的名称：

mysql> load data infile '/tmp/emp.txt' into table emp fields terminated by ',' enclosed by '"'  ignore 2 lines (id);

 

**例子4**：如果希望将 id 列的内容+10 后再加载到表中，可以如下操作：

mysql> load data infile '/tmp/emp.txt' into table emp fields terminated by ','  enclosed by '"'  set id=id+10;

 

#### 3. 用 mysqlimport 来实现导入数据详解，具体命令如下

shell>mysqlimport –u root –p*** [--LOCAL] dbname order_tab.txt [option]
其中 option 参数可以是以下选项：

1. --fields-terminated-by=name（字段分隔符）；
2. --fields-enclosed-by=name（字段引用符）；
3. --fields-optionally-enclosed-by=name（字段引用符，只用在 char、varchar 和 text 等字符型字段上
4. --fields-escaped-by=name（转义字符）；
5. --lines-terminated-by=name（记录结束符）；
6. -- ignore-lines=number（或略前几行）。

这与 mysqldump 的选项几乎完全相同，这里不再详细介绍，简单来看一个例子：

mysqlimport -uroot test /tmp/emp.txt --fields-terminated-by=',' --fields-enclosed-by='"'