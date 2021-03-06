---
title: mysql技巧
tags: mysql
date: 2018-12-06 18:13:55
---

# 查找一个表中存在而另一个表中不存在的记录
```
select j.id,j.job_name,c.member_name as company_id,j.job_address,j.salary,j.flush_time
from hr_job j
left join HR_COMPANY c on c.id = j.company_id
left join hr_job_apply ja on ja.job_id = j.id
where not exists (select ja.id from HR_JOB_APPLY where ja.job_id = j.id)

select * from t1
where id not in(select id from t2)
```
# 时间相关 
```
获取今天到明天的数据
where date_format(create_date,'%y-%m-%d') 
BETWEEN date_format(curdate(),'%y-%m-%d') 
and DATE_SUB(curdate(),INTERVAL -1 DAY)

当天数据1
where date_format(create_date,'%y-%m-%d') = date_format(curdate(),'%y-%m-%d')

当天数据2
select * from 表名 where to_days(时间字段名) = to_days(now());
注：to_days 是mysql的函数

昨天数据
SELECT * FROM 表名 WHERE TO_DAYS( NOW( ) ) - TO_DAYS( 时间字段名) <= 1

近7天
SELECT * FROM 表名 where DATE_SUB(CURDATE(), INTERVAL 7 DAY) <= date(时间字段名)

近30天
SELECT * FROM 表名 where DATE_SUB(CURDATE(), INTERVAL 30 DAY) <= date(时间字段名)

本月
SELECT * FROM 表名 WHERE DATE_FORMAT( 时间字段名, '%Y%m' ) = DATE_FORMAT( CURDATE( ) , '%Y%m' )

上一月
SELECT * FROM 表名 WHERE PERIOD_DIFF( date_format( now( ) , '%Y%m' ) , date_format( 时间字段名, '%Y%m' ) ) =1

查询本季度数据
select * from `ht_invoice_information` where QUARTER(create_date)=QUARTER(now());

查询上季度数据
select * from `ht_invoice_information` where QUARTER(create_date)=QUARTER(DATE_SUB(now(),interval 1 QUARTER));

查询本年数据
select * from `ht_invoice_information` where YEAR(create_date)=YEAR(NOW());

查询上年数据
select * from `ht_invoice_information` where year(create_date)=year(date_sub(now(),interval 1 year));

查询当前这周的数据
SELECT name,submittime FROM enterprise WHERE YEARWEEK(date_format(submittime,'%Y-%m-%d')) = YEARWEEK(now());

查询上周的数据
SELECT name,submittime FROM enterprise WHERE YEARWEEK(date_format(submittime,'%Y-%m-%d')) = YEARWEEK(now())-1;

查询上个月的数据
select name,submittime from enterprise where date_format(submittime,'%Y-%m')=date_format(DATE_SUB(curdate(), INTERVAL 1 MONTH),'%Y-%m')

select * from user where DATE_FORMAT(pudate,'%Y%m') = DATE_FORMAT(CURDATE(),'%Y%m') ; 

select * from user where WEEKOFYEAR(FROM_UNIXTIME(pudate,'%y-%m-%d')) = WEEKOFYEAR(now()) 

select * from user where MONTH(FROM_UNIXTIME(pudate,'%y-%m-%d')) = MONTH(now()) 

select * from user where YEAR(FROM_UNIXTIME(pudate,'%y-%m-%d')) = YEAR(now()) and MONTH(FROM_UNIXTIME(pudate,'%y-%m-%d')) = MONTH(now()) 

select * from user where pudate between  上月最后一天  and 下月第一天

查询当前月份的数据 
select name,submittime from enterprise   where date_format(submittime,'%Y-%m')=date_format(now(),'%Y-%m')

查询距离当前现在6个月的数据
select name,submittime from enterprise where submittime between date_sub(now(),interval 6 month) and now();
```

# 几种case-when的形式
```
CASE b.check_type
when '1' then b.real_name
when '2' then b.association_name
    else u2.name(有else)
end AS "user.name",
```

```
CASE a.exh_type
when '1' then a.is_art
when '2' then c.is_art
end AS "isArt"
```

```
case (case后边没东西)
when a.end_time > now() then '1'
when a.end_time &lt; now() then '2'
end AS "isPass"
```

```
where a.is_home = '1'
and a.publish_status = '1'
and (
case a.exh_type
    when '1' then a.audit_status = '2'
        else 1 = 1
        end
)
```
	
# rand()
order by RAND()：随机返回结果，随机函数效率很低，尽量不要使用，因为它的效率会随着数据量的增大，效率越来越低。

推荐一种：
```sql
SELECT
	* 
FROM
	`question` AS t1
	JOIN (
	SELECT
		ROUND(
			RAND( ) * ( ( SELECT MAX( id ) FROM `question` ) 
			- ( SELECT MIN( id ) FROM `question` ) ) 
			+ ( SELECT MIN( id ) FROM `question` ) 
		) AS id 
	) AS t2 
WHERE
	t1.id >= t2.id 
ORDER BY
	t1.id 
	LIMIT 5;
```
加上MIN(id)的判断，是因为最开始测试的时候，因为没有加上MIN(id)的判断，结果有一半的时间总是查询到表中的前面几行。这种方法缺点是会产生id连续的数据。

# 并发插入、存在不插入、存在更新操作
1. 通常会，先查询，不存在再插入，但是这样无法避免并发问题，不用考虑了。

2. 使用`replace into`，存在即更新，不存在即插入
`replace into table (id,name) values(1,"小明")`
如果发现表中已经有此行数据（根据主键或者唯一索引判断），则先删除此行数据，然后插入新的数据，否则，直接插入新数据。所以，插入时如果不插入主键(逐渐默认递增)，就需要保证一列是唯一索引，保证唯一，才可以。

3. 存在不插入，不存在再插入(感觉这种方法是最好的)
mysql 不能在插入时带where条件，故用临时表处理
```
INSERT INTO demo_in(a,b,c) 
SELECT 123, 2, 4 FROM DUAL 
WHERE NOT EXISTS(SELECT c FROM demo_in WHERE c = 4);
```
用临时表DUAL来标记数据，然后插入到demo_in表中。
条件是c=4，并且not exists，也就是当c=4条件满足，则不插入。

# union(union all)和order by
- 每一个子句可以使用()包围，但是当第一个子句使用了()时，其它的子句都必须使用括号包围。
- 每一个字句可以包含WHERE，GROUP BY，HAVING，JOIN，LIMIT，但是不能使用ORDER BY，如果需要使用ORDER BY，必须使用()包围子句。

# 给查询记录添加自然序号
表a，数据：

|id|name|
|-|-|
|5|张三|
|6|李四|
|7|王五|

通过一条sql获取这三条记录，并且给每行前面加上从1开始的序号

|序号|id|name|
|-|-|-|
|1|5|张三|
|2|6|李四|
|3|7|王五|
```sql
SELECT
	@rownum := @rownum + 1 AS rownum,
	a.* 
FROM
( SELECT @rownum := 0 ) r,
table a;
```

写法2：
```sql
SELECT
	@rowno := @rowno + 1 AS rowno,
	a.* 
FROM
	table a,
	( SELECT @rowno := 0 ) t
```

如果有按照某个字段排序，行号会不规则排列，换成先排序，外层加上行号会更加合适。
```sql
SELECT
	@ROWNO := @ROWNO + 1 AS ROWNO,
	T.* 
FROM
	(
	SELECT
		a.*
	FROM
		table_name a
		LEFT JOIN ...
	WHERE
		...
	ORDER BY
		... desc
	) T,( SELECT @ROWNO := 0 ) T3
ORDER BY
	ROWNO
```


# MySQL单表多次查询和多表联合查询，哪个效率高？
很多高性能的应用都会对关联查询进行分解。
简单地，可以对每个表进行一次单表查询，然后将结果在应用程序中进行关联。例如，下面这个查询：
```
select * from tableA a
join tableB b on b.tableA_id = a.id
join tableC c on c.tableB_id = b.id
where a.name = '123';
```

可以分解成下面这些查询来代替：
```
select id from tableA where name = '123'; -- 结果A
select * from tableB where tableA_id = id(结果A) -- 结果B
select * from tableC where id in (结果B)
```

到底为什么要这样做？
起初感觉，这样做并没有什么好处，原本一条查询，这里却变成了多条查询，返回结果又是一模一样。
事实上，用分解关联查询的方式重构查询具有如下优势：
- 让缓存效率更高
对于MySQL的查询缓存来说，如果关联中的某个表发生了变化，那么就无法使用查询缓存了，而拆分后，如果某个表很少改变，那么基于该表的查询就可以重复利用查询缓存结果了。
- 可以减少冗余记录的查询。
- 查询本身效率也可能会有所提升
- 将查询分解后，执行单个查询可以减少锁的竞争。
- 在应用层做关联，可以更容易对数据库进行拆分，更容易做到高性能和可扩展。
- 这样做相当于在应用中实现了哈希关联，而不是使用MySQL的嵌套环关联，某些场景哈希关联的效率更高很多。

# mysql存储表情
存储表情时报错：
`java.sql.SQLException: Incorrect string value: '\xF0\x9F\x92\x94' for colum n 'name' at row 1 `
使用mysql数据库的时候，如果字符集是UTF-8，当存储emoji表情的时候，会抛出以上异常。
这是由于字符集不支持的异常，因为utf-8编码有可能是两个，三个，四个字节，其中Emoji表情是四个字节，而mysql的utf-8编码最多三个字节，所以导致数据插不进去。
解决：

1. 修改database,table,column字符集
```
ALTER DATABASE database_name CHARACTER SET = utf8mb4 COLLATE = utf8mb4_unicode_ci;
ALTER TABLE table_name CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
ALTER TABLE table_name CHANGE column_name VARCHAR(191) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```
除了column的字符集，库和表的字符集不是必须修改。

2. 修改mysql配置文件
```
[client]
default-character-set = utf8mb4
[mysql]
default-character-set = utf8mb4
[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
init_connect='SET NAMES utf8mb4'
```

3. 检查`mysql-connector-java`版本，必须高于5.1.13否则仍然不能试用utf8mb4
4. 修改链接数据库地址
```
jdbc:mysql://localhost:3306/test?
"useUnicode=true" +
"&characterEncoding=UTF-8" +
"&zeroDateTimeBehavior=convertToNull" +
"&useSSL=false" +
"&allowMultiQueries=true" +
"&autoReconnect=true" +
"&rewriteBatchedStatements=TRUE");
```
如果升级了mysql-connector，其中的characterEncoding=utf8可以自动被识别为utf8mb4（兼容原来的utf8），
而autoReconnection（当数据库连接异常中断时，是否自动重新连接？默认为false）强烈建议配上，忽略这个属性，
可能导致缓存缘故 ，没有读取到DB最新的配置，导致一直无法试用utf8mb4字符集。

# mysql排序错误
检查排序的字段是否是int类型，如果是varchar，即使存的是数字，排序也是错误的。
这时最好把字段改成int类型，如果非要对varchar里存的数字字段排序，可以`order column+0`

最好不要对汉字进行排序，如果非要对汉字排序，只要进行简单转换即可排序了。
在mysql中使用order by对存储了中文信息的字段，默认出来的结果并不是按汉字拼音的顺序来排序，要想按汉字的拼音来排序，需要把数据库的字符集设置为UTF8，然后在order by 时候强制把该字段信息转换成GBK，这样出来的结果就是按拼音顺序排序的。例如：
`SELECT * FROM table_name ORDER BY CONVERT(column_name USING gbk);`

查询的时候，通过convert函数，把查询出来的数据使用的字符集gb2312编码就可以了，然后使用convert之后的中文排序。
但是如果真的去把表中字段的字符集改成gb2312，又会涉及到很多编码的问题，页面传值啊，从数据库中存取啊，很麻烦。
只要在查询的时候，指定一下字符集，并不是真的把物理字段改成gb2312，很简单。

# mysql进行树状所有子节点的查询
在Oracle 中我们知道有一个 Hierarchical Queries 通过CONNECT BY 我们可以方便的查了所有当前节点下的所有子节点。但很遗憾，在MySQL的目前版本中还没有对应的功能。
在MySQL中如果是有限的层次，比如我们事先如果可以确定这个树的最大深度是4, 那么所有节点为根的树的深度均不会超过4，则我们可以直接通过left join 来实现。
但很多时候我们无法控制树的深度。这时就需要在MySQL中用存储过程来实现或在你的程序中来实现这个递归。本文讨论一下几种实现的方法。

## 样例数据
```
create table treeNodes
	(
	id int primary key,
	nodename varchar(20),
	pid int
	);

select * from treenodes;
+----+----------+------+
| id | nodename | pid  |
+----+----------+------+
|  1 | A        |    0 |
|  2 | B        |    1 |
|  3 | C        |    1 |
|  4 | D        |    2 |
|  5 | E        |    2 |
|  6 | F        |    3 |
|  7 | G        |    6 |
|  8 | H        |    0 |
|  9 | I        |    8 |
| 10 | J        |    8 |
| 11 | K        |    8 |
| 12 | L        |    9 |
| 13 | M        |    9 |
| 14 | N        |   12 |
| 15 | O        |   12 |
| 16 | P        |   15 |
| 17 | Q        |   15 |
+----+----------+------+	
```
树形图如下：
```
1:A
  +-- 2:B
  |    +-- 4:D
  |    +-- 5:E
  +-- 3:C
       +-- 6:F
            +-- 7:G
 8:H
  +-- 9:I
  |    +-- 12:L
  |    |    +--14:N
  |    |    +--15:O
  |    |        +--16:P
  |    |        +--17:Q
  |    +-- 13:M
  +-- 10:J
  +-- 11:K
```

## 方法一：利用函数来得到所有子节点号
创建一个function getChildLst, 得到一个由所有子节点号组成的字符串。
```sql
CREATE FUNCTION `getChildLst` ( rootId INT ) RETURNS VARCHAR(1000) BEGIN
DECLARE sTemp VARCHAR ( 1000 );
DECLARE sTempChd VARCHAR ( 1000 );        
SET sTemp = '$';
SET sTempChd = cast( rootId AS CHAR );	       
WHILE
	sTempChd IS NOT NULL DO
	SET sTemp = concat( sTemp, ',', sTempChd );
	SELECT
	group_concat( id ) INTO sTempChd 
		FROM
	treeNodes 
		WHERE
	FIND_IN_SET( pid, sTempChd ) > 0;        
	END WHILE;
	RETURN sTemp;
END
```
使用我们直接利用find_in_set函数配合这个getChildlst来查找
```sql
select getChildLst(1);
+-----------------+
| getChildLst(1)  |
+-----------------+
| $,1,2,3,4,5,6,7 |
+-----------------+

select * from treeNodes where FIND_IN_SET(id, getChildLst(1));
+----+----------+------+
| id | nodename | pid  |
+----+----------+------+
|  1 | A        |    0 |
|  2 | B        |    1 |
|  3 | C        |    1 |
|  4 | D        |    2 |
|  5 | E        |    2 |
|  6 | F        |    3 |
|  7 | G        |    6 |
+----+----------+------+

select * from treeNodes where FIND_IN_SET(id, getChildLst(3));
+----+----------+------+
| id | nodename | pid  |
+----+----------+------+
|  3 | C        |    1 |
|  6 | F        |    3 |
|  7 | G        |    6 |
+----+----------+------+
```
优点: 简单，方便，没有递归调用层次深度的限制 (max_sp_recursion_depth,最大255) ；
缺点：长度受限，虽然可以扩大 RETURNS varchar(1000)，但总是有最大限制的。
MySQL目前版本( 5.1.33-community)中还不支持function 的递归调用。

## 方法二：利用临时表和过程递归
创建存储过程如下。createChildLst 为递归过程，showChildLst为调用入口过程，准备临时表及初始化。


入口过程
```sql
CREATE PROCEDURE showChildLst ( IN rootId INT ) BEGIN
	CREATE TEMPORARY TABLE
	IF
		NOT EXISTS tmpLst ( sno INT PRIMARY KEY auto_increment, id INT, depth INT );
	DELETE 
	FROM
		tmpLst;
	CALL createChildLst ( rootId, 0 );
	    SELECT
	tmpLst.*,
	treeNodes.* 
	FROM
		tmpLst,
		treeNodes 
	WHERE
		tmpLst.id = treeNodes.id 
	ORDER BY
	tmpLst.sno;
END;
```

递归过程
```sql
CREATE PROCEDURE createChildLst ( IN rootId INT, IN nDepth INT ) BEGIN
	DECLARE
		done INT DEFAULT 0;
	DECLARE
		b INT;
	DECLARE
		cur1 CURSOR FOR SELECT
		id 
	FROM
		treeNodes 
	WHERE
		pid = rootId;
	DECLARE
		CONTINUE HANDLER FOR NOT FOUND 
		SET done = 1;
	INSERT INTO tmpLst
	VALUES
		( NULL, rootId, nDepth );
	OPEN cur1;
	FETCH cur1 INTO b;
	WHILE
			done = 0 DO
			CALL createChildLst ( b, nDepth + 1 );
		FETCH cur1 INTO b;
	END WHILE;
CLOSE cur1;
END;
```

调用时传入结点
```
call showChildLst(1);
+-----+------+-------+----+----------+------+
| sno | id   | depth | id | nodename | pid  |
+-----+------+-------+----+----------+------+
|   4 |    1 |     0 |  1 | A        |    0 |
|   5 |    2 |     1 |  2 | B        |    1 |
|   6 |    4 |     2 |  4 | D        |    2 |
|   7 |    5 |     2 |  5 | E        |    2 |
|   8 |    3 |     1 |  3 | C        |    1 |
|   9 |    6 |     2 |  6 | F        |    3 |
|  10 |    7 |     3 |  7 | G        |    6 |
+-----+------+-------+----+----------+------+

call showChildLst(3);
+-----+------+-------+----+----------+------+
| sno | id   | depth | id | nodename | pid  |
+-----+------+-------+----+----------+------+
|   1 |    3 |     0 |  3 | C        |    1 |
|   2 |    6 |     1 |  6 | F        |    3 |
|   3 |    7 |     2 |  7 | G        |    6 |
+-----+------+-------+----+----------+------+
```

depth 为深度，这样可以在程序进行一些显示上的格式化处理。类似于oracle中的 level 伪列。sno 仅供排序控制。这样你还可以通过临时表tmpLst与数据库中其它表进行联接查询。

MySQL中你可以利用系统参数 max_sp_recursion_depth 来控制递归调用的层数上限。如下例设为12.
`set max_sp_recursion_depth=12;`

优点：可以更灵活处理，及层数的显示。并且可以按照树的遍历顺序得到结果。
缺点：递归有255的限制

## 方法三：利用中间表和过程
创建存储过程如下。由于MySQL中不允许在同一语句中对临时表多次引用，只以使用普通表tmpLst来实现了。当然你的程序中负责在用完后清除这个表。
```sql
drop PROCEDURE IF EXISTS  showTreeNodes;
CREATE PROCEDURE showTreeNodes (IN rootid INT)
BEGIN
DECLARE Level int;
drop TABLE IF EXISTS tmpLst;
CREATE TABLE tmpLst (
 id int,
 nLevel int,
 sCort varchar(8000)
);
 
Set Level = 0;
INSERT into tmpLst SELECT id,Level,ID FROM category WHERE parent_id = rootid;
WHILE ROW_COUNT() > 0
DO SET Level = Level + 1;
 INSERT into tmpLst
  SELECT A.ID,Level,concat(B.sCort,A.ID) FROM category A,tmpLst B
   WHERE A.parent_id = B.ID AND B.nLevel = Level-1;
END WHILE;
 
END;
CALL showTreeNodes(0);
```
执行完后会产生一个tmpLst表，nLevel 为节点深度，sCort 为排序字段。
使用方法
```
SELECT concat(SPACE(B.nLevel*2),'+--',A.nodename) FROM treeNodes A,tmpLst B WHERE A.ID=B.ID ORDER BY B.sCort;
+--------------------------------------------+
| concat(SPACE(B.nLevel*2),'+--',A.nodename) |
+--------------------------------------------+
| +--A                                       |
|   +--B                                     |
|     +--D                                   |
|     +--E                                   |
|   +--C                                     |
|     +--F                                   |
|       +--G                                 |
| +--H                                       |
|   +--J                                     |
|   +--K                                     |
|   +--I                                     |
|     +--L                                   |
|       +--N                                 |
|       +--O                                 |
|         +--P                               |
|         +--Q                               |
|     +--M                                   |
+--------------------------------------------+
```

优点：层数的显示。并且可以按照树的遍历顺序得到结果。没有递归限制。
缺点：MySQL中对临时表的限制，只能使用普通表，需做事后清理。

[参考地址](https://blog.csdn.net/ACMAIN_CHM/article/details/4142971)

## 新的思路
解决这种树状结构问题，可以在表中加一个字段`path`，标识这条数据的路径，即父子级结构路径。

|id|公司名|parent_id|path|
|-|-|-|-|
|1|总公司|0|/1/|
|2|技术部|1|/1/2/|
|3|人力部|1|/1/3/|
|4|技术部下-java部门|2|/1/2/4/|

这样如果我想查总公司下有哪些部门，只需`like id = %1`即可。

# 一些特殊的sql
- 1
```sql
<!-- 案件列表 -->
<select id="caseList" resultType="map">
select
	id,
	concat(
	<foreach collection="list" separator="," item="i">
		${i.columnEnglish}
		<if test="null != i.connector and '' != i.connector">
		  ,${i.connector}
		</if>
	</foreach>
	) dataName
from `${tableName}`
</select>
```
出自：archive档案管理系统，`ArchiveMapper.xml`

- 2
```
select 
	* 
from table_name 
where id in (1,2,3)
order by field(id,1,2,3)
```
根据in里的参数排序。
xml写法：
```
order by field(q.id,
<foreach collection="questionIdList" separator="," item="i">
  #{i}
</foreach>
)
```

# mysql导入data文件夹
昨天服务器被挖矿病毒攻击，导致服务器无法访问，数据库无法连接。
解决过程：
- 在阿里云控制台备份服务器硬盘快照
- 重装操作系统
- 将快照挂载到新系统上，我挂载到了`/mnt`目录下
- 进入/mnt目录，查看my.cnf`vim etc/my.cnf`，发现data文件在`var/lib/mysql`
- 进入目录`cd var/lib/mysql`，将该目录整体打包。
- 重新安装mysql，将新装mysql的`var/lib/mysql`删除，将刚才打包的data目录复制到`/var/lib`
- 解压，然后启动mysql
- 远程连接发现mysql无法访问，提示`table xxxxx doesn't exist`
- 在服务器任意位置输入`mysql_upgrade -uroot -p --force`，然后提示：
```
Upgrade process completed successfully.
Checking if update is needed.
```
- 再次尝试远程连接，成功，数据都在！

# 导入`.frm和.ibd`文件
`.frm`和`.ibd`分别对应数据库中的表名和表中的数据。
我们进入到mysql的data文件夹，找到一个库进去，可以证实我的想法。

# mysql账户
添加用户
`create user zhangsan identified by 'myPass';`
上面的命令创建了用户zhangsan，密码是myPass。在mysql.user表里可以查看到新增用户的信息。
`select * from mysql.user`

授权
`grant privilegesCode on dbName.tableName to username@host identified by "password";`

示例
`grant all privileges on zhangsanDb.* to zhangsan@'%' identified by 'myPass';`
`flush privileges;`
上面的语句将zhangsanDb数据库的所有操作权限都授权给了用户zhangsan。
在mysql.db表里可以查看到新增数据库权限的信息
`select * from mysql.db where User = 'zhangsan'`

也可以通过`show grants`命令查看权限授予执行的命令：
`show grants for 'zhangsan';`

privilegesCode表示授予的权限类型，常用的有以下几种类型：
`all privileges`：所有权限。
`select`：读取权限。
`delete`：删除权限。
`update`：更新权限。
`create`：创建权限。
`drop`：删除数据库、数据表权限。

dbName.tableName表示授予权限的具体库或表，常用的有以下几种选项：
`.`：授予该数据库服务器所有数据库的权限。
`dbName.*`：授予dbName数据库所有表的权限。
`dbName.dbTable`：授予数据库dbName中dbTable表的权限。
`username@host`：授予的用户以及允许该用户登录的IP地址。其中Host有以下几种类型：

`localhost`：只允许该用户在本地登录，不能远程登录。
`%`：允许在除本机之外的任何一台机器远程登录。
`192.168.1.15`：具体的IP表示只允许该用户从特定IP登录。
`password`：指定该用户登录时的密码。
`flush privileges`：表示刷新权限变更。

[参考](https://www.cnblogs.com/chanshuyi/p/mysql_user_mng.html)

## 修改密码
`update mysql.user set password = password('zhangsannew') where user = 'zhangsan' and host = '%';`
`flush privileges;`

## 删除用户
`drop user zhangsan@'%';`

# select执行顺序
```
(7) SELECT
(8) DISTINCT <select_list>
(1) FROM <left_table>
(3) <join_type> JOIN <right_table>
(2) ON <join_condition>
(4) WHERE <where_condition>
(5) GROUP BY <group_by_list>
(6) HAVING <having_condition>
(9) ORDER BY <order_by_condition>
(10) LIMIT <limit_number>
```
MySQL的SELECT执行顺序一共分为10步，如上所标注的那样，最先执行的是FROM操作，最后执行的是LIMIT操作。
其中每一次操作都会产生一张虚拟的表，这个虚拟的表作为一个处理的输入，只是这些虚拟的表对用户来说是透明的，
但是只有最后一个虚拟的表才会被作为结果返回。如果没有在语句中指定某一个子句，那么将会跳过相应的步骤。

下面我们来具体分析一下查询处理的每一个阶段
1. FORM: 对FROM左边的表和右边的表计算笛卡尔积，产生虚表VT1。
2. ON: 对虚表VT1进行ON过滤，只有那些符合<join-condition>的行才会被记录在虚表VT2中。
3. JOIN： 如果指定了OUTER JOIN（比如left join、 right join），那么保留表中未匹配的行就会作为外部行添加到虚拟表VT2中，产生虚拟表VT3。
4. WHERE： 对虚拟表VT3进行WHERE条件过滤。只有符合<where-condition>的记录才会被插入到虚拟表VT4中。
5. GROUP BY: 根据group by子句中的列，对VT4中的记录进行分组操作，产生VT5。
6. HAVING： 对虚拟表VT5应用having过滤，只有符合<having-condition>的记录才会被 插入到虚拟表VT6中。
7. SELECT： 执行select操作，选择指定的列，插入到虚拟表VT7中。
8. DISTINCT： 对VT7中的记录进行去重。产生虚拟表VT8.
9. ORDER BY: 将虚拟表VT8中的记录按照<order_by_list>进行排序操作，产生虚拟表VT9.
10. LIMIT：取出指定行的记录，产生虚拟表VT10, 并将结果返回。

# group by 获取最新的数据
因为group by 是获取的第一条数据，所以可以通过下面的方法获取最后一条数据。
```
select * from chapter_user_dub where id in(
 select MAX(id) from chapter_user_dub where user_id = 106175 group by user_id
)
```