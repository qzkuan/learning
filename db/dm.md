- 纵表实现方案
- 分页查询dm方式partition by
- 


## 约束
### 聚簇主键与非聚簇主键

**修改联合聚集主键**
```sql
--1. 创建测试表，C1,C3为联合主键，聚集属性
CREATE TABLE "T11"
(
"C1" CHAR(10) NOT NULL,
"C2" CHAR(10),
"C3" CHAR(10) NOT NULL,
 CLUSTER PRIMARY KEY("C1", "C3"))

--2. 创建聚集索引，将主键创建的索引属性变更为非聚集，然后删除新建的索引。此时主键索引已变成非聚集属性
create cluster  index "TS1" on "T11"("C1");
drop index "TS1";

--3. 修改联合主键。主键的约束名constraint "CONS134218806"要根据实际情况更改
alter table "T11" modify constraint "CONS134218806" to primary key ("C1","C2");
--4. 如需要，将主键移除的列变更为可以为空
alter table "T11" alter column "C3" set null;
```



### 锁等待
```sql
-- 查看锁等待的sessionid
select c.* from v$lock a , V$sessions c where a.trx_id =c.trx_id and a.lmode='X';

-- 记录下导致锁的动作的SESS_ID
 sp_close_session(sess_id) 杀死会话
```

 ### 自增主键插值
 ```sql
 SET IDENTITY_INSERT 模式名.表名 ON;
 ```

## 日志
> 归档不是直接可读的，主要是给备份用。sqllog采集的语句是可读的，主要做分析用。sqllog也有设置上限


### 归档日志
- /dmdata/***arch
- 上限4G

### 日志追踪

- /dmdata/sqllog
- 8个文件循环写入


## 常用工具
### disql
> 将屏幕显示的内容输出到指定文件
```
spool /home/user/1.log
spool "/home/user/1.log"
spool off
```
> 切换到操作系统命令host
```
# host可以不退出disql执行操作系统命令，单独执行disql直接从disql界面切换到操作系统，之后使用exit回到disql界面
host ls -lrt /home
host ps -ef |grep -i dmser

exit
```
> 获取对象结构信息describe

> 退出disql
```
exit/quit
```



```sql
cd /home/dmdba/dmdbms/bin
./disql user/pass@host:port
-- 转义的两种方式
./disql guwpadmin/\"guwp@1234\"@host:port
./disql guwpadmin/'"guwp@1234"'@host:port
-- 执行sql语句
./disql user/pass@host:port -e "select sysdate()"
-- 执行sql文件
./disql user/pass@host:port \`/home/user/1.sql

```

## 常用操作
```sql
-- 数据库名称
select cur_database;
select name from v$database;
-- 自增列
select ident_current('schame.table');
ident_seed/ident_incr
-- 数据库版本
select * from v$version;--查看数据库大版本号
select id_code;  --查看数据库详细版本
-- 查看数据库中所有表
select table_name from all_tables wehre owner='user';
-- 建表语句
select to_char( tabledef('schame','table' );
-- 索引信息
select * from ALL_INDEXES where owner='user' and table_name='table';

-- 查询最近10000条SQL历史记录，并按照耗时倒序排列
select * from v$sql_history order by time_used desc;

-- 确定高负载SQL
-- 显示最近1000条执行时间较长的SQL语句
select * from v$long_exec_sqls ORDER BY EXEC_TIME DESC;
-- 显示服务器启动以来执行时间最长的20条SQL语句
select * from v$system_long_exec_sqls ORDER BY EXEC_TIME DESC;



```

## 常用函数

### merge into
1. 按照指定的条件执行插入或更新操作
2. 如果满足条件的行存在，执行更新操作；否则执行插入操作：
– 避免多次重复执行插入和删除操作
– 提高效率而且使用方便
– 在数据仓库应用中经常使用作

```sql
MERGE INTO table_name table_alias
	USING (table|view|sub_query) alias
	ON (join condition)
	WHEN MATCHED THEN
		UPDATE SET
		col1 = col_val1,
		col2 = col2_val
	WHEN NOT MATCHED THEN
		INSERT (column_list)
		VALUES (column_values);
```
- 解释：<br/>
WHEN MATCHED THEN 存在则执行 UPDATE；
WHEN NOT MATCHED THEN 不存在则执行 INSERT
- 举例：
在对表COPY_EMP 使用merge 语句，根据指定的条件从表EMPLOYEES 中插入或更新数据。

```sql
MERGE INTO copy_emp c
	USING employees e
	ON (c.employee_id = e.employee_id)
WHEN MATCHED THEN
	UPDATE SET
	c.first_name = e.first_name,
	c.last_name = e.last_name,
	...
	c.department_id = e.department_id
WHEN NOT MATCHED THEN
	INSERT VALUES(e.employee_id, e.first_name, e.last_name,
				e.email, e.phone_number, e.hire_date, e.job_id,
				e.salary, e.commission_pct, e.manager_id,
				e.department_id);
```

### listagg
> 列值拼接函数
```sql
LISTAGG ( column | expression, delimiter ) WITHIN GROUP (ORDER BY column | expression) OVER (PARTITION BY column | expression)
-- eg 根据年纪排序后列转行
SELECT listagg(name, ','),age WITHIN GROUP (ORDER BY age ) from t_pub_company

```

### locate
> 返回字符串`char1`在`char2`中从位置`n`开始首次出现的位置，如果参数`n`省略或为负数，则从`char2`的最左边开始查找
```sql
locate(char1,char2[,n])
```

### coalesce
> 返回参数中第一个非空的值，如果所有参数均为null，则返回null
```sql
coalesce(n1,n2,...,nx)
```

### concat
> 返回多个字符串顺序联结成一个字符串，等价于`||`
```sql
concat(char1,char2,...)
```

### decode
> 将输入数值与函数中的参数列表相比较，根据输入值返回一个对应值
```sql
DECODE(control_value,value1,result1[,value2,result2…][,default_result]);
   control _value
-- 试图处理的数值。DECODE函数将该数值与后面的一系列的偶序相比较，以决定返回值。
　　value1 ：
-- 是一组成序偶的数值。如果输入数值与之匹配成功，则相应的结果将被返回。对应一个空的返回值，可以使用关键字NULL于之对应
　　result1：
-- 是一组成序偶的结果值。
```
> eg
```sql
select decode( x , 1 , ‘x is 1 ’, 2 , ‘x is 2 ’, ‘others’) from dual
```
- 当x等于1时，则返回‘x is 1’。
- 当x等于2时，则返回‘x is 2’。
- 否则，返回others’。