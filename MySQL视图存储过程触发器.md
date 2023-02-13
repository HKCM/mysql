
# 视图/存储过程/触发器

## 视图

### 介绍

视图(Viw)是一种虚拟存在的表。视图中的数据并不在数据库中实际存在，行和列数据来自定义视图的查询中使用的表，并且是在使用视图时动态生成的。

通俗的讲，视图只保存了查询的SQL逻辑，不保存查询结果。所以我们在创建视图的时候，主要的工作就落在创建这条SQL查询语句上。

视图的检查选项:
当使用WITH CHECK OPTION子句创建视图时，MySQL会通过视图检查正在更改的每个行，例如插入，更新，删除，以使其符合视图的定义。MySQL允许基于另一个视图创建视图，它还会检查依赖视图中的规则以保持一致性。为了确定检查的范围，MySQL提供了两个选项： CASCADED和LOCAL，默认值为CASCADED。

```sql
-- CREATE [OR REPLACE] VIEW 视图名称[(列名列表)] AS SELECT语句 [ WITH [ CASCADED | LOCAL ] CHECK OPTION ]

create or replace view stu_v_1 as select id,name from student where id <10 WITH CASCADED CHECK OPTION;

create view v1 as select id,name from student where id <20;
-- 基于v1创建视图v2(cascaded)
-- 此时不但需要满足id大于10,还需要满足id小于20(相当于为v1也添加约束)
create view v2 as select id,name from v1 where id >=10 with cascaded check option;
-- 基于v2创建视图v3(无cascaded)
-- v3没有check选项所以id <=15并不是插入约束
-- v3基于v2,v2基于v1,所以插入数据会有v1,v2的约束
create view v3 as select id,name from v2 where id <=15;
-- 基于v1创建视图v4(local)
-- 此时只需要满足id大于10(约束只作用于v4)
create view v4 as select id,name from v1 where id > 10 with local check option;

-- 查看创建视图语句：
SHOW CREATE VIEW viewName;
-- 查询视图数据：
SELECT * FROM viewName;
-- 修改视图
create or replace view stu_v_1 as select id,name from student where id <10;
ALTER VIEW viewName AS select id,name from student where id <10;
-- 删除视图
DROP VIEW IF EXISTS viewName;
```

### 视图的更新

要使视图可更新，视图中的行与基础表中的行之间必须存在一对一的关系。如果视图包含以下任何一项，则该视图不可更新：
1. 聚合函数或窗口函数(SUM()、MIN()、MAX()、COUNT()等)
2. DISTINCT
3. 3.GROUP BY
4. 4.HAVING
5. 5.UNION或者UNION ALL

### 作用

简单: 视图不仅可以简化用户对数据的理解，也可以简化他们的操作。那些被经常使用的查询可以被定义为视图，从而使得用户不必为以后的操作 每次指定全部的条件。
安全: 数据库可以授权，但不能授权到数据库特定行和特定的列上。通过视图用户只能查询和修改他们所能见到的数据
数据独立: 视图可帮助用户屏蔽真实表结构变化带来的影响。

```sql
-- 为了保证数据库表的安全性，开发人员在操作tb_user表时，只能看到的用户的基本字段，屏蔽手机号和邮箱两个字段。 
create view tb_user_view as select id,name,profession,age,gender,status from tb_user;
select * from tb_user_view;

-- 查询每个学生所选修的课程(三张表联查)，这个功能在很多的业务中都有使用到，为了简化操作，定义一个视图。
create view tb_stu_course_view as select s.name student_name, s.no, c.name course_name from student s,student_course sc,course c where s.id=sc.studentid and sc.courseid=c.id;
```

## 存储过程

### 存储过程介绍

存储过程是事先经过编译并存储在数据库中的一段SQL语句的集合，调用存储过程可以简化应用开发人员的很多工作，减少数据在数据库和应用服务器之间的传输，对于提高数据处理的效率是有好处的。 

存储过程思想上很简单，就是数据库SQL语言层面的代码封装与重用。

特点:
- 封装，复用 
- 可以接收参数，也可以返回数据
- 减少网络交互，效率提升

### 存储过程使用

```sql
-- 创建存储过程
CREATE PROCEDURE p1() 
BEGIN
SELECT COUNT(*) FROM user;
END;

-- 调用存储过程
CALL p1();

-- 查看存储过程
select * from information_schema.ROUTINES where ROUTINE_SCHEMA='test';

-- 查看存储过程的创建语句
SHOW CREATE PROCEDURE p1;

-- 删除存储过程
DROP PROCEDURE IF EXISTS p1;
```

注意：在命令行中，执行创建存储过程的SQL时， 需要通过关键字delimiter指定SQL语句的结束符。

```sql
-- 在命令行中创建存储过程
delimiter $$
CREATE PROCEDURE p1() 
BEGIN
SELECT COUNT(*) FROM user;
END$$
delimiter ;
```

### 变量

#### 系统变量

系统变量是MySQL服务器提供，不是用户定义的，属于服务器层面。分为全局变量(GLOBAL)、会话变量(SESSION)

```sql
-- 查看系统变量
SHOW SESSION VARIABLES;
SHOW GLOBAL VARIABLES LIKE 'auto%';
-- 查看指定系统变量
SELECT @@global.autocommit;

-- 设置系统变量
SET SESSION autocommit=0;
```

注意： 如果没有指定SESSION/GLOBAL，默认是SESSION，会话变量。 mysql服务重新启动之后，所设置的全局参数会失效，要想不失效，可以在`/etc/my.cnf`中配置。

#### 自定义变量

用户定义变量是用户根据需要自己定义的变量，用户变量不用提前声明，在用的时候直接用`@变量名`使用就可以。其作用域为当前连接。

```sql
-- 赋值
set @myname='itcast';
set @myage:=10;
set @mygender:='男', @myhobby:='女';
select @mycolor:='red';
select count(*) into @mycount from user;

-- 使用
select @myname,@myage
```

注意： 用户定义的变量无需对其进行声明或初始化，只不过获取到的值为NU儿L。

#### 局部变量

局部变量是根据需要定义的在局部生效的变量，访问之前，需要DECLARE声明。可用作存储过程内的局部变量和输入参数，局部变量的范围是在其内声明的BEGIN-END块。

```sql

CREATE PROCEDURE p1()
BEGIN
    -- 声明
    DECLARE stu_count int DEFAULT 0;
    -- 赋值
    SET stu_count:= 100;
    SELECT count(*) INTO stu_count FROM user;
    -- 查询
    SELECT stu_count;
END;
```

### IF条件判断

根据定义的分数score变量，判定当前分数对应的分数等级: 
1. score>=85分，等级为优秀。 
2. score>=60分且score<85分，等级为及格。
3. score<60分，等级为不及格。


```sql
CREATE PROCEDURE p1(IN score INT, OUT result varchar(4))
BEGIN
    -- -- 声明变量
    -- DECLARE score int DEFAULT 58;
    -- DECLARE result varchar(4);
    -- 赋值
    IF score >= 85 THEN
        SET result := '优秀';
    ELSEIF score >= 60 THEN
        SET result := '及格';
    ELSE
        SET result := '不及格';
    END IF;
    -- -- 查询
    -- SELECT result;
END;

CALL p1(58,@result);
SELECT @result;
```

将传入的200分制的分数，进行换算，换算成百分制，然后返回。
```sql
CREATE PROCEDURE p2(INOUT score DOUBLE)
BEGIN
    SET score := score * 0.5;
END;

SET @score := 188;
CALL p2(@score);
SELECT @score;
```

### CASE

```sql
CREATE PROCEDURE p1(in month int)
BEGIN
    DECLARE result varchar(10);
    CASE
        WHEN month >=1 and month <= 3 THEN
            SET result := '第一季度';
        WHEN month >=4 and month <= 6 THEN
            SET result := '第二季度';
        WHEN month >=7 and month <= 9 THEN
            SET result := '第三季度';
        WHEN month >=10 and month <= 12 THEN
            SET result := '第四季度';
        ELSE
            SET result := '非法参数';
    END CASE;
    SELECT concat('您输入的月份为:', month, ', 所属的季度为: ', result);
END;

CALL p1(16);
```

### WHILE循环

```sql
CREATE PROCEDURE p1(in n int)
BEGIN
	DECLARE iValue INT DEFAULT 0;
    DECLARE total INT DEFAULT 0;
    WHILE n > 0 DO
        SET total := total + n;
        SET n := n - 1;
       	SET iValue := iValue + 1;
    END WHILE;
    SELECT concat('您输入的值为:', iValue, ', 累加结果为: ', total);
END;

CALL p1(10);
```

### REPEAT循环

```sql
CREATE PROCEDURE p1(in n int)
BEGIN
	DECLARE iValue INT DEFAULT 0;
    DECLARE total INT DEFAULT 0;
    REPEAT
        SET total := total + n;
        SET n := n - 1;
       	SET iValue := iValue + 1;
    UNTIL n <= 0
    END REPEAT;
    SELECT concat('您输入的值为:', iValue, ', 累加结果为: ', total);
END;

CALL p1(10);
```

### LOOP循环

LOOP实现简单的循环,如果不在SQL逻辑中增加退出循环的条件,可以用其实现简单的死循环.LOOP可以配合以下两个语句使用
- LEAVE: 配合循环使用,退出循环
- ITERATE: 必须用在循环中,跳过当前循环,进入下一次循环

通过LOOP实现累加
```sql
CREATE PROCEDURE p1(in n int)
BEGIN
	DECLARE iValue INT DEFAULT 0;
    DECLARE total INT DEFAULT 0;
    SUM:LOOP
	    IF n <= 0 THEN
	    	LEAVE SUM;
	    END IF;
        SET total := total + n;
        SET n := n - 1;
       	SET iValue := iValue + 1;
    END LOOP SUM;
    SELECT concat('您输入的值为:', iValue, ', 累加结果为: ', total);
END;

CALL p1(10);
```

通过LOOP实现偶数计算
```sql
CREATE PROCEDURE p1(in n int)
BEGIN
	DECLARE iValue INT DEFAULT 0;
    DECLARE total INT DEFAULT 0;
    SUM:LOOP
	    IF n <= 0 THEN
	    	LEAVE SUM;
	    END IF;
        IF n%2=0 THEN
	    	SET total := total + n;
            SET n := n - 1;
        ELSE
        	SET n := n - 1;
	    END IF;
       	SET iValue := iValue + 1;
    END LOOP SUM;
    SELECT concat('您输入的值为:', iValue, ', 累加结果为: ', total);
END;

CALL p1(10);
```

### 游标

游标(CURSOR)是用来存储查询结果集的数据类型,在存储过程和函数中可以使用游标对结果集进行循环处理. 游标的使用包括游标的声明,OPEN,FETCH和CLOSE

根据传入参数`uage`来查询`tb_user`表中所有年龄小于`uage`的用户姓名(name)和性别(gender), 并将用户的姓名和性别插入到一张新表中
```sql
CREATE PROCEDURE p1(in uage int)
BEGIN
	DECLARE uname VARCHAR(10);
	DECLARE ugender CHAR(1);
	DECLARE u_cursor CURSOR FOR SELECT name,gender FROM emp WHERE age <= uage;
	DECLARE EXIT HANDLER FOR SQLSTATE '02000' CLOSE u_cursor;
	-- DECLARE EXIT HANDLER FOR NOT FOUND CLOSE u_cursor;
	-- SQLSTATE: 指定状态码
	-- SQLWARNING: 以01开头的状态码
	-- NOT FOUND: 以02开头的状态码
	-- SQLEXCEPTION: 其他异常状态码
	CREATE TABLE IF NOT EXISTS tb_user_gender(
		id INT PRIMARY KEY AUTO_INCREMENT,
		name VARCHAR(10),
		gender CHAR(1)
	);
	OPEN u_cursor;
	WHILE TRUE DO
		FETCH u_cursor INTO uname,ugender;
		INSERT INTO tb_user_gender VALUES (null,uname,ugender);
	END WHILE;
	CLOSE u_cursor;
END;

CALL p1(40);
```

## 存储函数

存储函数是有返回值的存储过程,存储函数的参数只能是IN类型

CHARACTERISTIC:
- NO SQL: 不包含SQL语句
- DETERMINISTIC: 相同的参数总是产生相同的结果
- READS SQL DATA: 包含读取数据语句,不包含写入数据语句

```sql
-- 参数不用写IN
CREATE FUNCTION f1(n INT)
RETURNS INT DETERMINISTIC
BEGIN
	DECLARE total INT DEFAULT 0;
	WHILE n > 0 DO
		SET total := total + n;
		SET n := n - 1;
	END WHILE;
	RETURN total;
END;

SELECT f1(100);
```

## 触发器

### 触发器介绍

触发器是与表有关的数据库对象，指在insert/update/delete之前或之后，触发并执行触发器中定义的SQL语句集合。触发器的这种特性可以协助应用在数据库端确保数据的完整性，日志记录，数据校验等操作。 

使用别名OLD和NEW来引用触发器中发生变化的记录内容，这与其他的数据库是相似的。**现在触发器还只支持行级触发，不支持语句级触发**。

- INSERT型触发器: NEW表示将要或者已经新增的数据 
- UPDATE型触发器: OLD表示修改之前的数据，NEW表示将要或已经修改后的数据 
- DELETE型触发器: OLD表示将要或者已经删除的数据

### 触发器使用

```sql
-- 创建
-- 通过触发器记录tb_user表的数据变更日志，将变更日志插入到日志表user logs中，包含增加，修改，删除；
CREATE TABLE user_logs(
    id int(11) NOT NULL PRIMARY KEY AUTO_INCREMENT,
    operation varchar(20) NOT NULL comment '操作类里,insert/update/delete',
    operate_time datetime NOT NULL comment '操作时间',
    operate_id int(11)NOT NULL comment '操作的iD',
    operate_params varchar(500) comment '操作参数'
)engine=innodb default charset=utf8;

CREATE TRIGGER emp_insert_trigger
AFTER INSERT on emp FOR EACH ROW -- 行级触发器
-- BEFORE/AFTER INSERT/UPDATE/DELETE
BEGIN
	INSERT INTO user_logs(id,operation,operate_time,operate_id,operate_params) VALUES
	(null,'INSERT',now(), NEW.id, CONCAT('插入的数据内容为: id=',NEW.id,',name=',NEW.name,',age=',NEW.age,',entrydate=',NEW.entrydate));
END;

-- 查看
SHOW TRIGGERS;

-- 往emp插入数据测试
insert into emp (id,workno,name,gender,age,idcard,workaddress,entrydate) values 
(17,'17','柳岩2','女',21,'123456789012345678','北京','2000-01-01');

-- 修改触发器
CREATE TRIGGER emp_update_trigger
AFTER UPDATE on emp FOR EACH ROW -- 行级触发器
-- BEFORE/AFTER INSERT/UPDATE/DELETE
BEGIN
	INSERT INTO user_logs(id,operation,operate_time,operate_id,operate_params) VALUES
	(null,'UPDATE',now(), NEW.id, 
        CONCAT('更新前的数据内容为: id=',OLD.id,',name=',OLD.name,',age=',OLD.age,',entrydate=',OLD.entrydate,
                    ' | 更新后的数据内容为: id=',NEW.id,',name=',NEW.name,',age=',NEW.age,',entrydate=',NEW.entrydate ));
END;

-- 如果更新影响了多行数据 触发器会触发多次
UPDATE emp SET name = '樱' WHERE id = 17;

-- 删除触发器
CREATE TRIGGER emp_delete_trigger
AFTER DELETE on emp FOR EACH ROW -- 行级触发器
-- BEFORE/AFTER INSERT/UPDATE/DELETE
BEGIN
	INSERT INTO user_logs(id,operation,operate_time,operate_id,operate_params) VALUES
	(null,'DELETE',now(), NEW.id, CONCAT('删除的数据内容为: id=',OLD.id,',name=',OLD.name,',age=',OLD.age,',entrydate=',OLD.entrydate ));
END;

DELETE FROM emp WHERE id = 17;

-- 删除
DROP TRIGGER emp_insert_trigger;
```