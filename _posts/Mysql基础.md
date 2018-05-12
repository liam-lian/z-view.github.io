`select database()`查看当前所在的数据库
`desc tablename`查看表的结构
`select version()`查看版本号

不区分大小写，但是建议关键字使用大写，其他使用小写


字段、常量、表达式、函数

可以使用\`\`来区分字段和关键字，select \`name\` from table

`SELECT 100`  查询常量值

起别名？

DISTINCT

`+` 运算符用来计算数值：
1. 100+90=190
2. '100'+90=190
3. 'asd'+100=0    转换失败为0
4. null+100=null  只要有null，结果就是null

```sql
concat()函数字符串拼接
SELECT concat('z',first_name,gender) FROM employees.employees;
```

```sql
ifnull(a,b)当a为null的时候，使用值b,否则使用值a
SELECT concat('z',ifnull(first_name,'zz'),gender) FROM
```

```sql
SELECT isnull(a) from table
判断字段是否为null，null则值为1，非null则值为0
```

模糊查询：  
like、between and、is null 、in

like——`%`任意字符串，`_代表单个字符`
```
SELECT * FROM employees where first_name like 'G%i_';
如何转义？
1. 直接使用\转义
2. 指定转义  like(_$_%) ESCAPE '$' 就是指定了$作为转义符
```

IS NULL/IS
NOT NULL不能使用——NULL不能使用=进行判断

安全等于<=>,可以用来判断null值和非nul值，但是可读性差


DISTINCT——   
1. 一般作用于单个字段
2. 作用于多个字段的时候，会以多个字段联合比对重复，并且DISTINCT关键字必须放在第一个字段上


排序查询？
```sql
SELECT * FROM table order by column (asc|desc)

# 这里是给了多个排序条件，并且按照别名进行了排序
SELECT length(first_name) namelength,first_name ,emp_no FROM employees ORDER BY namelength DESC,emp_no;
```


常见函数


- length(str) 返回字节数
- concat(str,str...) 拼接字符串
- upper()、lower()
- substr() 子串，注意索引从1开始
- instr('abcd','bc')  得到值2
- trim() 默认是去掉首尾的空格，也可以指定去掉的字符    
`select trim('*' from '***QQ**QQ***')`
- lpad('lz',5,\*) 左填充，结果是`***lz`
- rpad()
- replace()

- round()
- ceil()
- floor()
- turncate(1.9999,1) ==>1.9 小数点后截断的位数
- mod()

- NOW()
- CURDATE()
- CURTIME()
- YEAR(NOW())
- MONTH('2018-9-11')
- MONTHNAME(NOW())
- DATEDIFF(date,date)返回两个时间之间的相差的天数(int)


- STR_TO_DATE('2017-1-11','%Y-%m-%d') 具体的格式查阅即可
- DATE_FORMAT(DATE(),'%Y###%m###%d') 日期格式转换成字符串

- version()
- DATABASE()
- USER()


IF语句：
```SQL
SELECT IF(10<5,'大','小');
```

CASE语句：
```SQL
SELECT salary 原始工资,department_id,
CASE department_id
  WHEN 30 THEN salary*1.1
  WHEN 40 THEN salary*1.2
  WHEN 50 THEN salary*1.3
  ELSE salary
END AS 新工资
FROM employees;

SELECT salary,
CASE
  WHEN salary>20000 THEN 'A'
  WHEN salary>15000 THEN 'B'
  WHEN salary>10000 THEN 'C'
  ELSE 'D'
END AS 工资级别
FROM employees;
```

- SUM()
- AVG()
- MIN()
- MAX()
- COUNT()   
这些都是用来统计的函数方法，与他们搭配的函数字段，应该是只有一行的才有意义，我们常常使用group by和这些函数配合使用


分组查询：  
按照某些列进行分组，因此分组查询之后数据是一组一组存在的，因此一般跟上分组的统计函数。而且一般分组查询的条件都要显示出来(相当与一个key  )
```sql
查询每个工种的员工平均工资
SELECT AVG(salary),job_id
FROM employees
GROUP BY job_id;
```

HAVING子句
```SQL
SELECT birth_date,count(*) from employees WHERE gender='M' group by birth_date HAVING count(*) < 10

# 查询出所有的男性的，在同一天生日的人数不超多10人的日子。
# 注意where是针对与整张表首先进行一次筛选
# 筛选之后通过group by进行了分组
# having是真对于每一个分组内部进行判断，一般都是使用分组函数进行判断，如果不满足，该分组就会被直接移除
```

```SQL
INNER [OUTER] JOIN 内连接
# 三个表进行等值内连接,进行了两个JOIN
# 查看没一个部门中的员工人数
SELECT D.dept_no,D.dept_name,count(*)
FROM employees E INNER JOIN dept_emp DE ON E.emp_no=DE.emp_no
INNER JOIN departments D ON DE.dept_no=D.dept_no
GROUP BY D.dept_no
```
内连接就是在某一个条件之上，将两个表连接起来，事实上和
`FROM a,b where a.field=b.field` 是等效的

特殊的情况是自连接，就是把一个表左右连接起来

LEFT [OUTER] JOIN/RIGHT [OUTER] JOIN
得到的连接表的条数和主表的条数是一样的，如果没有匹配的项，就会用null值进行填充

```sql
# 查看没有男朋友的女孩的名字
SELECT girl.name from
girl LEFT JOIN boy
on girl.boyID=boy.id
where boy.id is null
```

MySQl不支持全外连接　

CROSS JOIN　交叉连接　　　
实际上就是得出了两个表的迪卡尔乘积




子查询
可以出现在
-　SELECT后面
-　FROM后面 (后面的结果一般是只能有一列数据的)
-  WHERE后面　(后面的结果一般是只能有一列数据的)
-  HAVING后面
-　exists

```SQL
SELECT last_name,salary
FROM employees
WHERE department_id = (
	SELECT department_id
	FROM employees
	WHERE last_name = 'Zlotkey'
)
```
多行子查询的操作符:IN/NOT IN/SOME/ANY/ALL

```sql
# 查询在部门的location_id为1700的部门工作的员工的员工号
SELECT employee_id
FROM employees
WHERE department_id =ANY(
	SELECT DISTINCT department_id
	FROM departments
	WHERE location_id  = 1700

);
```


exists的语法非常简单，exists(完整的sql查询字句)。得到的结果是１或者０。也就是一个boolean类型的结果。直接放在where/having后面可以直接进行判断。



分页查询。
`limit offset size`其中offset的编号从０开始

UNOIN 联合查询

```sql
＃　通过子查询进行插入
INSERT INTO emplotees (SELECT 字句)
```
```SQL
＃ 将lz的女朋友的电话号码更新一下
＃　在update中使用了连表，直接修改表的内容
UPDATE
boys INNER JOIN girls ON boys.girlID=girls.id
SET girls.num=100
WHERE boys.name='lz'
```

```SQL
＃ 将员工表的所有信息删除
trancate table employees
```
```
# 单表删除
DELETE FROM a WHERE...
＃　多表删除
DELETE a FROM a INNER JOIN b on.. WHERE ..
这样就是删连接结果中表ａ的部分
```



DDL
```sql
# 库
create database t;
create database IF NOT exists t;
ALTER DATABASE t CHARACTER SET GBK
DROP DATABASE IF exists t
＃　表
列名　列类型[(长度)约束]
ALTER TABLE book CHANGE COLUMN bookname bname varchar(20)＃修改列名
ALTER TABLE book MODIFY COLUMN bookname varchar(20)＃更改类型或者约束
ALTER TABLE book ADD COLUMN bookname varchar(20)＃添加一列
ALTER TABLE book DROP COLUMN bookname＃删除一列
ALTER TABLE book RENAME TO books#修改表名
DROP TABLE book ＃删除表　
＃　复制表
CREATE TABLE copy LIKE　book ＃copy和book有一样的结构
CREATE TABLE copy SELECT * FROM book ＃copy和book有一样的结构和数据
```


| 类型 | Header Two     | Header Two     |
| :------------- | :------------- |:------------- |
|Ｔinyint       　| 一共是８位      |Tinyint unsigned表示无符号，下同|
|Smallint      　　| 一共是１6位    ||
|Mediumint      　 |  一共是24位       ||
|int       |  一共是32位     ||
|Bigint       |  一共是64位    ||
|float(M,D)   |  一共是32位    |M表示整个数字的总的位数长度,D表示的是小数部分的长度,如果超出范围将会插入临界值|
|double(M,D)       |  一共是64位    |ＭＤ都可以忽略掉,忽略之后对精度没有任何要求|
|DEC(M,D)       |  一共是64位    |Ｍ默认是10,D默认是0,能够更加精准的表示数字，表示的是精确的数字，尽量使用DEC|
|char(M)       |  固定占用Ｍ个位置,M可省略，默认为１   |占用空间会比较浪费但是效率高一些|
|varvhar(M)       | 最大占用Ｍ个字符，按照实际的数据动态的占用 |节省空间,但是效率低一些，基本上都能用varchar代替char|
|text  |更长的文本 |不能设置默认值|
|ENUM('a','v','z')  |枚举 ||
|SET('a','v','z')  |可以放入的元素是set中元素的字集 ||
|date  |4byte ||
|datetime  |8byte(新版本改成了5byte，可以用datetime代替timestamp) |不受时区影响|
|timestamp  |4byte |受时区影响|
|time  |3byte ||
|year  |1byte ||



- PRIMARY KRY
- NOT NULL
- UNIQUE
- FOREIGN KEY REFERENCES t(thekey)
- DEFAULT
- CHECK

约束分为表级约束和列级约束。列级约束就是直接在列的定义后面添加约束。表级约束是在表的后面加上约束的内容
```sql
CREATE TABELE STU(
    id INT,
    name VARCHCAR(20),
    gender CHAR,
    seat INT,
    majorID INT,

    CONSTRAINT pk PRIMARY KEY(id),
    CONSTRAINT uq UNIQUE(seat),
    CONSTRAINT ck CHECK(gender in ('男','女')),
    CONSTRAINT fk FOREIGN KEY(majorID) REFERENCES major(id)
    ＃CONSTRAINT关键字是给这个约束起一个个名字，不使用也是可以的
)
```
外键依赖的属性必须是所依赖的表的键属性(主键或者UNIQUE都可以)　　　
插入数据的时候，先插入主表，在插入从表
删除数据的时候，先删除从表，在删除主表

```sql
# 使用add添加表级约束
ALTER TABLE STU ADD  CONSTRAINT fk FOREIGN KEY(majorID) REFERENCES major(id)
＃　使用MODIFY添加的是列级约束(实际上就是修改列的属性，顺便就把约束加了)
ALTER TABLE STU MODIFY COLUMN seat INT UNIQUE
```



TCL事务控制语言

只有Innodb存储引擎支持事务

默认事务都是自动提交的。每一条sql执行完毕之后都会自动的提交。　　
如果想要使用显式的事务，需要将首先自动提交禁用。`set autocommit=0`　　　

```
set autocommit=0;
start transation;#可选
事务中的sql语句
commit;提交事务
rollback;回滚
```

MYSQL默认的隔离级别是repetiable read

脏读: 事务Ａ修改的数据在没有提交的情况下就被事务Ｂ给读取到了。
不可重复读:　事务Ａ在同一个事务里面两个查询同一行的内容发现不一样。
repeatable read 确一个事务可以多次从同一个字段读取到完全相同的值
幻读:事务Ａ在同一个事务里面进行两个一样的查询操作，发现两次查询的结果数量不一样。
serializable 确保事务可以从表中读取相同的行，假如事务Ａ查询的表ａ,那么其他事务对于表ａ的增删改都会阻塞，直到事务Ａ提交为止。

```
SAVEPOINT a;设置一个保存点，起名字为ａ
ROLLBACK TO a;回滚到ａ保存点，而不会回滚到事务开始的位置
```

视图:
```sql
CREATE VIEW myview AS SELECT...
CREATE OR REPLACE VIEW myView AS ...
ALTER VIEW myView AS ...
DROP VIEW view1,view2...
DESC myview
视图的增删改对于原表也是起作用的
```
truncate 删除表数据是不支持rollback的，但是ｄelete是可以进行回滚的
　

主键、外键、UNIQUE 会默认创建索引


变量
- 系统变量
     -　全局变量　（针对于所有的会都是有效的，但是服务器重启之后就是无效的了）
     -  会话变量（仅仅真对于当前的会话）
- 自定义变量
　　　- 用户变量
　　　- 局部变量
```sql
SHOW GLOBAL/SESSION(默认) VARIABLES;
SHOW GLOBAL/SESSION(默认) VARIABLES　like ‘％char％’
SELECT @@global/session(默认).系统变量名

SET GLOBAL/SESSION 系统变量名=值
SET @@global/session(默认).系统变量名=值
```

自定义变量
```SQL
SET @变量名=值;
SET @变量名:=值;
SELECT @变量名:=值;

SELECT @变量名;
```
```sql
DECLARE 变量名 类型;
DECLARE 变量名 类型 【DEFAULT 值】;

SET 局部变量名=值;
SET 局部变量名:=值;
SELECT 局部变量名:=值;

SELECT 局部变量名;
```

存储过程
```SQL
#创建语法
CREATE PROCEDURE 存储过程名(参数列表)
BEGIN
	存储过程体（一组合法的SQL语句）
END

#注意：
1、参数列表包含三部分
参数模式  参数名  参数类型
举例：　in stuname varchar(20)
参数模式：
in：该参数可以作为输入，也就是该参数需要调用方传入值
out：该参数可以作为输出，也就是该参数可以作为返回值
inout：该参数既可以作为输入又可以作为输出，也就是该参数既需要传入值，又可以返回值


存储过程体中的每条sql语句的结尾要求必须加分号。
存储过程的结尾可以使用 delimiter 重新设置
语法： delimiter 结束标记

#调用语法
CALL 存储过程名(实参列表);


#案例：创建存储过程实现，用户是否登录成功

CREATE PROCEDURE myp4(IN username VARCHAR(20),IN PASSWORD VARCHAR(20))
BEGIN
	DECLARE result INT DEFAULT 0;#声明并初始化

	SELECT COUNT(*) INTO result#赋值
	FROM admin
	WHERE admin.username = username
	AND admin.password = PASSWORD;

	SELECT IF(result>0,'成功','失败');#使用
END $

#调用
CALL myp3('张飞','8888')$


#3.创建out 模式参数的存储过程
#案例1：根据输入的女神名，返回对应的男神名

CREATE PROCEDURE myp6(IN beautyName VARCHAR(20),OUT boyName VARCHAR(20))
BEGIN
	SELECT bo.boyname INTO boyname
	FROM boys bo
	RIGHT JOIN
	beauty b ON b.boyfriend_id = bo.id
	WHERE b.name=beautyName ;

END $

```
