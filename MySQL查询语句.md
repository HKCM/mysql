<!-- TOC -->
  * [数据库简介](#数据库简介)
  * [SQL分类](#sql分类)
    * [DDL](#ddl)
      * [数据库操作](#数据库操作)
      * [数据表操作](#数据表操作)
    * [DML](#dml)
    * [DQL](#dql)
      * [普通查询](#普通查询)
      * [聚合函数](#聚合函数)
      * [分组查询](#分组查询)
      * [排序查询](#排序查询)
      * [分页查询](#分页查询)
    * [DCL](#dcl)
      * [用户操作](#用户操作)
      * [权限控制](#权限控制)
  * [函数](#函数)
    * [字符串函数](#字符串函数)
    * [数值函数](#数值函数)
    * [日期函数](#日期函数)
    * [流程函数](#流程函数)
  * [约束](#约束)
    * [外键约束](#外键约束)
  * [多表查询](#多表查询)
    * [多表关系](#多表关系)
    * [多表查询分类](#多表查询分类)
    * [内连接](#内连接)
    * [外连接](#外连接)
    * [自连接](#自连接)
    * [联合查询](#联合查询)
    * [子查询](#子查询)
      * [标量子查询](#标量子查询)
      * [列子查询](#列子查询)
      * [行字查询](#行字查询)
      * [表子查询](#表子查询)
      * [练习](#练习)
<!-- TOC -->

## 数据库简介

关系型数据库
- 使用表存储数据，格式统一，便于维护 
- 使用SQL语言操作，标准统一，使用方便

SQL通用语法 
- SQL语句可以单行或多行书写，以分号结尾。
- SQL语句可以使用空格/缩进来增强语句的可读性。 
- MySQL数据库的SQL语句不区分大小写，关键字建议使用大写。 
- 注释： 
  - 单行注释：-- 注释内容或 # 注释内容(MySQL特有) 
  - 多行注释：/*注释内容*/


## SQL分类

### DDL

Data Definition Language 数据定义语言，用来定义数据库对象(数据库，表，字段) 


#### 数据库操作
```sql
-- 查询所有数据库
SHOW DATABASES;

-- 创建数据库
-- mysql中有些数据占4个字节,所以使用utf8mb4
CREATE DATABASE IF NOT EXISTS dbname DEFAULT CHARSET utf8mb4;
CREATE SCHEMA IF NOT EXISTS dbname DEFAULT CHARSET utf8mb4;

-- 删除数据库
DROP DATABASE IF EXISTS dbname;

-- 使用数据库
USE dbname;

-- 查询当前使用的数据库
SELECT DATABASE();
```

#### 数据表操作
```sql
-- 查询当前数据库的所有表
SHOW TABLES;

-- 查询表结构
DESC tableName;

-- 查询建表语句
SHOW CREATE TABLE tableName;

-- 创建表
create table tb_user( 
    id INT comment 'number', 
    name VARCHAR(50) comment 'name', 
    birthday DATE comment 'birthday',
    age TINYINT UNSIGNED comment 'age', 
    score DOUBLE(4,1) comment 'score', 
    gender char(1) comment 'gender' 
    ) comment 'user table';

-- 删除表
DROP TABLE IF EXISTS tableName;

-- 删除表并重新创建(清空表数据)
TRUNCATE TABLE tableName;
```

修改表
```sql
-- 修改表名
ALTER TABLE tableName RENAME to newTableName;

-- 添加字段
-- ALTER TABLE tableName ADD field type comment 'comment';
ALTER TABLE tableName ADD nickName varchar(10) comment 'nick name';

-- 修改字段数据类型
ALTER TABLE tableName MODIFY nickName char(10) comment 'nick name';

-- 修改字段名和数据类型
ALTER TABLE tableName CHANGE oldName newName varchar(10) comment 'nick name';

-- 删除字段
ALTER TABLE tableName DROP nickName;
```

### DML

Data Manipulation Language 数据操作语言，用来对数据库表中的数据进行增删改 
- 添加数据 (INSERT)
- 修改数据(UPDATE)
- 删除数据(DELETE)

```sql
-- 添加数据
INSERT INTO `user` (id,name,age) VALUES (1,'a',10);
INSERT INTO `user` VALUES (1,'a',10);
INSERT INTO `user` VALUES (1,'a',10), (2,'b',11),(3,'c',12);

-- 修改数据
UPDATE `user` SET name='CCC' WHERE id=3;
UPDATE `user` SET name='DDD', age = 18 WHERE id=4;

-- 删除数据
DELETE FROM `user` WHERE age <= 10;

```

### DQL

Data Query Language 数据查询语言，用来查询数据库中表的记录 

```sql
SELECT 字段 FROM 表名 WHERE 条件 GROUP BY 分组字段 HAVING 分组后条件 ORDER BY 排序字段 LIMIT 分页参数
```

#### 普通查询
```sql

create table emp( 
id int comment '编号', 
workno varchar(10) comment '工号', 
name varchar(10) comment '姓名', 
gender char comment '性别', 
age tinyint unsigned comment '年龄', 
idcard char(18) comment '身份证号', 
workaddress varchar(50) comment '工作地址', 
entrydate date comment'入职时间'
) comment '员工表';

insert into emp (id,workno,name,gender,age,idcard,workaddress,entrydate) values 
(1,'1','柳岩','女',20,'123456789012345678','北京','2000-01-01'),
(2,'2','张无忌','男',18,'123456789012345670','北京','2005-09-01'),
(3,'3','韦-笑','男',38,'123456789712345670','上海','2005-08-01'), 
(4,'4','赵傲','女',18,'123456757123845670','北京','2009-12-01'), 
(5,'5','小昭','女',16,'123456769012345678','上海','2007-07-01'), 
(6,'6','杨道','男',28,'12345678931234567X','北京','2006-01-01'), 
(7,'7','范瑶','男',40,'123456789212345670','北京','2005-05-01'), 
(8,'8','黛绮丝','女',38,'123456157123645670','天津','2015-05-01'), 
(9,'9','范凉凉','女',45,'123156789012345678','北京','2010-04-01'), 
(10,'10','陈友凉','男',53,'123456789012345670','上海','2011-01-01'), 
(11,'11','张士诚','男',55,'123567897123465670','江苏','2015-05-01'), 
(12,'12','常過春','男',32,'123446757152345670','北京','2004-02-01'), 
(13,'13','张三丰','男',88,'123656789012345678','江苏','2020-11-01'), 
(14,'14','灭绝','女',65,'123456719012345670','西安','2019-05-01'), 
(15,'15','胡青牛','男',70,'12345674971234567X','西安','2018-04-01'), 
(16,'16','周芷若','女',18,null,'北京','2012-06-01');

-- 1.查询指定字段name，workno，age返回 
SELECT name, workno, age FROM emp;

-- 2.查询所有字段返回(尽量不要写星号)
SELECT * FROM emp;

-- 3.查询所有员工的工作地址，起别名
SELECT workaddress as '工作地址' FROM emp;

-- 4.查询公司员工的上班地址(去重)
SELECT DISTINCT workaddress as '工作地址' FROM emp;

-- 5. 查询年龄等于88的员工 
SELECT * FROM emp WHERE age = 88;

-- 6.查询没有身份证好的员工信息 
SELECT * FROM emp WHERE idcard IS NULL;

-- 7.查询有身份证号的员工信息 
SELECT * FROM emp WHERE idcard IS NOT NULL;

-- 8.查询年龄在15岁(包含)到20岁(包含)之间的员工信息 
SELECT * FROM emp WHERE age BETWEEN 15 AND 20;
SELECT * FROM emp WHERE age >= 15 AND age <= 20;

-- 9.查询年龄等于18或20或40的员工信息 
SELECT * FROM emp WHERE age IN (18,20,40);

-- 10.查询姓名为两个字的员工信息
SELECT * FROM emp WHERE name LIKE '__';

-- 11.查询身份证号最后一位是X的员工信息
SELECT * FROM emp WHERE idcard LIKE '%X';
```

#### 聚合函数

注意：null值不参与所有聚合函数运算。

- count 统计数量 
- max 最大值 
- min 最小值 
- avg 平均值 
- sum 求和

```sql
-- 1.统计该企业员工数量 
SELECT COUNT(*) FROM emp;

SELECT COUNT(DISTINCT workaddress)  FROM emp;

-- 2.统计该企业员工的平均年龄 
SELECT AVG(age) FROM emp;

-- 3.统计该企业员工的最大年龄 
SELECT MAX(age) FROM emp;

-- 4.统计该企业员工的最小年龄
SELECT MIN(age) FROM emp;

#5.统计该企业西安地区的员工的年龄之和
SELECT SUM(age) FROM emp WHERE workaddress = '西安';
```

#### 分组查询

WHERE与HAVING区别 
- 执行时机不同：WHERE是分组之前进行过滤，不满足WHERE条件，不参与分组；而HAVING是分组之后对结果进行过滤。 
- 判断条件不同：WHERE不能对聚合函数进行判断，而HAVING可以。

```sql
-- 1.根据性别分组，统计男性员工和女性员工的数量 
SELECT gender, COUNT(*) FROM emp GROUP BY gender;

-- 2.根据性别分组，统计男性员工和女性员工的平均年龄 
SELECT gender, AVG(age) FROM emp GROUP BY gender;

-- 3.查询年龄小于45的员工，并根据工作地址分组，获取员工数量大于等于3的工作地址
SELECT workaddress, count(workaddress) as count_addr FROM emp WHERE age < 45 GROUP BY workaddress HAVING count_addr >= 3 ;
```

#### 排序查询

排序方式 
- ASC：升序(默议值)
- DESC：降序

```sql
-- 1.根据年龄对公司的员工进行升序排序 
SELECT * FROM emp ORDER BY age;

-- 2.根据入职时间，对员工进行降序排序 
SELECT * FROM emp ORDER BY entrydate DESC;

-- 3.根据年龄对公司的员工进行升序排序，年龄相同，再按照入职时间进行降序排序
SELECT * FROM emp ORDER BY age ASC, entrydate DESC;
```

#### 分页查询

注意
- 起始索引从0开始，起始索引=(查询页码-1)*每页显示记录数。
- 分页查询是数据库的方言，不同的数据库有不同的实现，MySQL中是LMT。 
- 如果查询的是第一页数据，起始索引可以省略，直接简写为limit 10。

```sql
-- 1.查询第1页员工数据，每页展示10条记录 
SELECT * FROM emp limit 0,10;
SELECT * FROM emp limit 10;

-- 2.查询第2页员工数据，每页展示10条记录
SELECT * FROM emp limit 10,10;
```

通过连表查询加子查询进行优化
```sql
select s.* from tb_sku s,(select id from tb_sku order by id limit 9000000,10) a where s.id = a.id;
```

### DCL

Data Control Language 数据控制语言，用来创建数据库用户、控制数据库的访问权限

#### 用户操作
```sql
-- 查询用户
USE mysql;
SELECT * FROM `user`;

-- 创建用户
CREATE USER 'it'@'localhost' IDENTIFIED BY '1234';
CREATE USER 'it'@'%' IDENTIFIED BY '1234';

-- 修改用户密码
ALTER USER 'it'@'localhost' IDENTIFIED WITH mysql_native_password BY '123456';

-- 删除用户
DROP USER 'it'@'localhost';
```

#### 权限控制
- ALL,ALL PRIVILEGES 所有权限 
- SELECT 查询数据 
- INSERT 插入数据 
- UPDATE 修改数据 
- DELETE 删除数据 
- ALTER 修改表 
- DROP 删除数据库/表/视图 
- CREATE 创建数据库/表

```sql
-- 查询权限
SHOW GRANTS FOR 'it'@'%';

-- 授予权限(test数据库中所有表的所有权限)
GRANT all ON test.* TO 'it'@'%';

-- 撤销权限(test数据库中所有表的所有权限)
REVOKE all ON test.* FROM 'it'@'%';
```

## 函数

### 字符串函数
```
CONCAT(S1，S2，。。。Sn) 字符串拼接，将S1，S2，。Sn拼接成一个字符串 
LOWER(str) 将字符串str全部转为小写 
UPPER(str) 将字符串str全部转为大写 
LPAD(str，n，pad) 左填充，用字符串pad对str的左边进行填充，达到n个字符串长度 
RPAD(str，n，pad) 右填充，用字符串pad对str的右边进行填充，达到n个字符串长度 
TRIM(str) 去掉字符串头部和尾部的空格 
SUBSTRING(str，start，len) 返回从字符串str从stat位置起的len个长度的字符串
```

```sql
-- 由于业务需求变更，企业员工的工号，统一为5位数
-- 目前不足5位数的全部在前面补0。
-- 比如：1号员工的工号应该为00001。
SELECT name, LPAD(workno,5,'0')  FROM `emp`;
UPDATE emp set workno = LPAD(workno,5,'0');
SELECT name ,workno FROM emp;
```

### 数值函数

```
CEIL(x) 向上取整 G 
FLOOR(x) 向下取整 
MOD(x，y) 返回x/y的模 
RAND() 返回0~1内的随机数
ROUND(x，y) 求参数x的四舍五入的值，保留y位小数
```

```sql
-- 通过数据库的函数，生成一个六位数的随机验证码。
-- 左填充6位补0
-- 四舍五入保留整数
-- 返回0~1内的随机数 * 1000000
SELECT LPAD(ROUND(RAND()*1000000,0),6,'0');
```


### 日期函数

```
CURDATE() 返回当前日期 
CURTIME() 返回当前时间 
NOW() 返回当前日期和时间 
YEAR(date) 获取指定date的年份 
MONTH(date) 获取指定date的月份 
DAY(date) 获取指定date的日期 
DATE ADD(date，INTERVAL expr type) 返回一个日期/时间值加上一个时间间隔xpr后的时间值 
DATEDIFF(date1，date2) 返回起始时间date1和结束时间date2之间的天数
```

```sql
-- 获取年月日
SELECT YEAR(NOW());
SELECT MONTH(NOW());
SELECT DAY(NOW());

-- 70天后
select date_add(now(),INTERVAL 70 DAY);
-- 获取相隔时间
select datediff('2021-12-01','2021-10-01');

-- 查询所有员工的入职天数，并根据入职天数倒序排序
SELECT name, entrydate, DATEDIFF(now(),entrydate) as days  FROM emp order by days DESC;
```
### 流程函数

```
IF(value,t,f) 如果value为true,则返回t,否则返回f 
IFNULL(value1,value2) 如果value1不为空,返回value1,否则返回value2 
CASE WHEN val1 THEN [res1]..ELSE default END 如果valI为true,返回resl,.否则返回default默认值 
CASE expr WHEN vall THEN [res1]..ELSE default END 如果expr的值等于vall,返回resl,否则返回default默认值
```

```sql
select if(false,'OK','Error');
select IFNULL('OK','Error');
select IFNULL(null,'Error');

-- 查询mp表的员工姓名和工作地址(北京/上海-->一线城市，其他-->二线城)
SELECT name, (CASE workaddress 
	WHEN '北京' THEN '一线城市' 
	WHEN '上海' THEN '一线城市' 
	ELSE '二线城市' END) as '工作城市' FROM emp;

SELECT name, IF(workaddress IN ('北京','上海'),'一线城市','二线城市') as '工作城市' FROM emp;

-- 查询mp表的员工姓名和年龄, 年龄大于60显示OLD,年龄小于20显示YOUNG,其他显示GREAT
SELECT name,age,CASE
	WHEN age > 60 THEN 'OLD'
	WHEN age < 20 THEN 'YOUNG' 
	ELSE 'GREAT' END FROM emp;
```

## 约束

概念：约束是作用于表中字段上的规则，用于限制存储在表中的数据。 

目的：保证数据库中数据的正确、有效性和完整性。

- NOT NULL: 非空约束 限制该字段的数据不能为nul 
- UNIQUE: 唯一约束 保证该字段的所有数据都是唯一、不重复的 
- PRIMARY KEY: 主键约束 主键是一行数据的唯一标识，要求非空且唯一 
- DEFAULT: 默认约束 保存数据时，如果未指定该字段的值，则采用默认值 
- CHECK: 检查约束(8.0.16版本之后) 保证字段值满足某一个条件 
- FOREIGN KEY: 外键约束 用来让两张表的数据之间建立连接，保证数据的一致性和完整性 

### 外键约束

- NO ACTION 当在父表中删除/更新对应记录时，首先检查该记录是否有对应外键，如果有则不允许删除/更新。(与RESTRICT一致)
- RESTRICT 当在父表中删除/更新对应记录时，首先检查该记录是否有对应外键，如果有则不允许删除更新。(与NO ACTION一致)
- CASCADE 当在父表中删除/更新对应记录时，首先检查该记录是否有对应外键，如果有，则也删除更新外键在子表中的记录。
- SET NULL 当在父表中删除对应记录时，首先检查该记录是否有对应外键，如果有则设置子表中该外键值为null(这就要求该外键允许取null)。 
- SET DEFAULT 父表有变更时，子表将外键列设置成一个默认的值(Innodb不支持)

```sql
CREATE TABLE user(
	id INT PRIMARY KEY AUTO_INCREMENT,
	name VARCHAR(10) NOT NULL UNIQUE,
	age TINYINT UNSIGNED CHECK(age > 0 && age < 150),
	status CHAR(1) DEFAULT '1',
	gender CHAR(1),
    dept_id INT
) comment 'user';

CREATE TABLE dept(
	id INT PRIMARY KEY AUTO_INCREMENT,
	name VARCHAR(10) NOT NULL UNIQUE
) comment 'department';

-- 添加外键约束
ALTER TABLE user ADD CONSTRAINT fk_emp_dept_id FOREIGN KEY (dept_id) REFERENCES dept(id);
-- 删除外键约束
ALTER TABLE user DROP FOREIGN KEY fk_emp_dept_id;

-- 添加外键约束 CASCADE
ALTER TABLE user ADD CONSTRAINT fk_dept_id FOREIGN KEY (dept_id) REFERENCES dept(id) 
    ON UPDATE CASCADE ON DELETE CASCADE;

ALTER TABLE user ADD CONSTRAINT fk_dept_id FOREIGN KEY (dept_id) REFERENCES dept(id) 
    ON UPDATE SET NULL ON DELETE SET NULL;
```

## 多表查询

### 多表关系
一对多(多对一)
案例：部门与员工的关系
关系：一个部门对应多个员工，一个员工对应一个部门 
实现：在多的一方建立外键，指向一的一方的主键

多对多
案例：学生与课程的关系 
关系：一个学生可以选修多门课程，一门课程也可以供多个学生选择
实现：建立第三张中间表，中间表至少包含两个外键，分别关联两方主键

一对一
案例：用户与用户详情的关系
关系：一对一关系，多用于单表拆分，将一张表的基础字段放在一张表中，其他详情字段放在另一张表中，以提升操作效率
实现：在任意一方加入外键，关联另外一方的主键，并且设置外键为唯一的UNIQUE)



### 多表查询分类
- 连接查询 
  - 内连接：相当于查询A、B交集部分数据
  - 外连接 
    - 左外连接：查询左表所有数据，以及两张表交集部分数据
    - 右外连接：查询右表所有数据，以及两张表交集部分数据 
    - 自连接：当前表与自身的连接查询，自连接必须使用表别名
- 子查询

### 内连接

```sql
-- 隐式内连接
-- 查询每一个员工的姓名，及关联的部门的名称(隐式内连接实现)
select emp.name,dept.name from emp,dept where emp.dept_id = dept.id;
select e.name,d.name from emp e,dept d where e.dept_id = d.id;

-- 显式内连接
-- 查询每一个员工的姓名，及关联的部门的名称(显式内连接实现)
select e.name,d.name from emp e INNER JOIN dept d ON e.dept_id = d.id;
```
### 外连接

```sql
-- 左外连接
-- 查询emp表的所有数据，和对应的部门信息(左外连接) 
SELECT e.*, d.name FROM emp e LEFT OUTER JOIN dept d ON e.dept_id = d.id;

-- 右外连接
-- 查询dept表的所有数据，和对应的员工信息(右外连接)
SELECT d.*, e.name FROM emp e RIGHT OUTER JOIN dept d ON e.dept_id = d.id;
```

### 自连接

```sql
-- 查询员工及其所属领导的名字 
SELECT a.name, b.name FROM emp a, emp b WHERE a.id = b.managerid;

-- 查询所有员工emp及其领导的名字emp，如果员工没有领导，也需要查询出来
SELECT a.name, b.name FROM emp a LEFT JOIN emp b ON a.id = b.managerid;
```

### 联合查询

对于联合查询的多张表的列数必须保持一致，字段类型也需要保持一致。 union all会将全部的数据直接合并在一起，union会对合并之后的数据去重。

主要用于分库分表的查询

```sql
-- 将薪资低于5000的员工，和年龄大于50岁的员工全部查询出来。
SELECT * FROM emp WHERE salary < 5000;
UNION
SELECT * FROM emp WHERE age > 50;

```
### 子查询

根据子查询结果不同，分为： 
- 标量子查询(子查询结果为单个值)
- 列子查询(子查询结果为一列)
- 行子查询(子查询结果为一行)
- 表子查询(子查询结果为多行多列)

#### 标量子查询
```sql
-- 1.查询'销售部'的所有员工信息 
SELECT * FROM emp WHERE dept_id = (SELECT id FROM dept WHERE name = '销售部');
-- 2.查询在'方东白'入职之后的员工信息
SELECT * FROM emp WHERE entrydate > (SELECT entrydate FROM emp WHERE name = '方东白');
```

#### 列子查询
```sql
-- 1.查询'销售部'和'市场部'的所有员工信息 
SELECT * FROM emp WHERE dept_id in (SELECT id FROM dept WHERE name='销售部' or name='市场部');
-- 2.查询比'财务部'所有人工资都高的员工信息
SELECT * FROM emp WHERE salary > ALL (
    SELECT salary FROM emp WHERE dept_id = (
        SELECT id FROM dept WHERE name='财务部'));
-- 3.查询比研发部最低工资高的员工信息
SELECT * FROM emp WHERE salary > ANY (
    SELECT salary FROM emp WHERE dept_id = (
        SELECT id FROM dept WHERE name='研发部'));
```

#### 行字查询
```sql
-- 1.查询与“张无忌”的薪资及直属领导相同的员工信息：
SELECT * FROM emp WHERE (salary,manager) = (SELECT salary, manager FROM emp WHERE name = '张无忌');
```

#### 表子查询
```sql
-- 查询与”鹿仗客”，“宋远桥”的职位和薪资相同的员工信息 
SELECT name,job,salary FROM emp WHERE (job,salary) in (SELECT job,salary FROM emp WHERE name = '鹿仗客' OR name = '宋远桥');

-- 查询入职日期是“2006-01-01“之后的员工信息，及其部门信息
SELECT e.name, d.name FROM (SELECT * FROM emp WHERE entrydate > '2006-01-01') e LEFT JOIN dept d ON e.dept_id = d.id;
```

#### 练习
```sql
-- 查询所有员工的工资等级
select e.*, s.grade, s.losal, s.hisal from emp e, salgrade s where e.salary between s.losal and s.hisal;
-- 查询“研发部“所有员工的信息及工资等级
SELECT e.*, s.grade FROM emp e, dept d, salgrade s WHERE e.dept_id = d.id and (e.salary between s.losal and s.hisal) and d.name = '研发部';
-- 查询研发部的平均薪资
select avg(e.salary) from emp e,dept d where e.dept_id=d.id and d.name='研发部';
-- 查询工资比“灭绝”高的员工信息。
SELECT * FROM emp WHERE salary > (SELECT salary FROM emp WHERE name = '灭绝'); 

-- 查询低于本部门平均薪资的员工
SELECT *, (SELECT AVG(e2.salary) FROM emp e2 WHERE e2.dept_id = e1.dept_id) 'avg' FROM emp e1 WHERE e1.salary < (SELECT AVG(e2.salary) FROM emp e2 WHERE e2.dept_id = e1.dept_id );
-- 查询所有部门信息并统计部门人数
select d.id,d.name,(select count(*) from emp e where e.dept_id = d.id ) '人数' from dept d;
```


