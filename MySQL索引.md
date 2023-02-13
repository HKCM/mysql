# MySQL索引

<!-- TOC -->
* [MySQL索引](#mysql索引)
  * [索引概述](#索引概述)
  * [索引结构](#索引结构)
  * [索引分类](#索引分类)
  * [索引语法](#索引语法)
    * [创建索引](#创建索引)
    * [查看索引](#查看索引)
    * [删除索引](#删除索引)
  * [SQL性能分析](#sql性能分析)
    * [慢查询日志](#慢查询日志)
    * [profile详情](#profile详情)
    * [explain执行计划](#explain执行计划)
  * [索引使用](#索引使用)
    * [最左前缀法则](#最左前缀法则)
    * [范围查询](#范围查询)
    * [索引列运算](#索引列运算)
    * [字符串不加引号](#字符串不加引号)
    * [模糊查询](#模糊查询)
    * [or连接的条件](#or连接的条件)
    * [数据分布影响](#数据分布影响)
    * [SQL提示](#sql提示)
    * [覆盖索引](#覆盖索引)
    * [前缀索引](#前缀索引)
    * [单列索引与联合索引](#单列索引与联合索引)
  * [索引设计原则](#索引设计原则)
<!-- TOC -->

## 索引概述

索引(index)是帮助MYSQL高效获取数据的数据结构（有序）。在数据之外，数据库系统还维护着满足特定查找算法的数据结构，这些数据结构以某种方式引用（指向）数据，这样就可以在这些数据结构上实现高级查找算法，这种数据结构就是索引。

优点:
- 提高数据检索的效率，降低数据库的IO成本
- 通过索引列对数据进行排序，降低数据排序的成本，降低CPU的消耗

缺点:
- 索引列也是要占用空间的。
- 索引大大提高了查询效率，同时却也降低更新表的速度，如对表进行INSERT、UPDATE、DELETE时，效率降低。

## 索引结构

MySQL的索引是在存储引擎层实现的，不同的存储引擎有不同的结构，主要包含以下几种：
- B+Tree索引 最常见的索引类型，大部分引擎都支持B+树索引(InnoDB MyISAM Memory)
- Hash索引 底层数据结构是用哈希表实现的，只有精确匹配索引列的查询才有效不支持范围查询(Memory)
- R-tree(空间索引) 空间索引是MyISAM引擎的一个特殊索引类型，主要用于地理空间数据类型，通常使用较少
- Full-text(全文索引) 是一种通过建立倒排索引，快速匹配文档的方式。类似于Lucene，Solr，ES(InnoDB MyISAM)

MySQL索引数据结构对经典的B+Tree进行了优化。在原B+Tree的基础上，增加一个指向相邻叶子节点的链表指针，就形成了带有顺序指针的B+Tree，提高区间访问的性能。

Hash索引特点:
- Hash索引只能用于对等比较(=，in)，不支持范围查询(between，>，<)
- 无法利用索引完成排序操作
- 查询效率高，通常只需要一次检索就可以了，效率通常要高于B+tree索引

为什么InnoDB存储引擎选择使用B+tree索引结构？
1. 相对于二叉树，层级更少，搜索效率高；
2. 对于B-tree，无论是叶子节点还是非叶子节点，都会保存数据，这样导致一页中存储的键值减少，指针跟着减少，要同样保存大量数据，只能增加树的高度，导致性能降低；
3. 相对Hash索引，B+tree支持范围匹配及排序操作；

## 索引分类

- 主键索引: 针对于表中主键创建的索引默认自动创建,只能有一个 PRIMARY
- 唯一索引: 避免同一个表中某数据列中的值重复,可以有多个 UNIQUE
- 常规索引: 快速定位特定数据,可以有多个
- 全文索引: 全文索引查找的是文本中的关键词，而不是比较索引中的值,可以有多个 FULLTEXT

在InnoDB存储引擎中，根据索引的存储形式，又可以分为以下两种：
- 聚集索引(Clustered Index) 将数据存储与索引放到了一块，索引结构的叶子节点保存了行数据,必须有，而且只有一个
- 二级索引(Secondary Index) 将数据与索引分开存储，索引结构的叶子节点关联的是对应的主键 可以存在多个
- 聚集索引的叶子节点下挂的是这一行的数据 。
- 二级索引的叶子节点下挂的是该字段值对应的主键值。

聚集索引选取规则：
- 如果存在主键，主键索引就是聚集索引。
- 如果不存在主键，将使用第一个唯一(UNIQUE)索引作为聚集索引。
- 如果表没有主键，或没有合适的唯一索引，则InnoDB会自动生成一个rowid作为隐藏的聚集索引。

回表查询: 先通过二级索引,找到对应的主键值,再通过主键值到聚集索引当中找到行数据

## 索引语法

### 创建索引

```sql
-- CREATE [UNIQUE|FULLTEXT] INDEX indexName ON tableName (index col name,...)
-- 1.name字段为姓名字段，该字段的值可能会重复，为该字段创建索引。
CREATE INDEX idx_user_name ON user(name);

-- 2.phone：手机号字段的值，是非空，且唯一的，为该字殿创建唯一索引。
CREATE UNION INDEX idx_user_phone ON user(phone);
-- 3.为profession、age、status创建联合索引。
CREATE UNION INDEX idx_user_profession_age_status ON user(profession,age,status);
-- 4.为email建立合适的索引来提升查询效率。
CREATE UNION INDEX idx_user_email ON user(email);
```

### 查看索引

```sql
SHOW INDEX FROM tableName\G;
```

### 删除索引

```sql
DROP INDEX indexName ON tableName;
```

## SQL性能分析

```sql
-- 服务器状态信息
-- show [session|global] status

SHOW GLOBAL STATUS LIKE 'Com_______'
```

### 慢查询日志

慢查询日志记录了所有执行时间超过指定参数(long_query_time，单位：秒，默认10秒)的所有SQL语句的日志。 MySQL的慢查询日志默认没有开启，需要在MySQL的配置文件(/etc/my.cnf)中配置如下信息：

```shell
# 开启MySOL慢日志查询开关
slow_query_log=1
# 设置慢日志的时间为2秒，SQL语句执行时间超过2秒，就会视为慢查询，记录慢查询日志
long_query_time=2
```

```sql
show variables like 'slow_query%';
```

### profile详情

show profiles 能够在做SQL优化时帮助我们了解时间都耗费到哪里去了。

```sql
-- 查看是否支持profiling
SELECT @@have_profiling;
-- 查看profiling状态,默认关闭
SELECT @@profiling;

SET [session|global] profiling=1;

-- 查看每一条SQL的耗时基本情况
SHOW PROFILES;

-- 查看指定query_id的SQL语句各个阶段的耗时情况
SHOW PROFILE FOR query 5;

-- 查看指定query_id的SQL语句CPU的使用情况
SHOW PROFILE CPU FOR query 5;
```

### explain执行计划

EXPLAIN或者DESC命令获取MySQL如何执行SELECT语句的信息，包括在SELECT语句执行过程中表如何连接和连接的顺序。

```sql
EXPLAIN SELECT * FROM user;
```

EXPLAIN字段说明:
- Id: SELECT查询的序列号，表示查询中执行SELECT子句或者是操作表的顺序(id相同，执行顺序众上到下；id不同，值越大，越先执行)。
- SELECT type: 表示SELECT的类型，常见的取值有SAMPLE(简单表，即不使用表连接或者子查询)、PRIMARY(主查询，即外层的查询)、 UNION(UNION中的第二个或者后面的查询语句)、SUBQUERY(SELECT/WHERE之后包含了子查询)等
- type: 表示连接类型,性能由好到差的连接类型为NULL、system、const(主键或唯一索引)、eq_ref、ref(非唯一索引)、range、index(遍历索引树)、all(全表扫描).
- possible_key: 显示可能应用在这张表上的索引，一个或多个
- Key: 实际使用的索引，如果为NULL，则没有使用索引
- Key_len:表示索引中使用的字节数，该值为索引字段最大可能长度，并非实际使用长度，在不损失精确性的前提下，长度越短越好。
- rows: MySQL认为必须要执行查询的行数，在innodb引擎的表中，是一个估计值，可能并不总是准确的。
- filtered: 表示返回结果的行数占需读取行数的百分比，filtered的值越大越好。

## 索引使用

```sql
-- 创建联合索引
CREATE UNION INDEX idx_user_profession_age_status ON user(profession,age,status);
```

### 最左前缀法则

如果索引了多列（联合索引），要遵守最左前缀法则。最左前缀法则指的是查询从索引的最左列开始，并且不跳过索引中的列。 如果跳跃某一列，索引将部分失效（后面的字段索引失效。

```sql
explain select * from tb_user where profession = '软件工程' and age = 31 and status= '0'; -- 索引有效
explain select * from tb_user where profession = '软件工程' and age = 31; -- 索引部分有效
explain select * from tb_user where age = 31 and status = '0'; -- 索引失效
```

### 范围查询

联合索引中，出现范围查询(`>`，`<`)，范围查询右侧的列索引失效, 在业务允许的情况下，尽可能的使用类似于 `>=` 或 `<=` 这类的范围查询，而避免使用 `>` 或 `<`

```sql
explain select * from tb_user where profession = '软件工程' and age > 30 and status= '0'; -- 索引部分有效 status索引失效
explain select * from tb_user where profession = '软件工程' and age >= 30 and status= '0'; -- 索引有效
```

### 索引列运算

不要在索引列上进行运算操作，索引将失效。

```sql
explain select * from tb_user where phone = '17799990015'; -- 索引有效
explain select * from tb_user where substring(phone,10,2) = '15'; -- 索引失效
```

### 字符串不加引号

如果索引时字符串类型字段,在查询时，不加引号，索引将失效。

```sql
explain select * from tb_user where profession = '软件工程' and age = 31 and status= '0'; -- 索引有效
explain select * from tb_user where profession = '软件工程' and age = 31 and status= 0; -- status索引失效
explain select * from tb_user where phone = '17799990015'; -- 索引有效
explain select * from tb_user where phone = 17799990015;  -- 索引失效
```

### 模糊查询

如果仅仅是尾部模糊匹配，索引不会失效。如果是头部模糊匹配，索引失效。

```sql
explain select * from tb_user where profession like '软件%'; -- 索引有效
explain select * from tb_user where profession like '%工程'; -- 索引失效
explain select * from tb_user where profession like '%工%'; -- 索引失效
```

### or连接的条件

用or分割开的条件，如果or前的条件中的列有索引，而后面的列中没有索引，那么涉及的索引都不会被用到。

```sql
-- id有索引, age没有索引, 则整句索引失效
explain SELECT FROM tb user WHERE id=10 or age=23; -- 索引失效
```

由于age没有索引，所以即使id有索引，索引也会失效。所以需要针对于age也要建立索引。建立索引后,索引生效

## 不同字符集导致索引失效

```sql
CREATE TABLE `t1` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`name` varchar(20) DEFAULT NULL,
`code` varchar(50) DEFAULT NULL,
PRIMARY KEY (`id`),
KEY `idx_code` (`code`),
KEY `idx_name` (`name`)
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8

CREATE TABLE `t2` (
`id` int(11) NOT NULL AUTO_INCREMENT,
`name` varchar(20) DEFAULT NULL,
`code` varchar(50) DEFAULT NULL,
PRIMARY KEY (`id`),
KEY `idx_code` (`code`),
KEY `idx_name` (`name`)
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8mb4

insert into t1 (`name`,`code`) values('aaa','aaa');
insert into t1 (`name`,`code`) values('bbb','bbb');
insert into t1 (`name`,`code`) values('ddd','ddd');
insert into t1 (`name`,`code`) values('eee','eee');

insert into t2 (`name`,`code`) values('aaa','aaa');
insert into t2 (`name`,`code`) values('bbb','bbb');
insert into t2 (`name`,`code`) values('ddd','ddd');
insert into t2 (`name`,`code`) values('eee','eee');

explain select * from t2 left join t1 on t1.code = t2.code where t2.name = 'dddd'

******************* 1. row ****************
id: 1
select_type: SIMPLE
table: t2
partitions: NULL
type: ref
possible_keys: idx_name
key: idx_name
key_len: 83
ref: const
rows: 1
filtered: 100.00
Extra: NULL
****************** 2. row **************
id: 1
select_type: SIMPLE
table: t1
partitions: NULL
type: ALL
possible_keys: NULL
key: NULL
key_len: NULL
ref: NULL
rows: 5
filtered: 100.00
Extra: Using where; Using join buffer (Block Nested Loop)
2 rows in set, 1 warning (0.01 sec)
```

### 数据分布影响

如果MySQL评估使用索引比全表更慢，则不使用索引。(查询的数据少走索引,查询的数据多走全表扫描)

```sql
select * from tb_user where phone >= '17799990005'; -- 因为结果的数据量大默认不走索引
select * from tb_user where phone >= '17799990015'; -- 因为结果的数据量小默认走索引
```

### SQL提示

SQL提示,是优化数据库的一个重要手段,简单来说,就是在SQL语句中加入一些人为的提示来达到优化操作的目的.


```sql
-- use index:
EXPLAIN SELECT * FROM tb_user USE INDEX (idk_user_pro) WHERE profession='软件工程';

-- ignore index:
EXPLAIN SELECT * FROM tb_user IGNORE INDEX (idx_user_pro) WHERE profession='软件工程';

-- force index:
EXPLAIN SELECT * FROM tb_user FORCE INDEX (idx_user_pro) WHERE profession='软件工程';
```

### 覆盖索引

尽量使用覆盖索引（查询使用了索引，并且需要返回的列，在该索引中已经全部能够找到），减少select *。

二级索引存的是字段值本身+主键id。查询的字段不存在就要回表查询

```sql
-- 走二级索引,二级索引存的是字段值本身+主键id没有额外字段不需要回表查询
explain select id, profession from tb_user where profession = '软件工程' and age =31 and status = '0' ;
-- 走二级索引,二级索引存的是字段值本身+主键id没有额外字段不需要回表查询
explain select id,profession,age, status from tb_user where profession = '软件工程'and age = 31 and status = '0' ;
-- 走二级索引,额外字段name不在索引中,需要回表查询name值
explain select id,profession,age, status, name from tb_user where profession = '软件工程' and age = 31 and status = '0' ;
-- 走二级索引,需要回表查询name值
explain select * from tb_user where profession = '软件工程' and age = 31 and status= '0';
```

Extra说明:
- NULL: 使用了回表查询数据
- using index condition：查找使用了索引，但是需要回表查询数据
- using where；using index：查找使用了索引，但是需要的数据都在索引列中能找到，所以不需要回表查询数据

### 前缀索引

当字段类型为字符串(varchar，text等)时，有时候需要索引很长的字符串，这会让索引变得很大，查询时，浪费大量的磁盘IO，影响查询效率。此时可以只将字符串的一部分前缀，建立索引，这样可以大大节约索引空间，从而提高索引效率。

前缀长度可以根据索引的选择性来决定，而选择性是指不重复的索引值（基数）和数据表的记录总数的比值，索引选择性越高则查询效率越高， 唯一索引的选择性是1，这是最好的索引选择性，性能也是最好的。

```sql
-- create index idx_xxxx on table_name(column(n));
-- 计算选择性
select COUNT(distinct substring(email,1,5))/COUNT (*) from tb_user;
-- 创建email前缀为5的索引
create index idx_tb_user_email_5 on table_name(email(5));
```

### 单列索引与联合索引

- 单列索引：即一个索引只包含单个列。
- 联合索引：即一个索引包含了多个列。

在业务场景中，如果存在多个查询条件，考虑针对于查询字段建立索引时，建议建立联合索引，而非单列索引。

联合索引创建时,也要考虑顺序.

## 索引设计原则

1. 针对于数据量较大，且查询比较频繁的表建立索引。(大于10000条数据)
2. 针对于常作为查询条件(where)、排序(ORDER BY)、分组(GROUP BY)操作的字段建立索引。
3. 尽量选择区分度高的列作为索引，尽量建立唯一索引，区分度越高，使用索引的效率越高。
4. 如果是字符串类型的字段，字段的长度较长，可以针对于字段的特点，建立前缀索引。
5. 尽量使用联合索引，减少单列索引，查询时，联合索引很多时候可以覆盖索引，节省存储空间，避免回表，提高查询效率。
6. 要控制索引的数量，索引并不是多多益善，索引越多，维护索引结构的代价也就越大，会影响增删改的效率。
7. 如果索引列不能存储NULL值，请在创建表时使用NOT NULL约束它。当优化器知道每列是否包含NULL值时，它可以更好地确定哪个索引最有效地用于查询。
