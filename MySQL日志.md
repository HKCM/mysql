
# 日志
<!-- TOC -->
* [日志](#日志)
  * [错误日志](#错误日志)
  * [二进制日志](#二进制日志)
  * [查询日志](#查询日志)
  * [慢查询日志](#慢查询日志)
<!-- TOC -->


## 错误日志

错误日志是MySQL中最重要的日志之一，它记录了当Mysql启动和停止时，以及服务器在运行过程中发生任何严重错误时的相关信息 。当数据库出现任何故障导致无法正常使用时，建议首先查看此日志。

该日志是默认开启的，默认存放目录`/var/log/`，默认的日志文件名为`mysqld.log`。在docker中使用`docker logs -f mysql`查看

查看日志位置: 
```sql
show variables like '%log_error%';
```

## 二进制日志

二进制日志(BINLOG)记录了所有的DDL(数据定义语言)语句和DML(数据操纵语言)语句，但不包括数据查询(SELECT、SHOW)语句。

作用: 
1. 灾难时的数据恢复；
2. M心ySQL的主从复制。在MySQL8版本中，默认二进制日志是开启着的，涉及到的参数如下: 
```sql
show variables like '%log_bin%';
```

MySQL服务器中提供了多种格式来记录二进制日志，具体格式及特点如下: 

- STATEMENT: 基于SQL语句的日志记录，记录的是SQL语句，对数据进行修改的sQL都会记录在日志文件中。
- ROW: 基于行的日志记录，记录的是每一行的数据变更。（默认）
- MIXED: 混合了STATEMENTa和ROW两种格式，默认采用STATEMENT，在某些特殊情况下会自动切换为ROW进行记录。

mysqlbinlog参数选项: 
```
-d 指定数据库名称，只列出指定的数据库相关操作。
-o 忽略掉日志中的前n行命令。
-v 将行事件（数据变更）重构为SQL语句
-vv 将行事件（数据变更）重构为SQL语句，并输出注释信息
```

```sql
show variables like '%binlog_format%'；

mysqlbinlog -d databaseName -v  binlong.000003
```

日志删除

对于比较繁忙的业务系统，每天生成的binlog数据巨大，如果长时间不清除，将会占用大量磁盘空间。可以通过以下几种方式清理日志(在mysql中): 
- reset master: 删除全部binlog日志，删除之后，日志编号，将从binlog.000001重新开始
- purge master logs to 'binlog.XXXXX' :删除***编号之前的所有日志
- purge master logs before 'yyyy-mm-dd hh24:mi:ss': 删除日志为`yyyy-mm-ddhh24:mi:ss`之前产生的所有日志

也可以在Mysql的配置文件中配置二进制日志的过期时间设置了之后,二进制日志过期会自动删除.
```sql
-- 2592000 默认30天
show variables like '%binlog_expire_logs_seconds%';
```

## 查询日志

查询日志中记录了客户端的所有操作语句，而二进制日志不包含查询数据的SQL语句。默认情况下，查询日志是未开启的。如果需要开启查询日志，可以设置以下配置: 

```sql
show variables like '%general%';
-- general_log	OFF
-- general_log_file	 /var/lib/mysql/efe5f52cface.log
```

修改MySQL的配置文件`/etc/my.cnf`文件，添加如下内容: 
```
# 该选项用来开启查询日志，可选值: 0或者1；0代表关闭，1代表开启 
general_log=1
# 设置日志的文件名，如果没有指定，默认的文件名为host_name。
log_general_log_file=mysql_query.log
```

## 慢查询日志

慢查询日志记录了所有执行时间超过参数`long_query_time`设置值并且扫描记录数不小于`min_examined_row_limit`的所有的SQL语句的日志，默认未开启。`long_query_time`默认为10秒，最小为0，精度可以到微秒。

```
# 慢查询日志 
slow_query_log=1 
# 执行时间参数 
long_query_time=2
```

默认情况下,不会记录管理语句,也不会记录不使用索引进行查找的查询.可以使用`log_slow_admin_statements`和更改此行为`log_queries_not_using_indexes`,如下所述.

```
# 记录执行较慢的管理语句 
log_slow_admin_statements=1 
# 记录执行较慢的未使用索引的语句 
log_queries_not_using_indexes=1
```





mysqldump --single-transaction -uroot-p123456 itcast itcast.sql