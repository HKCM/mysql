# 主从复制

## 概述

主从复制是指将主数据库的DDL和DML操作通过二进制日志传到从库服务器中，然后在从库上对这些日志重新执行（也叫重做），从而使得从库和主库的数据保持同步。

MySQL支持一台主库同时向多台从库进行复制，从库同时也可以作为其他从服务器的主库，实现链状复制。

MySQL复制的有点主要包含以下三个方面:
1. 主库出现问题，可以快速切换到从库提供服务。
2.  实现读写分离，降低主库的访问压力。
3.  可以在从库中执行备份，以避免备份期间影响主库服务。

## 原理

从库`IOthread`读取主库的`binlong`日志,将其记录到从库的`Relaylog`. 从库的`SQLthread`再读取`Relaylog`将数据库变更同步到从库

1. Master主库在事务提交时，会把数据变更记录在二进制日志文件Binlog中。
2. 从库读取主库的二进制日志文件Binlog，写入到从库的中继日志Relay Log。
3. slave重做中继日志中的事件，将改变反映它自己的数据。

## 搭建

### 主库配置

1. 在AWS EC2上安装mariadb
```shell
sudo yum install -y mariadb mariadb-server
sudo systemctl start mariadb.service

# 修改初始密码
mysqladmin -u root password '123456'

# 登陆MariaDB 输入 :
use mysql

#将用户名位root,密码位123456的权限开放给所有客户端:
mysql> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION;
mysql>  flush privileges;
```

2. 修改配置文件`/etc/my.cnf` MariaDB中为`/etc/my.cnf.d/server.cnf`
```
[mysqld]
# mysgl服务D，保证整个集群环境中唯一，取值范围: 1-2^23-1，默认为1 
server-id=1
# 是否只读，1代表只读，0代表读写 
read-only=0
# 忽略的数据，指不需要同步的数据库
# binlog-ignore-db=mysql 
# 指定同步的数据库
# binlog-do-db=db01
# 开启binlog, 'binlog'是文件前缀
log-bin=binlog
```

3. 重启服务

```shell
sudo systemctl restart mariadb.service 
```

4. 登录mysql,创建远程连接的账号,并授予主从复制权限
```sql
-- 创建itcast用户,并设置密码,该用户可在任意主机连接该MySQL服务
-- CREATE USER 'itcast'@'%' IDENTIFIED WITH mysql_native_password BY 'Root@123456'; 
CREATE USER 'itcast'@'%' IDENTIFIED BY 'Root@123456';
-- 为'itcast'@'%'用户分配主从复制权限
GRANT REPLICATION SLAVE ON *.* TO 'itcast'@'%';
```

5通过指令,查看二进制日志坐标
```sql
show master status;
```

#### 从库配置

1. 修改配置文件`/etc/my.cnf` MariaDB中为`/etc/my.cnf.d/server.cnf`
```
[mysqld]
# mysql服务ID，保证整个集群环境中唯一，取值范围: 1-232-1，和主库不一样即可
server-id=2
# 是否只读，1代表只读，0代表读写 
read-only=1
```
2. 重启服务

```shell
sudo systemctl restart mariadb.service 
```

3. 登录mysql,设置主库配置
```sql
-- CHANGE REPLICATION SOURCE TO SOURCE_HOST='xxx.xxx',SOURCE_USER='xxx',SOURCE _PASSWORD='xxx',SOURCE_LOG_FILE='xxx',SOURCE_LOG_POS=xxx;
-- 上述是8.0.23中的语法.如果mysql是8.0.23之前的版本,执行如下SQL:
-- CHANGE MASTER TO MASTER_HOST='xxx.xxxx.xx.xoxx',MASTER_USER='xxx',MASTER_PASSWORD='xxx',MASTER_LOG_FILE='xx',MASTER_LOG_POS=xxx;
-- show master status;
CHANGE MASTER TO MASTER_HOST='13.59.63.238',MASTER_USER='itcast',MASTER_PASSWORD='Root@123456',MASTER_LOG_FILE='binlog.000001',MASTER_LOG_POS=245;
```

4. 开启同步操作

```sql
start replica; -- 8.0.22之后 
start slave; -- 8.0.22之前, mariadb用这个
```

5. 查看从库状态

```sql
show replica status; -- 8.0.22之后 
show slave status; -- 8.0.22之前, mariadb用这个
```


```
            Master_Host: 13.59.63.238
                  Master_User: itcast
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: binlog.000001
          Read_Master_Log_Pos: 245
               Relay_Log_File: mariadb-relay-bin.000002
                Relay_Log_Pos: 526
        Relay_Master_Log_File: binlog.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
```

6. 测试

在主库创建数据表并插入数据,再去从库查询, 如果需要主库之前的数据,可以先将主库导出,在从库中还原,然后再开启主从复制.
```
create database if not exists test;
use test;
create table test1(id int primary key auto_increment,name varchar(10));
insert into test1 values (null,'AAA'),(null,'BBB');
```