## MySQL管理

### 系统数据库

- information_schema: 提供了访问数据库元数据的各种表和视图，包含数据库、表、字段类型及访问权限等
- mysql: 存储MySQL服务器正常运行所需要的各种信息（时区、主从、用户、权限等）
- proformance_schema: 为MySQL服务器运行时状态提供了一个底层监控功能，主要用于收集数据库服务器性能参数
- sys: 包含了一系列方便DBA和开发人员利用performance_schema性能数据库进行性能调优和诊断的视图

### 常用工具

#### mysql

选项：
```
-u，--user=name 指定用户名
-p，--password[=name]指定密码
-h，--host=name 指定服务器P或域名
-P，--port=port 指定连接端口▣
-e，--execute=name 执行SQL语句并退出
```

1. 在shell中使用
```shell
# 主要用在脚本工具中
mysql -h192.168.200.202 -P3306 -uroot -p1234 itcast -e "select * from stu"
```

2. 编写成脚本
```shell
#!/bin/bash
# execute sql
mysql -uroot -p123456 -e "
use test;
select * from user;
" >> user.txt
```

3. 通过shell脚本执行

sql文件`user.sql`
```
use test;
select * from user;
```

shell脚本执行
```shell
#!/bin/bash
# execute sql
mysql -uroot -p123456 -e "source user.sql" >> user.txt
```

在shell中使用
```shell
mysql -uroot -p123456 < user.sql >> user.txt
```

#### mysqladmin

mysqladmin是一个执行管理操作的客户端程序。可以用它来检查服务器的配置和当前状态、创建并删除数据库等.

```shell
mysqladmin -uroot -p1234 create test01
mysql -uroot -p1234 -e "show databases"
mysgladmin -uroot -p123456 drop test01
mysgladmin -uroot-p123456 version;
```

#### mysqlbinlog

```
-d,--database=name 指定数据库名称,只列出指定的数据库相关操作.
-o,-offset=# 忽略掉日志中的前n行命令.
-r,--result-file=name 将输出的文本格式日志输出到指定文件.
-s,--short-form 显示简单格式,省略掉一些信息.
--start-datatime=datel --stop-datetime=date2 指定日期间隔内的所有日志.
--start-position=pos1 --stop-position=pos2 指定位置间隔内的所有日志.
```

```shell
mysqlbinlog -s binlog.000006
```

#### mysqlshow

```shell
# 查询每个数据库的表的数量及表中记录的数量 
mysqlshow -uroot -p1234 --count

#查询test库中每个表中的字段书,及行数 
mysqlshow -uroot -p1234 test --count

#查询test库中book表的详细情况
mysqlshow -uroot-p1234 test book --count
```

#### mysqldump

mysqldump客户端工具用来备份数据库或在不同数据库之间进行数据迁移。备份内容包含创建表，及插入表的sQL语句。

```
连接选项：

-u，--user=name 指定用户名
-p，--password[=name] 指定密码
-h，--host=name 指定服务器ip或域名
-P，-port=# 指定连接端口

备份选项：
    --all-databases：备份所有数据库
    --databases db1 db2：备份指定的数据库
    --single-transaction：对事务引擎执行热备
    --flush-logs：更新二进制日志文件
    --master-data=2
        1：每备份一个库就生成一个新的二进制文件(默认)
        2：只生成一个新的二进制文件
    --quick：在备份大表时指定该选项

输出选项：

--add-drop-database 在每个数据库创建语句前加上drop database语句
--add-drop-table 在每个表创建语句前加上drop table语句，默认开启；不开启(-skip-add-drop-table)
-n, --no-create-db 不包含数据库的创建语句
-t, --no-create-info 不包含数据表的创建语句
-d, --no-data 不包含数据
-T, --tab=name 自动生成两个文件：一个sql文件，创建表结构的语句；一个txt文件，数据文件
```

```shell
# 备份test数据库
# sql中先删除表(IF EXISTS) 然后在创建表
mysqldump -uroot -p123456 test > test_back.sql

# 导出多张表
mysqldump -uroot -p123456 --databases test --tables t1 t2>two.sql
	
# -t 只导出表数据不导表结构，添加“-t”命令参数
mysqldump -uroot -p123456 -t test > test_back.sql

# -d 只导出表结构不导表数据，添加“-d”命令参数
mysqldump -uroot -h127.0.0.1 -p123456 -P3306 -d test > test_back.sql

mysql -uroot -p123456 -e "show variables like '%secure_file_pric%'"
# -T 自动生成两个文件：一个sql文件，创建表结构的语句；一个txt文件，数据文件
mysqldump -uroot -p123456 -T /var/lib/mysql-files/ test emp

#命令行导入
mysql> use test;
mysql> source /home/test/database.sql
```

备份脚本

```shell
#!/bin/bash
#NAME:数据库备份
#DATE:*/*/*
#USER:***

#设置本机数据库登录信息
mysql_user="user"
mysql_password="passwd"
mysql_host="localhost"
mysql_port="3306"
mysql_charset="utf8mb4"
date_time=`date +%Y-%m-%d-%H-%M`

#保存目录中的文件个数
count=10
#备份路径
path=/***/

#备份数据库sql文件并指定目录
mysqldump --all-databases --single-transaction --flush-logs --master-data=2 -h$mysql_host -u$mysql_user -p$mysql_password > $path_$(date +%Y%m%d_%H:%M).sql
[ $? -eq 0 ] && echo "-----------------数据备份成功_$date_time-----------------" || echo "-----------------数据备份失败-----------------"

#找出需要删除的备份
delfile=`ls -l -crt $path/*.sql | awk '{print $9 }' | head -1`
#判断现在的备份数量是否大于阈值
number=`ls -l -crt $path/*.sql | awk '{print $9 }' | wc -l`
if [ $number -gt $count ]
then
    rm $delfile  #删除最早生成的备份，只保留count数量的备份
    #更新删除文件日志
    echo "-----------------已删除过去备份sql $delfile-----------------"
fi
```

增加定时备份
```
crontab -e

*    *    *    *    *
-    -    -    -    -
|    |    |    |    |
|    |    |    |    +----------星期中星期几 (0 - 6) (星期天 为0)
|    |    |    +---------------月份 (1 - 12)
|    |    +--------------------一个月中的第几天 (1 - 31)
|    +-------------------------小时 (0 - 23)
+------------------------------分钟 (0 - 59)

添加定时任务(每天12:50以及23:50执行备份操作)
50 12,23 * * * cd /home/;sh backup.sh >> log.txt
```

#### mysqlimport/source

mysqlimport是客户端数据导入工具，用来导入mysqldump加-T参数后导出的文本文件。

```shell
mysqlimport -uroot -p123456 test /var/lib/mysql-files/emp.txt
```

如果需要导入sql文件，可以使用mysql中的source指令：

在连接mysql之后
```sql
-- mysql -uroot -p123456
-- 切换到对应数据库
use test;
source /var/lib/mysql/test_back.sql
```